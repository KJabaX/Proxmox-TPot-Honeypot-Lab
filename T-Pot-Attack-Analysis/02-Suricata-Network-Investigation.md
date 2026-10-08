# Suricata Network Investigation

Suricata provides a network-wide view across the exposed T-Pot services.

Unlike service-specific honeypot logs, Suricata is useful even when the source never touches Cowrie.

---

## 1. Define the Suricata log directory

```bash
SURICATA_LOGDIR="$HOME/tpotce/data/suricata/log"
```

If the location is unknown:

```bash
find "$HOME/tpotce/data" -type f -name 'eve.json*' 2>/dev/null
```

---

## 2. Export events related to the source IP

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

---

## 3. Count and hash the evidence

```bash
wc -l "$CASE/suricata-events.jsonl"

sha256sum "$CASE/suricata-events.jsonl" |
tee "$CASE/suricata-events.sha256"
```

---

## 4. Count event types

```bash
jq -r '.event_type // "unknown"' "$CASE/suricata-events.jsonl" |
sort |
uniq -c |
sort -nr
```

Common event types include:

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

## 5. Identify targeted ports and protocols

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

This is one of the most important steps in a multi-service honeypot environment.

It shows whether the source focused on one service or moved across several exposed services.

---

## 6. Show source/destination service combinations

```bash
jq -r '
[
    (.src_ip // ""),
    (.src_port // ""),
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

## 7. Review IDS alerts

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

## 8. Count alert signatures

```bash
jq -r '
select(.event_type == "alert") |
.alert.signature // empty
' "$CASE/suricata-events.jsonl" |
sort |
uniq -c |
sort -nr
```

---

## 9. Review flow events

```bash
jq -r '
select(.event_type == "flow") |
[
    .timestamp,
    .src_ip,
    (.src_port // ""),
    .dest_ip,
    (.dest_port // ""),
    (.proto // ""),
    (.flow.pkts_toserver // ""),
    (.flow.pkts_toclient // ""),
    (.flow.bytes_toserver // ""),
    (.flow.bytes_toclient // "")
]
| @tsv
' "$CASE/suricata-events.jsonl" |
sort
```

Flow records help distinguish short scans from longer interactions.

---

## 10. DNS events

```bash
jq -r '
select(.event_type == "dns") |
[
    .timestamp,
    .src_ip,
    .dest_ip,
    (.dns.rrname // ""),
    (.dns.rrtype // "")
]
| @tsv
' "$CASE/suricata-events.jsonl"
```

---

## 11. HTTP activity

```bash
jq -r '
select(.event_type == "http") |
[
    .timestamp,
    .src_ip,
    (.src_port // ""),
    .dest_ip,
    (.dest_port // ""),
    (.http.hostname // ""),
    (.http.url // ""),
    (.http.http_method // ""),
    (.http.http_user_agent // "")
]
| @tsv
' "$CASE/suricata-events.jsonl"
```

Useful for web scans, exploit requests and payload retrieval.

---

## 12. TLS activity

```bash
jq -r '
select(.event_type == "tls") |
[
    .timestamp,
    .src_ip,
    .dest_ip,
    (.dest_port // ""),
    (.tls.sni // ""),
    (.tls.version // ""),
    (.tls.subject // ""),
    (.tls.issuerdn // "")
]
| @tsv
' "$CASE/suricata-events.jsonl"
```

Encrypted payloads are not visible, but TLS metadata can still provide useful context.

---

## 13. SSH activity

```bash
jq -r '
select(.event_type == "ssh") |
[
    .timestamp,
    .src_ip,
    .dest_ip,
    (.dest_port // ""),
    (.ssh.client.proto_version // ""),
    (.ssh.client.software_version // ""),
    (.ssh.server.software_version // "")
]
| @tsv
' "$CASE/suricata-events.jsonl"
```

Compare with Cowrie when TCP/22 or TCP/23 activity exists.

---

## 14. Look for multi-port scanning

```bash
jq -r --arg ip "$IP" '
select(.src_ip == $ip and .dest_port != null) |
.dest_port
' "$CASE/suricata-events.jsonl" |
sort -n |
uniq -c |
sort -nr
```

A source hitting several exposed ports in a short period may be performing broad service discovery or automated scanning.

---

## 15. Correlate with honeypot logs

Use destination ports to determine which service-specific logs should be checked.

Examples:

```text
22 / 23      Cowrie
25 / 587     Mailoney
80           SNARE / Tanner
443          h0neytr4p
5060         SentryPeer
5432         Heralding
6379         Redis honeypot
69 / 135 /
445 / 1433 /
1723 / 1883 /
3306 / 27017 Dionaea
```

Compare timestamps across Suricata and honeypot logs.

---

## Suricata Investigation Order

```text
1. Export source-related events
2. Hash the evidence
3. Count event types
4. Identify destination ports and protocols
5. Review flows
6. Review IDS alerts
7. Review protocol metadata
8. Detect multi-port activity
9. Map ports to honeypots
10. Correlate timestamps
11. Add useful infrastructure to the IOC list
```
