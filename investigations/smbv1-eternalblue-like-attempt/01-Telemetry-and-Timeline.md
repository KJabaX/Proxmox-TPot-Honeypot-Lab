# Telemetry and relative timeline

## From alert to stream inspection

The initial network alert was `GPL NETBIOS SMB-DS IPC$ share access` (SID 2102465), with informational metadata and severity 3. It identified an IPC$ request; it did not by itself identify EternalBlue or prove a successful attack.

The associated Dionaea record showed an accepted TCP/445 connection handled by `smbd`. An application-level `accept` event records connection acceptance, not credential validity or code execution.

The network view contained TRANS2_SECONDARY activity and a stream reassembly gap. This justified inspecting the application recording rather than assigning an exploit family from the command name alone.

## Relative timeline

T0 is the Suricata flow start. These are log timestamps, not reconstructed packet timings.

| Relative time | Source | Observation |
|---|---|---|
| T0 | Suricata | Network flow starts |
| T0 + 28 ms | Dionaea | SMB TCP connection accepted |
| T0 + 109 ms | Suricata | IPC$ request alert |
| T0 + 177 ms | Suricata | TRANS2_SECONDARY event; stream-gap alert also observed |

The bistream filename timestamp is about 13 seconds earlier than the Dionaea acceptance timestamp. It is kept separate from this timeline because its meaning and clock relationship have not been verified.

## Correlation

The exact IPC$ request payload in the alert matches the corresponding message in the bistream. The original source endpoint metadata also matches. This is stronger evidence than shared source IP alone, but the timestamp discrepancy remains an explicit limitation.

Suricata and Dionaea report different destination addresses. Container networking/address translation is a plausible explanation, but the historical mapping was not independently verified.

A passive fingerprint record has `subject=srv` and `mod=syn+ack`. It concerns the responding server side and must not be used to label the remote client OS.

## Separate observations

A short negotiation-only session and a retransmission alert from other times were examined during triage. They are not merged into this exploitation sequence. Dashboard event counts are not attack counts, and panels grouped by a field may omit records without that field.
