# 5. Indicators and Limitations

## Reliable Case Indicators

The following indicators are directly supported by the collected evidence:

| Indicator | Value |
|---|---|
| Primary source IP | `62.210.205.239` |
| Target service | MSSQL |
| Target port | `1433/TCP` |
| Target username | `sa` |
| Primary behavior | Automated credential guessing / dictionary brute force |
| Suricata signature | `ET SCAN Suspicious inbound to MSSQL port 1433` |
| First Dionaea event | `2026-10-02T18:43:04.719456Z` |
| Last Dionaea event | `2026-10-03T21:51:50.979651Z` |

Candidate related sources identified through campaign correlation:

```text
195.154.176.7
62.210.124.129
62.210.204.18
62.210.205.126
195.154.167.21
195.154.176.27
```

These addresses showed highly similar MSSQL/1433/`sa` behavior and overlapping timing. They are campaign candidates, not confirmed common ownership.

## IOC Extraction Caveat

The generic IOC extraction in the investigation bundle produced many values that look syntactically like domains or hashes but originated inside the attacker's password dictionary.

Examples of password strings can contain periods or hex-looking values and therefore match generic IOC regular expressions.

For this case, the automatically generated `iocs.tsv` should **not** be treated as a validated IOC list without source-context review.

In particular:

- domain-like values extracted from credential/password fields may be false positives,
- hash-like values extracted from credential/password fields may be false positives,
- internal honeypot addresses are evidence context, not attacker IOCs.

A future collector version should exclude credential fields from generic IOC extraction unless a value is independently observed in URL, DNS, file, payload or network metadata.

## Evidence Limitations

### No PCAP

No relevant filtered packet capture was available in the generated bundle. Therefore the investigation cannot independently reconstruct the full MSSQL wire exchange from packet payloads.

### No Malware Artifact

No captured payload or malware artifact was linked to the source. This means there is no sample available for static or dynamic malware analysis in this case.

### No Successful Authentication Evidence

Dionaea records `connection.type=accept`, but this means the honeypot accepted the connection. It does not demonstrate that a tested password would have authenticated successfully against a real SQL Server.

### p0f Fingerprinting

p0f produced a recurring passive fingerprint compatible with older Linux TCP-stack characteristics, including a distance estimate around 14 hops.

Passive OS fingerprinting is heuristic and should not be used as definitive operating-system attribution.

### Infrastructure Attribution

ASN and geolocation identify source infrastructure, not the human operator. Hosting infrastructure may be rented, proxied, compromised or shared.

## Confidence Summary

| Conclusion | Confidence |
|---|---|
| Automated activity | High |
| MSSQL credential guessing | High |
| Targeting of `sa` | High |
| Dictionary-style attack | High |
| Same campaign as six high-volume peers | Moderate to high |
| Same human operator for all peer IPs | Unknown |
| Successful compromise | Not observed |
| Malware delivery | Not observed |
