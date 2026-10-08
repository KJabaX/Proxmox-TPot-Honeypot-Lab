# 1. SIP Activity

## Observed Protocol Behavior

Suricata parsed repeated SIP traffic directed at the honeypot service on UDP/5060.

The dominant SIP method was:

```text
INVITE
```

Representative events contained valid SIP/2.0 request structure and SDP media-session metadata.

This is more specific than a generic UDP/5060 scan. The source was generating actual SIP call-setup requests.

## Why This Matters

A simple port scan only establishes that a service might be reachable.

Here, the telemetry showed:

- SIP protocol parsing,
- repeated `INVITE` requests,
- SDP audio-session metadata,
- sustained activity over several hours.

This indicates automated interaction with what the source believed was a SIP/PBX service.

## Final Counts

Final Suricata aggregation:

| Metric | Result |
|---|---:|
| SIP INVITEs | 7,023 |
| SIP methods observed | INVITE only |
| Peak INVITEs/minute | 76 |
| Observed duration | ~5 h 18 min |

## Example ES|QL Pattern

The public case keeps the source anonymized:

```text
FROM logstash-*
| WHERE event_type == "sip"
  AND sip.method == "INVITE"
| STATS
    invites = COUNT(*),
    first_seen = MIN(@timestamp),
    last_seen = MAX(@timestamp)
```

In the private investigation, the query was additionally scoped to a single observed source.

## Interpretation

The protocol evidence supports automated SIP call-setup attempts rather than only service discovery.

The dial-plan pattern documented in the next section provides the strongest indication of the likely objective.
