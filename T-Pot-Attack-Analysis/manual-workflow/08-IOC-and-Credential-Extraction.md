# 08 - IOC and Credential Extraction

This stage converts the normalized case evidence into reusable indicators and credential statistics.

## Credentials

From Cowrie or Dionaea exports, extract `(service, username, password)` and count repeated combinations.

Example Cowrie pattern:

```bash
zcat "$CASE/evidence/cowrie.jsonl.gz" 2>/dev/null |
jq -r '
  select(.eventid == "cowrie.login.failed" or .eventid == "cowrie.login.success") |
  ["cowrie", (.username // ""), (.password // "")] | @tsv
' |
sort | uniq -c | sort -nr
```

Save the normalized result as:

```text
credentials.tsv
```

Recommended columns:

```text
service\tusername\tpassword\tcount
```

### RDP / NLA exception

Do not interpret an empty RDPHoneypot `password` field as an empty plaintext password.

RDP NLA commonly records NTLM challenge-response data instead:

```text
nt_challenge
nt_proof
nt_response
hashcat_line
```

For normalized output, record the observed username but use an explicit value such as:

```text
<NOT_CAPTURED>
```

for the plaintext-password field.

## IOCs

Search the case evidence for indicators that were actually observed, such as:

- source/destination IPs,
- domains,
- URLs,
- email addresses,
- malware/download file hashes.

Recommended output:

```text
iocs.tsv
```

with columns such as:

```text
type\tvalue
```

or, for a richer manual case file:

```text
type\tvalue\tsource\tcontext
```

## Avoid false IOCs

Do not promote every domain-like or hash-like token found in raw telemetry.

Common examples that should **not** automatically become IOCs include:

- event IDs such as `rdphoneypot.login`,
- protocol-state labels such as `ms.rdp.established`,
- RDP NTLM challenge/response values,
- JA3 fingerprints when the output category is specifically malware file hashes,
- random strings extracted from binary or encoded payload fields.

Prefer schema-aware extraction from fields that have indicator meaning, such as URL, hostname, domain, email, download path or explicit file-hash fields.

## Interpretation

A source IP is an indicator, not an identity. A domain or payload hash may be more useful for later correlation than the scanning IP itself.

Do not promote every string that merely looks like an IP/domain/hash to an IOC. Keep only values that have meaningful case context and note where they came from.
