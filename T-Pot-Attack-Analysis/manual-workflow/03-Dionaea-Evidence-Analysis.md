# 03 - Dionaea Evidence Analysis

Use this stage when the source targeted services emulated by Dionaea, such as MSSQL, SMB, MySQL, MQTT, MongoDB, RPC or TFTP.

## 1. Export matching Dionaea events

```bash
DIONAEA_LOGDIR="$HOME/tpotce/data/dionaea/log"

sudo zcat -f "$DIONAEA_LOGDIR"/dionaea.json* 2>/dev/null |
jq -c --arg ip "$IP" --arg since "$SINCE" '
  select(.src_ip == $ip and (.timestamp >= $since))
' > "$CASE/evidence/dionaea.jsonl"

gzip -1 "$CASE/evidence/dionaea.jsonl"
```

## 2. Identify protocols and destination ports

```bash
zcat "$CASE/evidence/dionaea.jsonl.gz" |
jq -r '
  [
    (.connection.protocol // .protocol // ""),
    (.dst_port // .connection.local_port // ""),
    (.connection.transport // "")
  ] | @tsv
' | sort | uniq -c | sort -nr
```

## 3. Extract credentials

Dionaea schema varies by protocol/version, so inspect a small sample first:

```bash
zcat "$CASE/evidence/dionaea.jsonl.gz" | head -5 | jq .
```

For MSSQL-like rows containing credential arrays:

```bash
zcat "$CASE/evidence/dionaea.jsonl.gz" |
jq -r '
  [
    (.timestamp // ""),
    (.connection.protocol // ""),
    ((.credentials.username // .username // [""]) | if type == "array" then .[0] else . end),
    ((.credentials.password // .password // [""]) | if type == "array" then .[0] else . end)
  ] | @tsv
'
```

## 4. Credential frequency

```bash
zcat "$CASE/evidence/dionaea.jsonl.gz" |
jq -r '
  ((.credentials.username // .username // [""]) | if type == "array" then .[0] else . end) as $u |
  ((.credentials.password // .password // [""]) | if type == "array" then .[0] else . end) as $p |
  select(($u != "") or ($p != "")) |
  [$u, $p] | @tsv
' | sort | uniq -c | sort -nr
```

## 5. Timing and rate

```bash
zcat "$CASE/evidence/dionaea.jsonl.gz" |
jq -r '.timestamp // empty' |
sort | sed -n '1p;$p'
```

For very high-volume activity, calculate per-minute or per-second counts from the exported case file rather than rereading the full Dionaea history.

## Interpretation notes

- `connection.type: accept` means the honeypot accepted the network connection; it does **not** mean authentication succeeded.
- Credential-bearing events support classification as credential guessing when attempts are repeated at automation-like volume/rate.
- A single source IP does not establish actor identity.

Record the protocol, target port, usernames, password corpus, rate, duration and any payload references.
