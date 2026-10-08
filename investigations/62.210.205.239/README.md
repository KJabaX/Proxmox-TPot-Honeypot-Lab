# Investigation: 62.210.205.239

## Summary

This investigation documents high-volume automated activity from source IP `62.210.205.239` against the T-Pot Dionaea MSSQL honeypot on TCP port `1433`.

The available evidence is consistent with an **automated MSSQL credential dictionary attack** targeting the SQL Server administrator account `sa`.

During the observed activity:

- Dionaea recorded **250,538 events** from the source.
- **250,529 events contained credentials**.
- The only observed username was `sa`.
- **250,304 unique password values** were attempted.
- Suricata recorded **252,869 flow events** and **6,150 alerts**.
- **6,145 Suricata alerts** were `ET SCAN Suspicious inbound to MSSQL port 1433`.
- No Cowrie activity was associated with the source.
- No payload download, malware artifact, command execution or post-authentication activity was identified in the collected evidence.

The source was observed by Dionaea from `2026-10-02 18:43:04 UTC` until `2026-10-03 21:51:50 UTC`.

The investigation also identified a group of six additional high-volume sources with nearly identical service, port, username and timing characteristics. Elastic dashboard enrichment associated this group with **ASN 12876 / Scaleway SAS**. This is consistent with a coordinated or common-tool campaign, but does not prove common human ownership or attribution.

## Assessment

**Classification:** MSSQL credential guessing / dictionary brute force
**Target service:** MSSQL
**Target port:** TCP/1433
**Target username:** `sa`
**Automation confidence:** High
**Evidence of successful compromise:** None observed
**Evidence of payload delivery:** None observed

Dionaea `connection.type=accept` records indicate that the honeypot accepted TCP/application connections. They must not be interpreted as successful authentication of the supplied credentials.

## Investigation Files

| File | Purpose |
|---|---|
| [01-Dionaea-MSSQL-Analysis.md](01-Dionaea-MSSQL-Analysis.md) | MSSQL credential activity and attack-rate analysis |
| [02-Suricata-Network-Analysis.md](02-Suricata-Network-Analysis.md) | Network flows and IDS alerts |
| [03-Attack-Timeline.md](03-Attack-Timeline.md) | Chronological attack timeline |
| [04-Campaign-Correlation.md](04-Campaign-Correlation.md) | Related source and campaign-level analysis |
| [05-Indicators-and-Limitations.md](05-Indicators-and-Limitations.md) | Reliable indicators, caveats and evidence limitations |
| [06-Evidence-Collection.md](06-Evidence-Collection.md) | Collection method and investigation bundle contents |

## Evidence Source

The case was collected with the `tpot-ip-report` workflow using a seven-day window. The investigation bundle included:

- Suricata events
- Dionaea events
- p0f observations
- unified timeline
- credential summary
- campaign context
- campaign password-similarity sketch
- IOC extraction
- Docker and system snapshots
- SHA-256 evidence manifest

No relevant PCAP or captured malware artifact was available in the generated bundle.
