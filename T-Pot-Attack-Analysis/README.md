# T-Pot Attack Analysis

This directory contains repeatable workflows for investigating attacks captured by the T-Pot honeypot environment.

The lab exposes multiple honeypot services, so investigations should not begin by assuming that every source IP belongs to a Cowrie SSH case.

Start with:

[00 - T-Pot Attack Triage](00-T-Pot-Attack-Triage.md)

The triage step identifies which ports and honeypots were targeted before moving into service-specific analysis.

---

## Investigation Flow

```text
Source IP
    |
    v
Attack triage
    |
    v
Targeted ports / services
    |
    +--> Cowrie        SSH / Telnet
    +--> Mailoney      SMTP
    +--> Dionaea       SMB / MQTT / databases / other services
    +--> SNARE/Tanner  HTTP
    +--> h0neytr4p     HTTPS
    +--> SentryPeer    SIP
    +--> Heralding     PostgreSQL
    +--> Redis honeypot
    +--> RDP honeypot  RDP / NLA / NTLM telemetry
    |
    v
Suricata correlation
    |
    v
PCAP / malware / OSINT
    |
    v
IOCs and final timeline
```

---

## Documents

| Document | Purpose |
|---|---|
| [00 - T-Pot Attack Triage](00-T-Pot-Attack-Triage.md) | Identify targeted ports, protocols and relevant honeypots |
| [01 - Cowrie Attack Investigation](01-Cowrie-Attack-Investigation.md) | SSH and Telnet sessions, credentials, commands and downloads |
| [02 - Suricata Network Investigation](02-Suricata-Network-Investigation.md) | Network-wide correlation across exposed services |
| [03 - Malware Static Analysis](03-Malware-Static-Analysis.md) | Static analysis of captured payloads |
| [04 - IP OSINT](04-IP-OSINT.md) | WHOIS, ASN, reverse DNS and infrastructure context |
| [05 - PCAP Network Analysis](05-PCAP-Network-Analysis.md) | Packet-level TCP/UDP analysis |
| [06 - T-Pot Investigation Workflow](06-T-Pot-Investigation-Workflow.md) | End-to-end investigation process |
| [07 - RDPHoneypot Investigation](07-RDPHoneypot-Investigation.md) | RDP sessions, NLA/NTLM telemetry, usernames and Suricata correlation |
| [Manual workflow](manual-workflow/README.md) | Stage-by-stage manual equivalent of the automated IP-report collector |

The existing guides above are retained as detailed reference material. The `manual-workflow/` directory does **not** replace them: it explains the complete evidence-collection and analysis sequence in the same small stages used by the automated collector.

---

## Collector update: v3.4

The v3.4 collector update adds dedicated RDPHoneypot analysis, RDP-specific behavior flags and safer IOC extraction.

Important changes:

- `RDPHONEYPOT ANALYSIS` section in `report.txt`,
- counts for connections, logins, closes and unique sessions,
- NLA authentication methods, usernames, domains and session durations,
- `<NOT_CAPTURED>` instead of treating missing RDP plaintext passwords as empty passwords,
- RDP/NTLM challenge-response values are no longer treated as malware hashes,
- event/protocol strings such as `rdphoneypot.login` and `ms.rdp.established` are not promoted to domain IOCs,
- behavior flags for high-volume RDP traffic, repeated authentication activity, Administrator targeting and short-lived sessions.

See [07 - RDPHoneypot Investigation](07-RDPHoneypot-Investigation.md) for interpretation details.

---

## Manual vs automated workflow

The same source IP can be investigated in two complementary ways.

Automated collection:

```bash
tpot-ip-report <IP> [days]
```

Manual learning / verification path:

```text
Preparation
    -> Suricata
    -> Cowrie / Dionaea / other services
    -> p0f
    -> system and indexed context
    -> unified timeline
    -> credentials and IOCs
    -> PCAP / artifacts when available
    -> campaign correlation when useful
    -> integrity manifest and bundle
    -> final assessment
```

See [Manual T-Pot Investigation Workflow](manual-workflow/README.md) for the full stage mapping, commands, interpretation notes and expected output files.

---

## Current Service Mapping

| Port | Protocol | Honeypot / Service |
|---:|---|---|
| 22 | TCP | Cowrie / SSH |
| 23 | TCP | Cowrie / Telnet |
| 25 | TCP | Mailoney / SMTP |
| 69 | UDP | Dionaea / TFTP |
| 80 | TCP | SNARE / HTTP |
| 135 | TCP | Dionaea / RPC |
| 443 | TCP | h0neytr4p / HTTPS |
| 445 | TCP | Dionaea / SMB |
| 587 | TCP | Mailoney / SMTP Submission |
| 1433 | TCP | Dionaea / MSSQL |
| 1723 | TCP | Dionaea / PPTP |
| 1883 | TCP | Dionaea / MQTT |
| 3306 | TCP | Dionaea / MySQL |
| 3389 | TCP | RDPHoneypot / RDP |
| 5060 | TCP/UDP | SentryPeer / SIP |
| 5432 | TCP | Heralding / PostgreSQL |
| 6379 | TCP | Redis honeypot / Redis |
| 27017 | TCP | Dionaea / MongoDB |

This mapping is used for triage. Actual container state and port mappings should still be verified during an investigation.

---

## Recommended Starting Point

For a newly observed source IP:

```text
1. Create a case directory
2. Export Suricata events
3. Identify destination ports and protocols
4. Map ports to honeypots
5. Inspect relevant honeypot logs
6. Correlate timestamps
7. Continue with malware, OSINT or PCAP when useful
8. Produce IOCs and a final timeline
```
