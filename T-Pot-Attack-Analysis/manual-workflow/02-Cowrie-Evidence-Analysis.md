# 02 - Cowrie Evidence Analysis

Use this stage when the source targeted SSH or Telnet, normally TCP/22 or TCP/23.

## 1. Export matching Cowrie events

```bash
COWRIE_LOGDIR="$HOME/tpotce/data/cowrie/log"

sudo zcat -f "$COWRIE_LOGDIR"/cowrie.json* 2>/dev/null |
jq -c --arg ip "$IP" --arg since "$SINCE" '
  select(.src_ip == $ip and (.timestamp >= $since))
' > "$CASE/evidence/cowrie.jsonl"

gzip -1 "$CASE/evidence/cowrie.jsonl"
```

## 2. Count event types

```bash
zcat "$CASE/evidence/cowrie.jsonl.gz" |
jq -r '.eventid // "unknown"' |
sort | uniq -c | sort -nr
```

## 3. Login attempts

```bash
zcat "$CASE/evidence/cowrie.jsonl.gz" |
jq -r '
  select(.eventid == "cowrie.login.failed" or .eventid == "cowrie.login.success") |
  [.timestamp, .eventid, (.username // ""), (.password // ""), (.session // "")] | @tsv
'
```

A `cowrie.login.success` event means the honeypot accepted the configured emulated credential. It does not mean a real production account was compromised.

## 4. Commands

```bash
zcat "$CASE/evidence/cowrie.jsonl.gz" |
jq -r '
  select(.eventid == "cowrie.command.input") |
  [.timestamp, (.session // ""), (.input // "")] | @tsv
'
```

## 5. Downloads and URLs

```bash
zcat "$CASE/evidence/cowrie.jsonl.gz" |
jq -r '
  select(.eventid | test("download|file"; "i")) |
  [.timestamp, (.session // ""), (.url // ""), (.outfile // ""), (.shasum // "")] | @tsv
'
```

## What to record

- failed and accepted credentials,
- session IDs,
- commands and command order,
- download URLs,
- referenced files and hashes,
- first/last activity.

If Cowrie contains no matching rows, record that as **no Cowrie activity observed in the selected window** and continue. Do not infer that the source never used SSH/Telnet outside the retained evidence.

For deeper analysis, see `../01-Cowrie-Attack-Investigation.md`.
