# Troubleshooting

This page collects the main issues encountered while building the T-Pot lab.

## NAT Failed with Proxmox Firewall Enabled

### Symptom

T-Pot could not reliably reach the Internet even though routing and MASQUERADE rules appeared correct.

### Cause

Traffic traversing Proxmox firewall bridge interfaces needed an explicit connection-tracking zone.

### Fix

```bash
iptables -t raw -I PREROUTING \
-i fwbr+ \
-j CT --zone 1
```

Verify:

```bash
sudo iptables -t raw -L PREROUTING -v -n --line-numbers
```

## Docker Network Overlap

### Symptom

The T-Pot VM attempted to route traffic for a trusted management address through a Docker bridge instead of the Proxmox gateway.

### Cause

One T-Pot Docker network covered:

```text
<docker-network-subnet>
```

which overlaps the trusted workstation address:

```text
<trusted-client-ip>
```

### Workaround

A more specific host route was added:

```bash
ip route replace \
<trusted-client-ip>/32 \
via <lab-gateway> \
dev ens18
```

A long-term improvement would be to avoid overlapping address ranges entirely.

## VPN Policy Routing Conflict

### Symptom

Management traffic from the workstation toward T-Pot followed a VPN interface instead of the local LAN route.

### Cause

Mullvad policy routing intercepted traffic for the lab network.

### Diagnosis

Check:

```bash
ip route get <honeypot-ip>
```

The desired path is through the local Proxmox gateway rather than the VPN tunnel.

A split-tunnel exception is a cleaner long-term solution than disabling the VPN manually.

## Kibana Dashboards Missing

### Symptom

T-Pot was running and Elasticsearch had data, but expected dashboards were missing from Kibana.

### Important Discovery

```text
127.0.0.1:9200
```

was the Elasticpot honeypot, not the actual Elasticsearch service.

The relevant internal services were:

```text
Kibana:        127.0.0.1:64296
Elasticsearch: 127.0.0.1:64298
```

The dashboard export was located at:

```text
/home/<user>/tpotce/docker/tpotinit/dist/etc/objects/kibana_export.ndjson
```

Saved Objects were imported manually with Kibana's import API.

## Attack Map Empty

An empty Attack Map does not necessarily mean that public ingress is broken.

If firewall counters and packet captures already confirm that Internet traffic reaches honeypots, investigate the telemetry pipeline:

```text
Honeypot
   |
Logstash
   |
Elasticsearch
   |
   +--> Kibana
   |
   +--> Attack Map
```

Useful checks:

```bash
docker ps --format 'table {{.Names}}\t{{.Status}}' \
| grep -E 'map_|elasticsearch|logstash'

docker logs map_data --since 20m 2>&1 | tail -n 100
```

## DNAT Stopped Matching

### Symptom

An ingress rule worked initially but stopped after the upstream address changed.

### Cause

The DNAT rule was tied to a specific dynamically assigned WAN IPv4 address.

### Fix

Use:

```text
-m addrtype --dst-type LOCAL
```

for traffic arriving on `vmbr0`, rather than embedding one public address permanently.

## Rule Exists but Traffic Still Fails

Check the full path rather than only one rule.

Recommended order:

1. capture on `vmbr0`,
2. inspect DNAT counters,
3. inspect `FORWARD` and `TPOT-INGRESS` counters,
4. capture on T-Pot `ens18`,
5. inspect honeypot logs.

Useful commands:

```bash
sudo tcpdump -ni vmbr0
sudo iptables -t nat -L PREROUTING -v -n --line-numbers
sudo iptables -L FORWARD -v -n --line-numbers
sudo iptables -L TPOT-INGRESS -v -n --line-numbers
sudo tcpdump -ni ens18
```

## Persistence Problems

Runtime rules added manually with `iptables` disappear after reboot.

Only mark a rule as persistent after:

1. adding the corresponding `post-up` configuration,
2. adding matching cleanup where appropriate,
3. validating syntax,
4. rebooting,
5. checking rules and counters again.

Useful validation:

```bash
ifquery vmbrlab
```

After reboot:

```bash
sudo iptables -L FORWARD -v -n --line-numbers
sudo iptables -L TPOT-INGRESS -v -n --line-numbers
sudo iptables -t nat -L PREROUTING -v -n --line-numbers
sudo iptables -t nat -L POSTROUTING -v -n --line-numbers
sudo iptables -t raw -L PREROUTING -v -n --line-numbers
```

For a chronological record of the original implementation and tests, see [Implementation Notes](implementation-notes.md).
