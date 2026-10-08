# T-Pot Attack Triage

This is the recommended starting point for a new source IP observed by the T-Pot environment.

The goal is to determine when the source was active, which services it targeted, which honeypot logs matter, and what type of activity was observed.

Do not assume that every source IP belongs to Cowrie.

---

## 1. Define the source IP

```bash
IP="<source-ip>"
echo "$IP"
```

---

## 2. Create a case directory

```bash
CASE="$HOME/analysis/$IP-$(date -u +%Y%m%dT%H%M%SZ)"
mkdir -p "$CASE"
```

---

## 3. Define the Suricata log directory

```bash
SURICATA_LOGDIR="$HOME/tpotce/data/suricata/log"
```

If needed:

```bash
find "$HOME/tpotce/data" -type f -name 'eve.json*' 2>/dev/null
```

---

## 4. Export network evidence

```bash
sudo zcat -f "$SURICATA_LOGDIR"/eve.json* 2>/dev/null |
jq -c --arg ip "$IP" '
select(
    .src_ip == $ip
    or
    .dest_ip == $ip
)
' > "$CASE/suricata-events.jsonl"
```

Count and hash the export:

```bash
wc -l "$CASE/suricata-events.jsonl"

sha256sum "$CASE/suricata-events.jsonl" |
tee "$CASE/suricata-events.sha256"
```

---

## 5. First seen and last seen

```bash
jq -r '.timestamp // empty' "$CASE/suricata-events.jsonl" |
sort |
sed -n '1p;$p'
```

---

## 6. Identify targeted ports

```bash
jq -r --arg ip "$IP" '
select(.src_ip == $ip) |
[
    (.dest_port // ""),
    (.proto // "")
]
| @tsv
' "$CASE/suricata-events.jsonl" |
sort |
uniq -c |
sort -nr
```

This quickly reveals single-service attacks versus multi-port scanning.

---

## 7. Show source-to-service combinations

```bash
jq -r '
[
    (.src_ip // ""),
    (.dest_ip // ""),
    (.dest_port // ""),
    (.proto // "")
]
| @tsv
' "$CASE/suricata-events.jsonl" |
sort |
uniq -c |
sort -nr
```

---

## 8. Count event types

```bash
jq -r '.event_type // "unknown"' "$CASE/suricata-events.jsonl" |
sort |
uniq -c |
sort -nr
```

Typical values include:

```text
alert
flow
dns
http
tls
ssh
anomaly
```

---

## 9. Review alerts

```bash
jq -r '
select(.event_type == "alert") |
[
    .timestamp,
    .src_ip,
    (.src_port // ""),
    .dest_ip,
    (.dest_port // ""),
    (.proto // ""),
    (.alert.signature // ""),
    (.alert.category // ""),
    (.alert.severity // "")
]
| @tsv
' "$CASE/suricata-events.jsonl" |
sort
```

---

## 10. Map ports to honeypots

| Port | Protocol | Continue with |
|---:|---|---|
| 22 | TCP | Cowrie |
| 23 | TCP | Cowrie |
| 25 | TCP | Mailoney |
| 69 | UDP | Dionaea |
| 80 | TCP | SNARE / Tanner |
| 135 | TCP | Dionaea |
| 443 | TCP | h0neytr4p |
| 445 | TCP | Dionaea |
| 587 | TCP | Mailoney |
| 1433 | TCP | Dionaea |
| 1723 | TCP | Dionaea |
| 1883 | TCP | Dionaea |
| 3306 | TCP | Dionaea |
| 3389 | TCP | RDP honeypot |
| 5060 | TCP/UDP | SentryPeer |
| 5432 | TCP | Heralding |
| 6379 | TCP | Redis honeypot |
| 27017 | TCP | Dionaea |

Verify the current runtime mapping:

```bash
sudo docker ps --format 'table {{.Names}}\t{{.Ports}}'
```

---

## 11. Choose the investigation path

### SSH / Telnet

Use:

```text
01-Cowrie-Attack-Investigation.md
```

Investigate logins, credentials, sessions, commands and downloads.

### HTTP / HTTPS

Review Suricata HTTP/TLS data plus SNARE/Tanner or h0neytr4p logs.

Look for paths, methods, Host headers, User-Agent values, scanner fingerprints and exploit requests.

### SMTP

Review Mailoney events.

Look for:

```text
EHLO / HELO
MAIL FROM
RCPT TO
relay attempts
recipient enumeration
```

### SIP

Review SentryPeer events.

Look for:

```text
REGISTER
INVITE
OPTIONS
extension probing
authentication attempts
```

### Dionaea-backed services

For SMB, MQTT, databases, TFTP and related services, inspect Dionaea logs and correlate them with Suricata.

Look for protocol requests, payload transfer, exploit attempts and captured files.

### RDP / Redis / Heralding

Inspect the relevant honeypot logs and correlate their timestamps with Suricata.

---

## 12. Working classification

A useful initial classification is:

```text
Single-port scan
Multi-port scan
Brute force
Credential attack
Protocol enumeration
Exploit attempt
Payload delivery
Malware upload/download
SMTP abuse
SIP abuse
Database probing
Unknown / requires deeper analysis
```

This is a working description of observed behavior, not attribution.

---

## Triage Output

Record:

```text
Source IP:
First seen:
Last seen:

Targeted ports:
Protocols:
Relevant honeypots:

Suricata alerts:
Observed behavior:

Next investigation step:
Notes:
```
