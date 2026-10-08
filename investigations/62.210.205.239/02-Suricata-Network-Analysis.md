# 2. Suricata Network Analysis

## Summary

Suricata recorded **259,019 events** associated with `62.210.205.239` during the collected window.

| Event type | Count |
|---|---:|
| Flow | 252,869 |
| Alert | 6,150 |

The activity was overwhelmingly associated with **TCP/1433**, matching the Dionaea MSSQL observations.

## Targeted Service

The dominant network pattern was:

```text
Source: 62.210.205.239
Transport: TCP
Destination port: 1433
Service: MSSQL
```

The collected Suricata event distribution included:

```text
250586  1433/TCP  app_proto=failed  flow
  6146  1433/TCP                    alert
  2283  1433/TCP                    flow
     4  1433/TCP  app_proto=failed  alert
```

`app_proto=failed` is Suricata application-protocol detection status. It must not be interpreted as a failed MSSQL authentication result.

## IDS Alerts

The dominant signature was:

```text
ET SCAN Suspicious inbound to MSSQL port 1433
```

Observed count:

```text
6145 alerts
```

A further five events were:

```text
SURICATA STREAM spurious retransmission
```

The IDS alert count is much lower than the Dionaea credential count because Suricata alerts are detections, not a one-to-one count of authentication attempts.

## Flow Volume

Suricata flow totals:

| Metric | Value |
|---|---:|
| Flow events | 252,869 |
| Packets to server | 2,022,718 |
| Packets to client | 1,758,699 |
| Bytes to server | 194,843,718 |
| Bytes to client | 147,414,694 |

The large number of short flows is consistent with an automated client repeatedly creating MSSQL connections for credential testing.

## Time Range

Suricata first observed the source at:

```text
2026-10-02T18:43:04.679031+0000
```

The last Suricata event in the bundle was:

```text
2026-10-03T21:57:24.296980+0000
```

Dionaea credential activity ended a few minutes earlier at approximately `21:51:50 UTC`, while Suricata still observed related network events afterwards.

## Correlation with Dionaea and p0f

The evidence sources align closely:

```text
Suricata flow events: 252,869
Dionaea events:       250,538
Dionaea credentials:  250,529
p0f client SYN count:  252,870
```

This close agreement provides strong cross-source support that the activity represents a very large number of separate inbound TCP connection attempts rather than a dashboard counting artifact.

## Assessment

Suricata independently supports the Dionaea conclusion: the source generated sustained and highly automated MSSQL-targeted traffic against TCP/1433.

No Suricata evidence in the collected bundle showed the source targeting another exposed service during this case.
