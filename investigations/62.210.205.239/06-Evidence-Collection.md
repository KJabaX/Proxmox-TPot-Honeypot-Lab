# 6. Evidence Collection

## Collector

The case was collected using the `tpot-ip-report` workflow with a seven-day window.

Example usage:

```bash
tpot-ip-report 62.210.205.239 7
```

The collector is designed to minimize impact on the honeypot VM by using:

```text
nice -n 19
ionice -c 3
```

and by avoiding repeated full passes over the same large log source where possible.

## Bundle Contents

The generated investigation bundle contained:

```text
report.txt
timeline.tsv.gz
campaign-context.tsv
campaign-password-similarity.tsv
iocs.tsv
credentials.tsv
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

No relevant PCAP or captured malware artifact was present in this case.

## Evidence Integrity

The bundle includes `SHA256SUMS` so that extracted evidence files can be checked for integrity after collection or transfer.

The raw matching events are retained alongside the human-readable report. This allows conclusions in the documentation to be verified against the underlying telemetry without rerunning the full collection.

## Data Sources Used

### Dionaea

Used to identify:

- MSSQL protocol activity,
- destination port,
- username and password attempts,
- attack timing,
- credential-attempt rate,
- related source activity.

### Suricata

Used to identify:

- network flows,
- IDS alerts,
- target port and transport,
- packet and byte volume,
- network-level start/end times.

### p0f

Used for passive TCP fingerprint context. p0f results are treated as heuristic observations rather than definitive host attribution.

### Campaign Context

The Dionaea stream was used to construct a bounded campaign sketch while the log source was already being read. This avoids a separate complete pass over the same Dionaea history solely for peer discovery.

### Elasticsearch

The bundle contained a best-effort Elasticsearch count result for the target IP. Elasticsearch is useful as an indexed source for future investigations, but the raw honeypot/Suricata evidence remains the primary evidentiary source for this case.

## Collection Limitations

The collector can only report data that was retained by T-Pot and available during the requested time window.

It cannot reconstruct:

- expired or deleted logs,
- packets that were never captured,
- encrypted application content that was not decrypted,
- artifacts that the honeypot did not save.

The investigation documentation therefore distinguishes between **not observed** and **proven absent**.
