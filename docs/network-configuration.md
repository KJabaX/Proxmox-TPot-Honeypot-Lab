# Network Configuration

## Overview

The T-Pot VM is connected to the isolated Proxmox bridge `vmbrlab`.

```text
T-Pot:    <honeypot-address-cidr>
Gateway:  <lab-gateway>
Bridge:   vmbrlab
WAN:      vmbr0
```

The Proxmox host performs routing, NAT, ingress forwarding, and egress filtering.

## IPv4 Forwarding

IPv4 forwarding must be enabled on the Proxmox host:

```bash
sysctl net.ipv4.ip_forward
```

Expected value:

```text
net.ipv4.ip_forward = 1
```

## Outbound NAT

The lab network uses MASQUERADE when accessing the Internet through `vmbr0`:

```bash
iptables -t nat -A POSTROUTING \
-s <lab-subnet> \
-o vmbr0 \
-j MASQUERADE
```

Traffic path:

```text
<honeypot-ip>
   |
vmbrlab
   |
Proxmox routing
   |
MASQUERADE
   |
vmbr0
   |
Internet
```

## Connection Tracking

When Proxmox firewall bridges are enabled, a conntrack zone is used:

```bash
iptables -t raw -I PREROUTING \
-i fwbr+ \
-j CT --zone 1
```

Verify with:

```bash
sudo iptables -t raw -L PREROUTING -v -n --line-numbers
```

## Public Ingress

Selected public services are forwarded with DNAT.

Example for Cowrie SSH:

```bash
iptables -t nat -I PREROUTING \
-i vmbr0 \
-p tcp --dport 22 \
-m addrtype --dst-type LOCAL \
-j DNAT --to-destination <honeypot-ip>:22
```

Using `--dst-type LOCAL` avoids hard-coding a DHCP-assigned WAN address.

## TPOT-INGRESS Chain

A dedicated chain keeps public honeypot filtering separate from the main `FORWARD` chain.

```bash
iptables -N TPOT-INGRESS

iptables -I FORWARD \
-i vmbr0 \
-o vmbrlab \
-d <honeypot-ip> \
-j TPOT-INGRESS
```

Example policy:

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

The final DROP rule must remain after the ACCEPT rules.

## Exposed Services

The environment has been expanded beyond the original SSH/HTTP baseline to expose selected honeypot services.

Examples include:

| Port | Protocol | Honeypot / Service |
|---:|---|---|
| 22 | TCP | Cowrie / SSH |
| 23 | TCP | Cowrie / Telnet |
| 25 | TCP | Mailoney / SMTP |
| 80 | TCP | SNARE / HTTP |
| 443 | TCP | HTTPS honeypot |
| 445 | TCP | Dionaea / SMB |
| 5060 | TCP/UDP | SentryPeer / SIP |

Only intentionally selected honeypot ports should be forwarded.

Management ports such as `64295/tcp` and `64297/tcp` are not intended for public DNAT.

## Egress Policy

The lab is restricted from initiating arbitrary outbound traffic.

Current design:

```text
ESTABLISHED,RELATED        ACCEPT

vmbrlab -> vmbr1           DROP
vmbrlab -> vmbr2           DROP

DNS -> 1.1.1.1 UDP/53      ACCEPT
DNS -> 1.1.1.1 TCP/53      ACCEPT

HTTP TCP/80                ACCEPT
HTTPS TCP/443              ACCEPT

Other NEW traffic          LOG
Other outbound traffic     DROP
```

## Persistence

Tested rules are made persistent with `post-up` and `post-down` commands under the `vmbrlab` interface in:

```text
/etc/network/interfaces
```

The persistent configuration should:

- create the custom ingress chain,
- add the jump from `FORWARD`,
- add ACCEPT rules before the final DROP,
- recreate DNAT rules,
- recreate MASQUERADE,
- restore the conntrack zone,
- restore egress filtering,
- remove corresponding rules on interface shutdown.

## Verification

Useful commands:

```bash
sudo iptables -L FORWARD -v -n --line-numbers
sudo iptables -L TPOT-INGRESS -v -n --line-numbers
sudo iptables -t nat -L PREROUTING -v -n --line-numbers
sudo iptables -t nat -L POSTROUTING -v -n --line-numbers
sudo iptables -t raw -L PREROUTING -v -n --line-numbers
```

Direct packet verification on T-Pot:

```bash
sudo tcpdump -ni ens18 'dst host <honeypot-ip>'
```

See [Security](security.md) for the reasoning behind the filtering policy.
