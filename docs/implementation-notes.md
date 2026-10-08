# T-Pot Honeypot Lab — Implementation Notes

This document preserves the detailed implementation history that was originally kept in the repository root README.

> **Public portfolio note:** This file is a sanitized historical implementation log. Environment-specific addresses and credentials have been replaced or redacted. Statements such as "current" or "next" below reflect the point in time when the original notes were written and should not be read as the live configuration today.

The root README is now intended to be a concise project overview, while this file keeps the lower-level commands, validation steps, troubleshooting notes, and historical implementation details.

---

# Proxmox T-Pot Honeypot Lab

A self-hosted cybersecurity lab built with **Proxmox VE**, **Debian**, and **T-Pot**.

The goal of this project is to run an Internet-facing honeypot in an isolated network while protecting the Proxmox host and trusted internal networks from the honeypot environment.

The lab includes:

- Dedicated honeypot network
- Proxmox firewall isolation
- Network Address Translation (NAT)
- Controlled outbound Internet access
- Egress filtering
- Connection tracking
- T-Pot honeypots
- Elasticsearch, Logstash, and Kibana
- Persistent firewall rules
- Reboot and isolation testing

> **Historical snapshot:** At this stage of the build, public ingress had been validated for **TCP/22** (Cowrie SSH) and **TCP/80** (SNARE/Tanner HTTP). The `TPOT-INGRESS` chain was in use, with persistence work still being completed. Later documentation in this repository describes the expanded service exposure and current investigation workflow.

---

## Architecture

```text
                         Internet
                            |
                         Public IP
                            |
                          vmbr0
                            |
                     +-------------+
                     |   Proxmox    |
                     |<proxmox-host>|
                     +-------------+
                            |
       +--------------------+--------------------+
       |                    |                    |
     vmbr1                vmbr2               vmbrlab
<trusted-lan-gateway-cidr>    <wifi-gateway-cidr>     <lab-gateway-cidr>
       |                    |                    |
 Trusted LAN              Wi-Fi                T-Pot
       |                                    <honeypot-ip>
 Workstation
<trusted-client-ip>
```

`vmbrlab` is an internal Proxmox bridge:

```text
bridge-ports none
```

The T-Pot VM therefore does not have direct access to a physical network interface.

Internet access is routed through the Proxmox host.

---

## Environment

### Proxmox Host

- **Hypervisor:** Proxmox VE
- **Hostname:** `<proxmox-host>`
- **Public network:** `vmbr0`
- **Trusted LAN:** `vmbr1`
- **Wi-Fi network:** `vmbr2`
- **Honeypot network:** `vmbrlab`

### T-Pot Virtual Machine

| Setting | Value |
|---|---|
| Guest OS | Debian 13.7 Trixie |
| Honeypot platform | T-Pot 24.04.1 |
| vCPU | 4 |
| RAM | 16 GiB |
| Disk | 256 GiB |
| Swap | 8 GiB |
| Network interface | VirtIO |
| Bridge | `vmbrlab` |
| IP address | `<honeypot-address-cidr>` |
| Gateway | `<lab-gateway>` |
| DNS | `1.1.1.1` |

---

## Debian VM Setup

A dedicated Debian VM was created for T-Pot.

The VM was connected only to:

```text
vmbrlab
```

with the following static network configuration:

```text
IP address: <honeypot-address-cidr>
Gateway:    <lab-gateway>
DNS:        1.1.1.1
```

The Proxmox bridge uses:

```text
<lab-gateway-cidr>
```

as the gateway for the honeypot network.

Basic connectivity was verified with:

```bash
ip -br addr
ip route
ping 1.1.1.1
```

---

## Disk Expansion

The virtual disk was expanded to approximately **256 GiB**.

After expanding the disk from Proxmox, the Debian partition and ext4 filesystem were extended.

Example tools used:

```bash
sudo apt install cloud-guest-utils
sudo growpart /dev/sda 1
sudo resize2fs /dev/sda1
```

The final root filesystem uses most of the virtual disk.

---

## Swap

An 8 GiB swap file was created:

```bash
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

This gives the VM additional memory protection because T-Pot runs several resource-intensive services such as:

- Elasticsearch
- Kibana
- Logstash
- Suricata
- Multiple honeypots

---

## T-Pot Installation

T-Pot was installed on the Debian VM using the official T-Pot installation process.

The installation was performed using a normal administrative user with `sudo` privileges instead of working directly as `root`.

After installation, T-Pot moved its real management services away from common honeypot ports.

Important management ports:

| Service | Port |
|---|---:|
| T-Pot SSH | `64295/tcp` |
| T-Pot Web UI | `64297/tcp` |
| Kibana | `127.0.0.1:64296` |
| Elasticsearch | `127.0.0.1:64298` |

Example SSH connection:

```bash
ssh -p 64295 <user>@<honeypot-ip>
```

The normal TCP port `22` is used by the **Cowrie SSH honeypot**.

---

## Network Isolation

The primary security requirement was:

> The honeypot must not be able to initiate connections to trusted internal networks.

The following traffic is blocked:

```text
vmbrlab -> vmbr1
vmbrlab -> vmbr2
```

Firewall rules:

```bash
iptables -I FORWARD 2 -i vmbrlab -o vmbr1 -j DROP
iptables -I FORWARD 3 -i vmbrlab -o vmbr2 -j DROP
```

Existing and related connections are allowed first:

```bash
iptables -I FORWARD 1 \
-m conntrack \
--ctstate ESTABLISHED,RELATED \
-j ACCEPT
```

This creates an important distinction:

```text
Trusted workstation -> T-Pot      ALLOWED
T-Pot reply -> workstation        ALLOWED
T-Pot -> workstation NEW          BLOCKED
```

---

## Isolation Test

The trusted workstation uses:

```text
<trusted-client-ip>
```

From the T-Pot VM:

```bash
ping -c 3 <trusted-client-ip>
```

Result:

```text
3 packets transmitted
0 received
100% packet loss
```

This confirmed that the T-Pot VM cannot directly reach the trusted workstation.

---

## Network Address Translation

The T-Pot VM uses a private IPv4 address, so the Proxmox host performs **Network Address Translation (NAT)**.

The rule is:

```bash
iptables -t nat -A POSTROUTING \
-s <lab-subnet> \
-o vmbr0 \
-j MASQUERADE
```

Traffic therefore follows:

```text
T-Pot
<honeypot-ip>
     |
     v
vmbrlab
     |
     v
Proxmox
     |
 MASQUERADE
     |
     v
vmbr0
     |
     v
Internet
```

---

## Connection Tracking

During testing, NAT did not initially work correctly when the Proxmox VM firewall was enabled.

A Linux **Connection Tracking (conntrack)** zone was required for Proxmox firewall bridges:

```bash
iptables -t raw -I PREROUTING \
-i fwbr+ \
-j CT --zone 1
```

This assigns traffic from Proxmox firewall bridges to connection tracking zone `1`.

After adding this rule, T-Pot Internet connectivity worked correctly.

Verification:

```bash
sudo iptables -t raw -L PREROUTING -v -n --line-numbers
```

Expected rule:

```text
CT  ...  fwbr+  ...  CT zone 1
```

---

## Egress Filtering

Initially T-Pot had unrestricted outbound Internet access.

This was replaced with a restricted egress policy.

Current policy:

```text
ESTABLISHED / RELATED                ACCEPT

T-Pot -> Trusted LAN                DROP
T-Pot -> Wi-Fi                      DROP

DNS -> 1.1.1.1 UDP/53               ACCEPT
DNS -> 1.1.1.1 TCP/53               ACCEPT

HTTP TCP/80                         ACCEPT
HTTPS TCP/443                       ACCEPT

Other NEW outbound connections      LOG
Other outbound connections          DROP
```

The goal is to reduce the ability of a compromised honeypot to attack other systems.

### DNS

DNS is allowed only to:

```text
1.1.1.1
```

Rules:

```bash
iptables -I FORWARD 4 \
-i vmbrlab -o vmbr0 \
-d 1.1.1.1 \
-p udp --dport 53 \
-m conntrack --ctstate NEW \
-j ACCEPT
```

```bash
iptables -I FORWARD 5 \
-i vmbrlab -o vmbr0 \
-d 1.1.1.1 \
-p tcp --dport 53 \
-m conntrack --ctstate NEW \
-j ACCEPT
```

### HTTP and HTTPS

Outbound HTTP:

```bash
iptables -I FORWARD 6 \
-i vmbrlab -o vmbr0 \
-p tcp --dport 80 \
-m conntrack --ctstate NEW \
-j ACCEPT
```

Outbound HTTPS:

```bash
iptables -I FORWARD 7 \
-i vmbrlab -o vmbr0 \
-p tcp --dport 443 \
-m conntrack --ctstate NEW \
-j ACCEPT
```

---

## Blocked Egress Logging

Blocked outbound connections are logged before being dropped.

Logging is rate-limited:

```bash
iptables -I FORWARD 8 \
-i vmbrlab -o vmbr0 \
-m conntrack --ctstate NEW \
-m limit \
--limit 10/min \
--limit-burst 20 \
-j LOG \
--log-prefix "TPOT-EGRESS: " \
--log-level 6
```

The rate limit applies only to log generation.

Blocked traffic is still always dropped:

```bash
iptables -I FORWARD 9 \
-i vmbrlab -o vmbr0 \
-j DROP
```

Logs can be inspected with:

```bash
journalctl -k | grep TPOT-EGRESS
```

---

## Persistent Proxmox Firewall Configuration

The rules are stored in:

```text
/etc/network/interfaces
```

under the `vmbrlab` interface.

Current structure:

```text
auto vmbrlab
iface vmbrlab inet static
        address <lab-gateway-cidr>
        bridge-ports none
        bridge-stp off
        bridge-fd 0

        # Allow return traffic
        post-up iptables -I FORWARD 1 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

        # Public SSH honeypot ingress
        post-up iptables -I FORWARD 2 -i vmbr0 -o vmbrlab -d <honeypot-ip> -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT

        # Block T-Pot from internal networks
        post-up iptables -I FORWARD 3 -i vmbrlab -o vmbr1 -j DROP
        post-up iptables -I FORWARD 4 -i vmbrlab -o vmbr2 -j DROP

        # DNS
        post-up iptables -I FORWARD 5 -i vmbrlab -o vmbr0 -d 1.1.1.1 -p udp --dport 53 -m conntrack --ctstate NEW -j ACCEPT
        post-up iptables -I FORWARD 6 -i vmbrlab -o vmbr0 -d 1.1.1.1 -p tcp --dport 53 -m conntrack --ctstate NEW -j ACCEPT

        # HTTP / HTTPS
        post-up iptables -I FORWARD 7 -i vmbrlab -o vmbr0 -p tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT
        post-up iptables -I FORWARD 8 -i vmbrlab -o vmbr0 -p tcp --dport 443 -m conntrack --ctstate NEW -j ACCEPT

        # Rate-limited egress logging
        post-up iptables -I FORWARD 9 -i vmbrlab -o vmbr0 -m conntrack --ctstate NEW -m limit --limit 10/min --limit-burst 20 -j LOG --log-prefix "TPOT-EGRESS: " --log-level 6

        # Block all other T-Pot egress
        post-up iptables -I FORWARD 10 -i vmbrlab -o vmbr0 -j DROP

        # WAN TCP/22 -> Cowrie
        post-up iptables -t nat -I PREROUTING 1 -i vmbr0 -p tcp --dport 22 -m addrtype --dst-type LOCAL -j DNAT --to-destination <honeypot-ip>:22

        # T-Pot outbound NAT
        post-up iptables -t nat -A POSTROUTING -s <lab-subnet> -o vmbr0 -j MASQUERADE

        # Proxmox firewall conntrack zone
        post-up iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
```

Corresponding `post-down` rules remove the same firewall, DNAT, NAT, and conntrack rules when the interface is stopped.

---

## Egress Testing

HTTP was tested using:

```bash
curl -I http://example.com
```

HTTPS:

```bash
curl -I https://example.com
```

Both returned:

```text
200 OK
```

A connection to a non-allowed port was also tested.

Example:

```bash
timeout 3 bash -c 'echo > /dev/tcp/1.1.1.1/12345'
```

The connection timed out.

The Proxmox firewall counters showed traffic hitting:

```text
TPOT-EGRESS LOG
```

followed by:

```text
DROP
```

This confirmed that the egress filter works.

---

## Docker Network Conflict

T-Pot creates a large number of internal Docker networks.

One of these was:

```text
<docker-network-subnet>
```

This range includes:

```text
<trusted-client-ip>
```

which is also the address of the trusted management workstation.

As a result, T-Pot initially attempted to route management traffic to a Docker bridge instead of through the Proxmox gateway.

A specific `/32` host route was added:

```bash
ip route replace \
<trusted-client-ip>/32 \
via <lab-gateway> \
dev ens18
```

Persistent configuration:

```text
up ip route replace <trusted-client-ip>/32 via <lab-gateway> dev ens18
down ip route del <trusted-client-ip>/32 via <lab-gateway> dev ens18 || true
```

Long term, changing the trusted LAN subnet would be a cleaner solution.

---

## VPN Routing Conflict

The management workstation uses Mullvad VPN.

When Mullvad was enabled, policy routing caused traffic for:

```text
<honeypot-ip>
```

to be routed through:

```text
wg0-mullvad
```

instead of the physical LAN.

With Mullvad disabled, the correct route was:

```text
<honeypot-ip>
via <trusted-lan-gateway>
dev enp5s0
```

A future improvement would be to configure a split-tunnel exception for the lab network.

---

## Kibana Dashboard Issue

T-Pot started successfully and Elasticsearch contained events, but Kibana initially did not show the expected T-Pot dashboards.

An important discovery was:

```text
127.0.0.1:9200
```

is the **Elasticpot honeypot**, not the actual Elasticsearch service.

The real services are:

```text
Elasticsearch:
127.0.0.1:64298

Kibana:
127.0.0.1:64296
```

The dashboard export was found at:

```text
/home/<user>/tpotce/docker/tpotinit/dist/etc/objects/kibana_export.ndjson
```

The Saved Objects were imported manually:

```bash
curl -X POST \
'http://127.0.0.1:64296/api/saved_objects/_import?overwrite=true' \
-H 'kbn-xsrf: true' \
--form file=@/home/<user>/tpotce/docker/tpotinit/dist/etc/objects/kibana_export.ndjson
```

The import completed successfully:

```text
successCount: 307
success: true
```

After the import, the T-Pot dashboards appeared in Kibana.

The exact reason for the failed automatic dashboard import was not determined.

---

## T-Pot Services

The current installation includes several honeypots and monitoring services, including:

- Cowrie
- Dionaea
- Conpot
- Snare
- Tanner
- Heralding
- Elasticpot
- Mailoney
- Honeytrap
- SentryPeer
- RDP honeypot
- Redis honeypot
- Suricata
- Elasticsearch
- Logstash
- Kibana

Core containers successfully start after a full Proxmox reboot.

---

## Reboot Testing

The entire Proxmox server was rebooted to verify persistence.

After reboot, the following were checked.

### FORWARD Rules

```bash
sudo iptables -L FORWARD -v -n --line-numbers
```

Expected order:

```text
1  ACCEPT  ESTABLISHED,RELATED
2  ACCEPT  vmbr0 -> vmbrlab -> <honeypot-ip> TCP/22

3  DROP    vmbrlab -> vmbr1
4  DROP    vmbrlab -> vmbr2

5  ACCEPT  DNS UDP/53 -> 1.1.1.1
6  ACCEPT  DNS TCP/53 -> 1.1.1.1

7  ACCEPT  HTTP/80
8  ACCEPT  HTTPS/443

9  LOG     TPOT-EGRESS
10 DROP    vmbrlab -> vmbr0

11 PVEFW-FORWARD
```

No duplicate rules were found.

### DNAT Ingress

```bash
sudo iptables -t nat -L PREROUTING -v -n --line-numbers
```

The persistent ingress rule was restored after reboot:

```text
vmbr0 TCP/22
  -> DNAT
  -> <honeypot-ip>:22
```

The rule uses:

```text
-m addrtype --dst-type LOCAL
```

instead of hard-coding the public IPv4 address. This is important because `vmbr0` receives its WAN address dynamically and the public address changed during testing.

### NAT

```bash
sudo iptables -t nat -L POSTROUTING -v -n --line-numbers
```

The following network was correctly masqueraded:

```text
<lab-subnet> -> vmbr0
```

### Connection Tracking

```bash
sudo iptables -t raw -L PREROUTING -v -n --line-numbers
```

Result:

```text
CT zone 1
```

### Internet

```bash
curl -I http://example.com
curl -I https://example.com
```

Both returned:

```text
200 OK
```

### Trusted LAN Isolation

```bash
ping -c 3 <trusted-client-ip>
```

Result:

```text
100% packet loss
```

---

## Current Security State

At the current stage:

| Security control | Status |
|---|---|
| Dedicated honeypot network | ✅ |
| T-Pot Internet access | ✅ |
| NAT | ✅ |
| Proxmox conntrack integration | ✅ |
| Trusted LAN isolation | ✅ |
| Wi-Fi isolation | ✅ |
| Proxmox management isolation | ✅ |
| DNS egress restriction | ✅ |
| HTTP/HTTPS egress | ✅ |
| Other outbound ports blocked | ✅ |
| Rate-limited blocked traffic logging | ✅ |
| Persistent firewall configuration | ✅ |
| Full reboot test | ✅ |
| T-Pot containers after reboot | ✅ |
| Kibana dashboards | ✅ |
| Public honeypot ingress | ✅ TCP/22 to Cowrie and TCP/80 to SNARE/Tanner (runtime verified) |

---

## Public Honeypot Ingress

### Persistent baseline

The first public honeypot service enabled was **TCP/22**, forwarded from the WAN interface to the Cowrie SSH honeypot.

TCP/22 remains the persistent baseline rule currently stored in `/etc/network/interfaces`.

```text
Internet TCP/22
       |
       v
Proxmox vmbr0
       |
      DNAT
       |
       v
<honeypot-ip>:22
       |
       v
Docker port mapping
       |
       v
Cowrie
```

The persistent DNAT rule is:

```bash
iptables -t nat -I PREROUTING 1 -i vmbr0 -p tcp --dport 22 -m addrtype --dst-type LOCAL -j DNAT --to-destination <honeypot-ip>:22
```

The corresponding forwarding rule is:

```bash
iptables -I FORWARD 2 -i vmbr0 -o vmbrlab -d <honeypot-ip> -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
```

### Why `--dst-type LOCAL` Is Used

The public IPv4 address on `vmbr0` is assigned dynamically and changed during testing.

The first temporary DNAT rule was tied to the old public IP address and therefore stopped matching after the WAN address changed.

The final rule instead uses:

```text
-m addrtype --dst-type LOCAL
```

This matches traffic addressed to a local address on the Proxmox host without permanently hard-coding a specific public IPv4 address.

### Ingress Verification

The ingress path was verified in several layers:

1. `tcpdump` on `vmbr0` showed incoming TCP/22 connections from the Internet.
2. The DNAT rule counter increased.
3. The `FORWARD` rule from `vmbr0` to `vmbrlab` increased.
4. Cowrie recorded the external SSH session.
5. Cowrie identified the test client as `OpenSSH_for_Windows_9.5`.
6. A test username and password attempt appeared in the Cowrie log.
7. The rules remained present and active after a full Proxmox reboot.

Example Cowrie events included:

```text
New connection: <EXTERNAL_IP>:<SOURCE_PORT>
Remote SSH version: SSH-2.0-OpenSSH_for_Windows_9.5
login attempt [testuser/<test-password>] failed
```

Unrelated automated Internet scans also reached Cowrie shortly after TCP/22 was exposed, confirming that the honeypot was publicly reachable.

### Management Ports Remain Private

The T-Pot management interfaces are intentionally not forwarded from the Internet:

```text
64295/tcp  T-Pot SSH management
64297/tcp  T-Pot Web UI
```

The next ingress stage was to expose HTTP selectively and reorganize the ingress firewall so that additional honeypot ports can be added without repeatedly reordering the main `FORWARD` chain.

---

## Current Runtime Ingress Structure

A dedicated custom iptables chain called:

```text
TPOT-INGRESS
```

is now used for public honeypot traffic.

The main `FORWARD` chain sends Internet traffic destined for the T-Pot VM into this chain:

```bash
iptables -I FORWARD 2 \
-i vmbr0 \
-o vmbrlab \
-d <honeypot-ip> \
-j TPOT-INGRESS
```

The current chain contains:

```bash
iptables -A TPOT-INGRESS \
-p tcp --dport 22 \
-m conntrack --ctstate NEW \
-j ACCEPT

iptables -A TPOT-INGRESS \
-p tcp --dport 80 \
-m conntrack --ctstate NEW \
-j ACCEPT

iptables -A TPOT-INGRESS -j DROP
```

Current policy:

```text
TCP/22  -> ACCEPT
TCP/80  -> ACCEPT
Other   -> DROP
```

This keeps the main `FORWARD` chain simple and makes it easier to add additional honeypot services later.

### TCP/80 / SNARE DNAT

HTTP traffic is currently forwarded to T-Pot with:

```bash
iptables -t nat -I PREROUTING 2 \
-i vmbr0 \
-p tcp --dport 80 \
-m addrtype --dst-type LOCAL \
-j DNAT --to-destination <honeypot-ip>:80
```

As with TCP/22, `--dst-type LOCAL` is used so the rule continues to work even if the DHCP-assigned WAN address changes.

### Verified Firewall Counters

The following counters were observed on the Proxmox host:

```text
DNAT TCP/22 -> <honeypot-ip>:22    943 packets
DNAT TCP/80 -> <honeypot-ip>:80    526 packets

TPOT-INGRESS TCP/22 ACCEPT         1146 packets
TPOT-INGRESS TCP/80 ACCEPT          698 packets

FORWARD -> TPOT-INGRESS            1844 packets
```

The `TPOT-INGRESS` counters match the forwarded traffic:

```text
1146 + 698 = 1844
```

This confirms that the custom chain is receiving and allowing the expected public traffic.

### Direct Packet Capture Verification

Traffic was verified directly on the T-Pot VM with:

```bash
sudo tcpdump -ni ens18 \
'dst host <honeypot-ip> and tcp and (dst port 22 or dst port 80)'
```

Command explanation:

- `tcpdump` captures packets in real time.
- `-n` keeps addresses numeric instead of resolving DNS names.
- `-i ens18` listens on the T-Pot VM network interface.
- `dst host <honeypot-ip>` shows only packets whose destination is the T-Pot VM.
- `dst port 22 or dst port 80` limits the capture to the currently exposed honeypot ports.

Real Internet SSH traffic was observed, for example:

```text
2.57.122.238.42158 > <honeypot-ip>.22: Flags [S]
2.57.122.238.42158 > <honeypot-ip>.22: SSH: SSH-2.0-Go
```

This confirms the path:

```text
Internet
  |
vmbr0
  |
DNAT
  |
TPOT-INGRESS
  |
vmbrlab
  |
<honeypot-ip>:22
  |
Cowrie
```

For only new TCP connection attempts, the following filter can be used:

```bash
sudo tcpdump -ni ens18 \
'dst host <honeypot-ip> and tcp[tcpflags] & tcp-syn != 0 and (dst port 22 or dst port 80)'
```

---

## Cowrie Captured File Upload

A real Internet SSH session reached Cowrie and progressed beyond a simple port scan.

The remote client attempted several credentials. Plaintext password values are redacted in this public portfolio snapshot:

```text
root / <REDACTED-1>  -> failed
root / <REDACTED-2>  -> failed
root / <REDACTED-3>  -> failed
root / <REDACTED-4>  -> failed
root / <REDACTED-5>  -> succeeded
```

After the emulated login succeeded, the client opened an SFTP session and uploaded a file named:

```text
sshd
```

Cowrie stored the captured file using the SHA-256 value:

```text
59f7ddd5211671eed5b8c378e228a24d849fe0a1c043941dfd4602029c66f216
```

The captured file was approximately:

```text
10.2 MB
```

Static inspection with `file` showed:

```text
ELF 64-bit LSB executable
x86-64
dynamically linked
interpreter /lib64/ld-linux-x86-64.so.2
```

`readelf` reported:

```text
Reading 2240 bytes extends past end of file for section headers
the dynamic segment offset + size exceeds the size of the file
```

The SSH/SFTP session timed out while the upload was still in progress, so the captured file appears to be incomplete.

T-Pot later rotated the sample into:

```text
/home/<user>/tpotce/data/cowrie/downloads.tgz.1
```

The file exists inside the archive as:

```text
data/cowrie/downloads/59f7ddd5211671eed5b8c378e228a24d849fe0a1c043941dfd4602029c66f216
```

Captured samples must not be executed. Static analysis tools such as `file`, `sha256sum`, `readelf`, and `strings` are preferred.

---

## Attack Map Troubleshooting

The firewall and routing path have been verified independently of the Attack Map.

Real Internet traffic reaches Cowrie, and HTTP traffic has also reached SNARE/Tanner. Therefore, if the Attack Map is empty, the first troubleshooting target should be the telemetry pipeline rather than DNAT or the Proxmox firewall.

Relevant path:

```text
Honeypot
   |
Logstash
   |
Elasticsearch
   |
   +--> Kibana
   |
   +--> Attack Map components
```

Useful container check:

```bash
docker ps --format 'table {{.Names}}\t{{.Status}}' \
| grep -E 'map_|elasticsearch|logstash'
```

Useful Attack Map data-service log check:

```bash
docker logs map_data --since 20m 2>&1 | tail -n 100
```

Because packet capture and firewall counters confirm ingress, an empty map does not by itself indicate that the honeypot is unreachable.

---

## Pending Persistence Update

The custom `TPOT-INGRESS` chain and TCP/80 DNAT rule are currently verified at runtime.

Before the next full Proxmox reboot, the persistent `/etc/network/interfaces` configuration should be updated so that:

```text
TCP/22
TCP/80
TPOT-INGRESS
```

are recreated automatically.

After editing the persistent configuration, verify it with:

```bash
ifquery vmbrlab
```

and after reboot verify again with:

```bash
sudo iptables -t nat -L PREROUTING -v -n --line-numbers
sudo iptables -L TPOT-INGRESS -v -n --line-numbers
sudo iptables -L FORWARD -v -n --line-numbers
```

Only after this reboot test should the new ingress structure be marked as fully persistent.

---

## Project Status

The internal T-Pot environment is operational.

The outbound path is working:

```text
T-Pot
 |
vmbrlab
 |
Proxmox Firewall
 |
Connection Tracking
 |
Egress Filtering
 |
NAT
 |
vmbr0
 |
Internet
```

The currently tested inbound honeypot paths are:

```text
Internet TCP/22
 |
vmbr0
 |
DNAT
 |
TPOT-INGRESS
 |
vmbrlab
 |
<honeypot-ip>:22
 |
Cowrie
```

and:

```text
Internet TCP/80
 |
vmbr0
 |
DNAT
 |
TPOT-INGRESS
 |
vmbrlab
 |
<honeypot-ip>:80
 |
SNARE / Tanner
```

At the same time:

```text
T-Pot -> Trusted LAN
```

is blocked.

The next development stage is to persist the new `TPOT-INGRESS` + TCP/80 configuration, reboot-test it, and then continue selective exposure of additional honeypot services and attack telemetry collection.
