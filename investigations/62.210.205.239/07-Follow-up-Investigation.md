# 7. Follow-up Investigation

The current evidence is sufficient to classify the activity from `62.210.205.239` as automated MSSQL credential guessing. Additional work should therefore focus on **campaign correlation and infrastructure context**, not on proving the basic attack type again.

## Priority 1 — Exact Peer Correlation

The six strongest peer candidates should be compared directly:

```text
195.154.176.7
62.210.124.129
62.210.204.18
62.210.205.126
195.154.167.21
195.154.176.27
```

Useful comparisons:

- exact credential ordering,
- password-set overlap,
- whether different IPs used different sections of one larger dictionary,
- per-second and per-minute timing,
- pause/resume times,
- source-port behavior,
- p0f/TCP fingerprints.

The current bounded password sketch reported little direct overlap. Because the high-volume sources may be using **sharded portions of one dictionary**, an exact sequence/set comparison would be more informative than the current sketch.

This should preferably be performed from indexed Elasticsearch data or in one bounded streaming pass rather than rescanning all compressed Dionaea logs separately for every IP.

## Priority 2 — Cross-Service Search for Peer Sources

Check whether the related IPs also interacted with other exposed services before, during or after the MSSQL campaign.

Questions to answer:

- Did the same sources probe SSH/Telnet, SMB, SIP, SMTP, Redis or web honeypots?
- Did they perform reconnaissance before beginning MSSQL credential guessing?
- Did any peer source attempt payload delivery through another service?

A cross-service hit would add useful campaign context even if `62.210.205.239` itself remained MSSQL-only.

## Priority 3 — Infrastructure Enrichment

For the seven-source group, document:

- ASN and hosting organization,
- reverse DNS where available,
- public service exposure,
- reputation / scan-source classification,
- historical or passive-DNS context where available.

The current Elastic enrichment associates the main group with ASN 12876 / Scaleway SAS. This supports shared infrastructure context but does not establish common ownership.

## Priority 4 — Recurrence Monitoring

The target source stopped producing Dionaea events on `2026-10-03` in the collected evidence. Future observations should check whether:

- the same IP returns,
- the six peer addresses return together,
- the campaign shifts to different addresses but retains the same timing/profile,
- the target username or password corpus changes.

A recurring behavioral profile is often more useful for campaign tracking than a static IP list.

## Priority 5 — Future Packet Capture

No relevant PCAP was available for this case. If storage permits, a bounded rolling packet-capture strategy for selected honeypot ports could provide protocol-level evidence for future campaigns.

PCAP collection should remain size- and retention-limited so that packet capture does not become a new resource bottleneck on the T-Pot VM.

## Current Investigation Status

No additional analysis is required to establish the primary conclusion:

> `62.210.205.239` performed a high-volume automated MSSQL `sa` credential-dictionary attack against TCP/1433.

The most valuable next investigation is **campaign-level comparison of the six related high-volume sources**, especially exact credential-list partitioning and synchronized timing.
