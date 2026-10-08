## Suricata Network Analysis

Suricata network events involving the source IP `103.174.102.29` were extracted from the T-Pot EVE JSON logs.

A total of **1,772 events** were identified:

* **706 SSH events**
* **706 flow events**
* **357 alerts**
* **3 anomaly events**

The observed traffic was TCP traffic targeting SSH port `22`.

### IDS Alerts

The Suricata alerts consisted primarily of:

```text
236  ET INFO SSH-2.0-Go version string Observed in Network Traffic
118  ET INFO SSH session in progress on Expected Port
2    SURICATA STREAM spurious retransmission
1    SURICATA STREAM Packet with invalid timestamp
```

The `SSH-2.0-Go` identification is consistent with the SSH client version observed in the Cowrie logs.

### Cowrie / Suricata Correlation

The successful Cowrie session `db841bd9b899` originated from source port `58394`.

Cowrie recorded the connection at:

```text
2026-09-22T09:59:54.464274Z
```

Suricata recorded traffic from the same source IP and source port at:

```text
2026-09-22T09:59:54.462634+0000
103.174.102.29:58394 -> <honeypot-ip>:22
```

Suricata identified the connection as using an `SSH-2.0-Go` client.

The very close timestamps and identical source port strongly correlate the Cowrie and Suricata records to the same SSH connection.

Suricata observed the destination as `<honeypot-ip>:22`, while Cowrie recorded the connection reaching its container-side address `<cowrie-container-ip>:22`. This is consistent with the different network observation points in the T-Pot and container environment.

### DNS and HTTP Activity

No DNS events involving the investigated IP were identified.

No HTTP events involving the investigated IP were identified.

Combined with the Cowrie data, no evidence of HTTP-based payload retrieval or DNS activity was observed during the investigated activity.

### Stream Anomalies

Three Suricata stream anomalies were observed:

* Two `SURICATA STREAM spurious retransmission` events
* One `SURICATA STREAM Packet with invalid timestamp` event

These events were recorded for the SSH traffic but do not by themselves demonstrate exploitation or malicious post-authentication behavior.

### Network Assessment

The Suricata evidence independently supports the Cowrie findings.

The network activity was dominated by repeated SSH connections to TCP port 22 using an SSH client identified as `SSH-2.0-Go`.

No HTTP or DNS activity was observed for the source IP in the extracted dataset.

Together, Cowrie and Suricata provide evidence consistent with an automated SSH credential dictionary attack rather than a web-based or payload-download attack.
