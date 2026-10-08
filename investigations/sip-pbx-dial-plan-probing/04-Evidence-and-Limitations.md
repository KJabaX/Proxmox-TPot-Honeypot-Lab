# 4. Evidence and Limitations

## Evidence Sources

The investigation correlated two telemetry sources:

### Suricata

Provided:

- network-level source and destination context,
- UDP transport information,
- SIP protocol parsing,
- SIP method,
- SIP URI and request structure,
- SDP media metadata,
- timestamps and flow context.

### SentryPeer

Provided:

- honeypot application context,
- called-number data,
- repeated dialing variants,
- timestamps and event identifiers.

The agreement between these sources strengthened the interpretation.

## Final Evidence Summary

| Metric | Result |
|---|---:|
| SIP INVITEs | 7,023 |
| SentryPeer events | 7,023 |
| SIP methods observed | INVITE only |
| Unique called-number variants | 25 |
| Dominant six variants | 6,838 |
| Peak INVITEs/minute | 76 |
| Duration | ~5 h 18 min |

The matching aggregate counts across Suricata and SentryPeer are strong cross-source consistency, but they are not presented as proof of strict one-to-one event pairing without event-level correlation.

## Confidence

| Conclusion | Confidence |
|---|---|
| Automated SIP activity | High |
| SIP INVITE call-setup attempts | High |
| Systematic dial-plan probing | High |
| Toll-fraud reconnaissance objective | High |
| Attribution to a specific person or group | Not established |

## Limitations

This public version intentionally excludes:

- source IP address,
- honeypot public and private addresses,
- exact called numbers,
- raw SIP request lines containing environment details,
- internal hostnames,
- management ports,
- filesystem paths,
- raw evidence bundles,
- credentials or unrelated captured data.

GeoIP and ASN data are also omitted because they do not materially improve the public technical explanation and can encourage over-attribution.

## Public vs. Private Documentation

The private case retains the full evidence record needed for reproducibility.

The public case keeps only the technical observations required to demonstrate the investigation method, telemetry correlation, ES|QL analysis and evidence-based conclusion.
