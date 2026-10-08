## 6. Indicators of Compromise (IOCs)

The following indicators were identified during the investigation.

### 6.1 Network Indicators

| Indicator | Type | Source |
|---|---|---|
| `103.174.102.29` | Source IP address | Cowrie / Suricata |
| `103.174.102.0/23` | Network range | WHOIS / OSINT |
| `AS133719` | ASN | WHOIS / OSINT |
| `IDIGITALCAMP WEB SERVICES` | Network provider | WHOIS / OSINT |

The source IP `103.174.102.29` is the primary network indicator associated with the observed SSH activity.

### 6.2 SSH Indicators

| Indicator | Type |
|---|---|
| `SSH-2.0-Go` | SSH client version |
| `98ddc5604ef6a1006a2b49a58759fbe6` | HASSH fingerprint |
| TCP `22` | Target service |

The same SSH client version was observed by both Cowrie and Suricata.

The HASSH fingerprint remained consistent during the observed activity and can be used as an additional indicator when correlating similar SSH sessions.

### 6.3 Authentication Indicators

The only successful authentication observed during the investigation used:

- **Username:** `root`
- **Password:** `<REDACTED>`
- **Session ID:** `db841bd9b899`

The credentials are included as investigation artifacts and should not be treated as globally unique indicators of malicious activity.

### 6.4 Command Indicator

The only command observed after successful authentication was:

```bash
echo xsec
```

### 6.5 External Enrichment Indicators

Shodan InternetDB associated the source IP with the hostname:

`stage-zoho.api.mytyles.digital`

However, a DNS lookup showed that the hostname currently resolves to:

`43.204.226.112`

Because the hostname no longer resolves to the investigated IP, it was treated as historical enrichment information and not as a confirmed IOC for this incident.

### 6.6 Indicators Not Observed

No additional indicators were identified in the available Cowrie and Suricata datasets:

- No downloaded file hashes
- No malware samples
- No payload URLs
- No HTTP requests
- No DNS queries
- No command-and-control domains
- No persistence-related commands

The primary IOC for this investigation is therefore the source IP `103.174.102.29`, supported by the SSH client information and HASSH fingerprint.
