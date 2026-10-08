# 13 - Final Assessment

The final assessment turns collected evidence into a concise, defensible conclusion.

## Recommended structure

Document:

1. **Observed activity** — what the source actually did.
2. **Primary evidence** — which logs support the conclusion.
3. **Time window** — first and last observed timestamps.
4. **Targeted services** — ports/protocols/honeypots.
5. **Behavior classification** — scanning, credential guessing, command execution, payload delivery, etc.
6. **Indicators** — IPs, domains, URLs, hashes and credentials that matter.
7. **Limitations** — missing PCAP, encrypted traffic, retention gaps, unavailable artifacts.
8. **Confidence** — what is established versus inferred.

## Example conclusion pattern

```text
The source <IP> generated repeated application-level credential attempts
against <service/port> during <time window>. Honeypot application logs and
Suricata network telemetry correlate in time and target. The observed rate
and repeated credential corpus are consistent with automated credential
guessing.
```

Keep conclusions evidence-based. Avoid actor attribution unless independent evidence supports it.

## When is a single-IP investigation complete?

A case is normally complete when you can answer:

- what happened,
- when it happened,
- which service was targeted,
- how the activity was performed,
- which evidence supports that conclusion,
- whether commands/downloads/payloads were observed,
- what important evidence was unavailable,
- whether additional work would materially change the primary conclusion.

If the remaining work is mainly comparison with other source IPs, move that work into a separate campaign-level investigation instead of leaving the single-IP case permanently open.
