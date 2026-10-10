# Assessment, evidence limits and publication scope

## Assessment

| Finding | Assessment |
|---|---|
| SMB1 negotiation and IPC$ request in the recording | Directly observed |
| Large NT transaction followed by 16 TRANS2_SECONDARY messages | Directly observed |
| Crafted and malformed FEA data | Directly observed in static parsing |
| EternalBlue-like exploitation attempt | Strongly supported inference |
| Successful code execution or system compromise | Not established |
| WannaCry or DoublePulsar infection | Not established |
| Identity of a person or group | Not established |

## Correlation limits

The network alert's IPC$ request payload matches the corresponding bistream message byte-for-byte. Source endpoint metadata and protocol sequence also support association. However, the bistream filename timestamp precedes the application acceptance event by approximately 13 seconds. The recording has no per-message timestamps; the cause of this difference has not been verified.

Suricata also reported a stream reassembly gap in the investigated network flow. The network telemetry and application recording have different coverage and counting semantics. Missing parsed commands and byte-count differences must not be silently corrected or treated as identical measurements.

The IOC-like source address is a network observation, not actor attribution. Other sessions from the same source remain separate unless additional evidence connects them. An unrelated source's DoublePulsar alert was excluded.

## What the evidence cannot show

- Whether the target would have been exploitable as a real Windows SMB server.
- Whether any malicious code executed or persistence was created.
- Whether other connections performed heap grooming or delivered a later payload.
- Why the timestamp discrepancy occurred.
- Whether an absent log entry means no activity or incomplete visibility.

No completed search for all related connections, host execution evidence, or follow-on downloads is claimed. These are optional follow-ups, not prerequisites for documenting the supported findings.

## Public release scope

This case publishes interpreted findings only. Source and destination IPs, original timestamps, ephemeral ports, flow and document identifiers, GUIDs, GeoIP/ASN details, local usernames, hostnames, management settings, absolute filesystem paths and raw artifact hashes are omitted. Endpoints use role labels and the timeline uses relative times.

No screenshots, raw JSON/Base64 payloads, packet captures, bistream files, credentials, binary samples or private-repository links are included. Protocol names, standard service port TCP/445 and aggregate message/data counts are retained because they explain the analysis without revealing the environment.

This sanitization statement applies to this case and its index additions, not a retrospective security audit of the entire repository.
