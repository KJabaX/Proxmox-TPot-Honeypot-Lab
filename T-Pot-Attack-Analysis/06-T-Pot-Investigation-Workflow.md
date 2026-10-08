# T-Pot Investigation Workflow

This document describes the recommended end-to-end process for investigating activity captured by the T-Pot environment.

The workflow is service-agnostic: the investigation begins with network triage and only then moves into the relevant honeypot logs.

---

## Investigation Overview

```text
Source IP
    |
    v
Create case
    |
    v
Suricata triage
    |
    v
Targeted ports / protocols
    |
    v
Identify relevant honeypots
    |
    +--> Cowrie
    +--> Dionaea
    +--> Mailoney
    +--> SNARE / Tanner
    +--> h0neytr4p
    +--> SentryPeer
    +--> Heralding
    +--> Redis honeypot
    +--> RDP honeypot
    |
    v
Correlate service events
    |
    v
Malware / OSINT / PCAP
    |
    v
IOCs
    |
    v
Final timeline
```

---

## Phase 1 – Select the source

```bash
IP="<source-ip>"
```

Start with:

```text
00-T-Pot-Attack-Triage.md
```

---

## Phase 2 – Create the case

```bash
CASE="$HOME/analysis/$IP-$(date -u +%Y%m%dT%H%M%SZ)"
mkdir -p "$CASE"
```

Keep exported evidence and analysis output inside the case directory.

---

## Phase 3 – Collect Suricata evidence

Export all Suricata events involving the source IP.

Important outputs:

```text
suricata-events.jsonl
suricata-events.sha256
```

Determine:

```text
first seen
last seen
destination ports
protocols
event types
alerts
flows
```

---

## Phase 4 – Identify targeted services

Map destination ports to the exposed honeypots.

Examples:

```text
22 / 23      Cowrie
25 / 587     Mailoney
80           SNARE / Tanner
443          h0neytr4p
5060         SentryPeer
5432         Heralding
6379         Redis honeypot
3389         RDP honeypot
69 / 135 /
445 / 1433 /
1723 / 1883 /
3306 / 27017 Dionaea
```

Verify runtime mappings with:

```bash
sudo docker ps --format 'table {{.Names}}\t{{.Ports}}'
```

---

## Phase 5 – Collect service-specific evidence

Only investigate honeypot logs relevant to the ports actually targeted.

### Cowrie

Use:

```text
01-Cowrie-Attack-Investigation.md
```

Investigate:

```text
sessions
credentials
successful logins
commands
downloads
```

### Dionaea

Investigate the source in Dionaea logs when the activity targets services such as SMB, MQTT, TFTP or emulated database services.

Look for:

```text
connections
protocol requests
exploit behavior
payload transfer
captured malware
```

### Mailoney

Look for SMTP behavior such as:

```text
EHLO / HELO
MAIL FROM
RCPT TO
relay attempts
recipient probing
```

### SNARE / Tanner / h0neytr4p

Look for:

```text
HTTP methods
request paths
Host values
User-Agent
scanner fingerprints
exploit requests
```

### SentryPeer

Look for:

```text
REGISTER
INVITE
OPTIONS
SIP identities
extension probing
authentication attempts
```

### Heralding / Redis / RDP

Review the corresponding honeypot events and correlate them with Suricata.

---

## Phase 6 – Correlate events

Build a common timeline across:

```text
Suricata
honeypot logs
captured files
PCAP
```

Questions to answer:

```text
Did the source scan multiple ports?
Which service was contacted first?
Did activity progress beyond a connection attempt?
Were credentials attempted?
Was a command or exploit request sent?
Was a payload transferred?
Did the source return later?
```

---

## Phase 7 – Classify the observed activity

Useful evidence-based categories include:

```text
Single-port scan
Multi-port scan
Brute force
Credential attack
Protocol enumeration
Exploit attempt
Payload delivery
Malware upload/download
SMTP abuse
SIP abuse
Database probing
Unknown
```

Do not infer the real identity or physical location of the operator from the source IP alone.

---

## Phase 8 – Analyze captured files

If a honeypot captured a file, use:

```text
03-Malware-Static-Analysis.md
```

Collect:

```text
SHA-256
file type
architecture
metadata
ELF information
strings
URLs
domains
IP addresses
```

Do not execute captured samples.

---

## Phase 9 – Investigate infrastructure

Use:

```text
04-IP-OSINT.md
```

Collect:

```text
WHOIS
network owner
ASN
CIDR
reverse DNS
```

Treat this as infrastructure context, not proof of attacker identity.

---

## Phase 10 – PCAP analysis

Use:

```text
05-PCAP-Network-Analysis.md
```

Review both TCP and UDP.

Focus on the protocols and destination ports identified during triage.

---

## Phase 11 – Collect IOCs

Example structure:

```text
Source IP:

First seen:
Last seen:

Targeted ports:
Protocols:
Honeypots:

Additional IPs:
Domains:
URLs:

Usernames:
Passwords:

Captured files:
SHA-256:

Interesting requests / commands:

Suricata signatures:
Notes:
```

Only include fields supported by the evidence.

---

## Phase 12 – Write the final timeline

Translate raw telemetry into chronological observations.

Example:

```text
1. The source IP was first observed contacting multiple exposed services.

2. Suricata recorded connections to TCP/22, TCP/445 and UDP/5060.

3. Cowrie recorded repeated SSH authentication attempts.

4. Dionaea recorded SMB-related interaction from the same source.

5. SentryPeer recorded SIP probing.

6. The timestamps were correlated across the three services.

7. No payload transfer was observed.

8. The activity was documented as multi-service automated reconnaissance with credential probing.
```

The wording should describe what the evidence shows without overstating attribution or intent.

---

## Quick Reference

| What do I want to investigate? | File |
|---|---|
| Where should I start? | `00-T-Pot-Attack-Triage.md` |
| Targeted ports / protocols | `00-T-Pot-Attack-Triage.md`, `02-Suricata-Network-Investigation.md` |
| SSH / Telnet | `01-Cowrie-Attack-Investigation.md` |
| IDS alerts / flows | `02-Suricata-Network-Investigation.md` |
| HTTP / TLS / SSH metadata | `02-Suricata-Network-Investigation.md` |
| Malware hashes / ELF | `03-Malware-Static-Analysis.md` |
| WHOIS / ASN / rDNS | `04-IP-OSINT.md` |
| TCP / UDP packet analysis | `05-PCAP-Network-Analysis.md` |
| Full workflow | `06-T-Pot-Investigation-Workflow.md` |

---

## Suggested Case Directory

```text
analysis/
└── <source-ip>-<UTC-timestamp>/
    ├── suricata-events.jsonl
    ├── suricata-events.sha256
    ├── events.jsonl
    ├── events.sha256
    ├── timeline.tsv
    ├── commands.tsv
    ├── downloads.tsv
    ├── whois.txt
    ├── reverse-dns.txt
    ├── traffic.pcap
    ├── traffic.pcap.sha256
    └── iocs.txt
```

Not every investigation will contain every file. Cowrie-specific outputs, for example, are only needed when Cowrie was involved.
