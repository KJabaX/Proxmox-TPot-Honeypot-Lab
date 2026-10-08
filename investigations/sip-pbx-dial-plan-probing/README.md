# SIP/PBX Dial-Plan Probing Investigation

## Summary

This case documents sustained automated SIP activity observed by a T-Pot honeypot.

The source repeatedly sent SIP `INVITE` requests to UDP/5060 while testing multiple dialing representations of the same destination. The dominant variants differed mainly by international and PBX-style prefixes.

The behavior is strongly consistent with **automated PBX dial-plan probing associated with toll-fraud reconnaissance**.

This public version is intentionally sanitized. It excludes the source address, honeypot addressing, exact called numbers, raw event identifiers, infrastructure-specific paths, and other environment details that are not needed to understand the investigation.

## Final Metrics

| Metric | Result |
|---|---:|
| SIP INVITE events | 7,023 |
| SentryPeer events | 7,023 |
| SIP methods observed | INVITE only |
| Unique called-number variants | 25 |
| Dominant six variants | 6,838 events |
| Share of dominant six variants | 97.37% |
| Observed activity duration | ~5 h 18 min |
| Peak rate | 76 INVITEs/min |

## Investigation Result

The strongest evidence was the combination of:

- repeated SIP `INVITE` requests,
- valid SIP/SDP call-setup structure,
- systematic dialing-prefix variation,
- near-identical counts across the six dominant variants,
- matching activity visible in both Suricata and SentryPeer telemetry.

Because the target was a honeypot rather than a production PBX, this case evaluates the **attempted behavior** rather than successful external call routing.

## Assessment

**Classification:** Automated SIP/PBX dial-plan probing  
**Likely objective:** Discovery of accepted outbound or international call-routing formats  
**Toll-fraud reconnaissance confidence:** High  
**Attribution:** Not established

## Case Files

| File | Purpose |
|---|---|
| [01-SIP-Activity.md](01-SIP-Activity.md) | SIP/INVITE and Suricata analysis |
| [02-Dial-Plan-Analysis.md](02-Dial-Plan-Analysis.md) | Called-number and prefix-pattern analysis |
| [03-Timeline.md](03-Timeline.md) | Duration and event-rate analysis |
| [04-Evidence-and-Limitations.md](04-Evidence-and-Limitations.md) | Evidence quality, queries and limitations |
