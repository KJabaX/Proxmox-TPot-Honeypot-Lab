# Manual T-Pot Investigation Workflow

This directory expands the automated `tpot-ip-report <IP> [days]` workflow into small, reviewable investigation stages.

The purpose is not to replace the existing `00`-`06` guides in the parent directory. Those documents remain the detailed reference guides for triage, Cowrie, Suricata, malware, OSINT, PCAP and the end-to-end investigation process.

This manual workflow answers a different question:

> What does the automated collector do, stage by stage, and how can the same evidence be collected and interpreted manually?

## Design principles

- preserve raw evidence before interpretation,
- use a bounded time window,
- avoid repeatedly rescanning large compressed logs,
- keep collection read-only,
- hash exported evidence,
- distinguish **not observed** from **proven absent**,
- correlate multiple data sources before drawing conclusions,
- do not attribute an attacker beyond what the evidence supports.

For large T-Pot installations, use low-priority execution where practical:

```bash
nice -n 19 <command>
ionice -c 3 <command>
```

## Workflow

```text
Source IP + time window
        |
        v
00 Preparation and case directory
        |
        v
01 Suricata evidence
        |
        +--> 02 Cowrie
        +--> 03 Dionaea
        +--> 04 p0f
        +--> 05 Other honeypots
        |
        v
06 System / Elasticsearch context
        |
        v
07 Unified timeline
        |
        v
08 Credentials and IOCs
        |
        +--> 09 PCAP
        +--> 10 Artifacts / malware
        |
        v
11 Campaign correlation
        |
        v
12 Evidence integrity and bundle
        |
        v
13 Final assessment
```

## Automated-to-manual mapping

| Automated collector function | Manual stage |
|---|---|
| Case setup and bounded collection | [00 - Preparation and Case Directory](00-Preparation-and-Case-Directory.md) |
| Suricata flows, ports, alerts and protocol metadata | [01 - Suricata Evidence Collection](01-Suricata-Evidence-Collection.md) |
| SSH/Telnet sessions, logins, commands and downloads | [02 - Cowrie Evidence Analysis](02-Cowrie-Evidence-Analysis.md) |
| Dionaea database/service events and credentials | [03 - Dionaea Evidence Analysis](03-Dionaea-Evidence-Analysis.md) |
| Passive TCP fingerprint context | [04 - p0f Fingerprinting](04-P0F-Fingerprinting.md) |
| Remaining T-Pot services | [05 - Other Honeypot Services](05-Other-Honeypot-Services.md) |
| Docker/system snapshot and indexed fallback | [06 - System and Elasticsearch Context](06-System-and-Elasticsearch-Context.md) |
| Cross-source chronological reconstruction | [07 - Unified Timeline](07-Unified-Timeline.md) |
| Credential statistics and IOC extraction | [08 - IOC and Credential Extraction](08-IOC-and-Credential-Extraction.md) |
| Existing packet captures | [09 - PCAP Analysis](09-PCAP-Analysis.md) |
| Captured payloads and file metadata | [10 - Artifacts and Malware](10-Artifacts-and-Malware.md) |
| Related sources and behavioral similarity | [11 - Campaign Correlation](11-Campaign-Correlation.md) |
| SHA-256 manifest and investigation archive | [12 - Evidence Integrity and Bundle](12-Evidence-Integrity-and-Bundle.md) |
| Evidence-based conclusion and limitations | [13 - Final Assessment](13-Final-Assessment.md) |

## Typical case outputs

A complete investigation may produce files such as:

```text
report.txt
timeline.tsv.gz
iocs.tsv
credentials.tsv
campaign-context.tsv
campaign-password-similarity.tsv
evidence/suricata.jsonl.gz
evidence/dionaea.jsonl.gz
evidence/cowrie.jsonl.gz
evidence/p0f-matches.jsonl.gz
evidence/other-service-matches.tsv.gz
pcap-manifest.tsv
artifact-metadata.tsv
artifact-references.tsv
docker-ps.txt
system-snapshot.txt
elasticsearch-fallback.json
SHA256SUMS
```

Not every case will contain every file. For example, a case may have no relevant Cowrie events, PCAP or captured artifact.

## Relationship to the parent guides

Use the manual workflow for repeatability and understanding of the collector. Use the parent guides when a stage requires deeper analysis:

- `../01-Cowrie-Attack-Investigation.md`
- `../02-Suricata-Network-Investigation.md`
- `../03-Malware-Static-Analysis.md`
- `../04-IP-OSINT.md`
- `../05-PCAP-Network-Analysis.md`
- `../06-T-Pot-Investigation-Workflow.md`
