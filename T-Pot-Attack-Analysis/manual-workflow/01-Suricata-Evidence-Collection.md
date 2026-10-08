# 01 - Suricata Evidence Collection

Suricata gives the network-level view of the source IP: flows, alerts, ports, protocols and timing.

## 1. Export matching events

```bash
SURICATA_LOGDIR="$HOME/tpotce/data/suricata/log"

sudo zcat -f "$SURICATA_LOGDIR"/eve.json* 2>/dev/null |
jq -c --arg ip "$IP" --arg since "$SINCE" '
  select((.src_ip == $ip or .dest_ip == $ip) and (.timestamp >= $since))
' > "$CASE/evidence/suricata.jsonl"
```

Compress after collection:

```bash
gzip -1 "$CASE/evidence/suricata.jsonl"
```

The important point is to scan the large Suricata source once and analyze the smaller case export afterwards.

## 2. Count events

```bash
zcat "$CASE/evidence/suricata.jsonl.gz" | wc -l
```

## 3. First and last seen

```bash
zcat "$CASE/evidence/suricata.jsonl.gz" |
jq -r '.timestamp // empty' |
sort | sed -n '1p;$p'
```

## 4. Targeted ports and protocols

```bash
zcat "$CASE/evidence/suricata.jsonl.gz" |
jq -r --arg ip "$IP" '
  select(.src_ip == $ip) |
  [(.dest_port // ""), (.proto // ""), (.app_proto // "")] | @tsv
' | sort | uniq -c | sort -nr
```

This identifies whether the activity is single-service or multi-port scanning.

## 5. Event-type distribution

```bash
zcat "$CASE/evidence/suricata.jsonl.gz" |
jq -r '.event_type // "unknown"' |
sort | uniq -c | sort -nr
```

## 6. Alert signatures

```bash
zcat "$CASE/evidence/suricata.jsonl.gz" |
jq -r '
  select(.event_type == "alert") |
  [(.alert.signature // ""), (.alert.category // ""), (.alert.severity // "")] | @tsv
' | sort | uniq -c | sort -nr
```

## Interpretation notes

- A Suricata alert is supporting network evidence, not proof of attribution.
- `app_proto: failed` means application-protocol detection failed; it does **not** mean a login failed.
- Use honeypot application logs to determine credentials, commands or protocol-specific actions.

Use the observed destination ports to decide which service-specific stages matter next.

For deeper Suricata analysis, see `../02-Suricata-Network-Investigation.md`.
