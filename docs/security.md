# Security Design

## Security Objective

The honeypot is intentionally Internet-facing, so the surrounding network design assumes that the T-Pot VM may receive hostile traffic continuously.

The main goal is containment:

> Internet attackers may interact with honeypot services, but the honeypot environment should not have unrestricted access to trusted systems or the Internet.

## Network Segmentation

T-Pot is placed in:

```text
<lab-subnet>
```

on the dedicated virtual bridge:

```text
vmbrlab
```

Trusted systems are on separate networks:

```text
vmbr1  <trusted-lan-subnet>
vmbr2  <wifi-subnet>
```

New connections from the lab toward these networks are blocked.

## Stateful Filtering

Return traffic for permitted connections is accepted with conntrack:

```bash
iptables -I FORWARD 1 \
-m conntrack \
--ctstate ESTABLISHED,RELATED \
-j ACCEPT
```

This allows a trusted system to initiate an allowed management connection and receive the reply without allowing arbitrary new connections from the honeypot.

Conceptually:

```text
Trusted -> T-Pot        permitted where intended
T-Pot reply -> Trusted  permitted as ESTABLISHED
T-Pot -> Trusted NEW    blocked
```

## Restricted Egress

The honeypot is not given unrestricted outbound Internet access.

Permitted new outbound traffic is limited primarily to:

- DNS to `1.1.1.1`,
- HTTP,
- HTTPS.

Other new outbound connections are logged and dropped.

Example logging rule:

```bash
iptables -I FORWARD \
-i vmbrlab -o vmbr0 \
-m conntrack --ctstate NEW \
-m limit --limit 10/min --limit-burst 20 \
-j LOG \
--log-prefix "TPOT-EGRESS: " \
--log-level 6
```

The logging rate limit affects log generation only. The following DROP policy still blocks the traffic.

## Ingress Allowlisting

Public traffic is forwarded only to selected honeypot services.

A dedicated `TPOT-INGRESS` chain provides:

```text
selected honeypot ports -> ACCEPT
everything else         -> DROP
```

This reduces accidental exposure of unrelated services.

## Management Isolation

T-Pot management interfaces are kept separate from public honeypot ports.

Important examples:

```text
64295/tcp  T-Pot SSH management
64297/tcp  T-Pot Web UI
```

These should remain reachable only from intended management networks.

## Captured Malware

Cowrie may capture files uploaded by attackers.

Captured samples are treated as potentially malicious.

Safe handling principles:

- do not execute captured files,
- prefer static analysis,
- calculate cryptographic hashes,
- inspect file type and headers,
- use `strings`, `readelf`, and similar tools,
- keep samples isolated from normal workstation workflows.

Example static-analysis commands:

```bash
sha256sum <sample>
file <sample>
readelf -h <sample>
strings <sample> | less
```

## Logging

Blocked outbound attempts can be reviewed with:

```bash
journalctl -k | grep TPOT-EGRESS
```

Honeypot telemetry is additionally collected through T-Pot components such as Cowrie, Suricata, Elasticsearch, and Kibana.

## Validation

Security controls are checked using several independent methods:

- firewall packet counters,
- direct packet capture,
- connectivity tests,
- honeypot logs,
- Suricata events,
- reboot persistence testing.

A control should not be considered verified only because a rule exists; packet counters and observed traffic should confirm that the intended path is actually being used.

## Scope

This repository documents a learning and research environment.

The design reduces risk but does not make an Internet-facing honeypot risk-free. Isolation, patching, log monitoring, and conservative service exposure remain important operational controls.
