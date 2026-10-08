# 00 - Preparation and Case Directory

This stage defines the investigation target, time window and output location before any large log source is scanned.

## 1. Define the target

```bash
IP="<source-ip>"
DAYS=7
SINCE="$(date -u -d "$DAYS days ago" +%Y-%m-%dT%H:%M:%SZ)"
```

`IP` is the source being investigated. `DAYS` keeps the search bounded instead of reading unlimited history.

## 2. Create a case directory

```bash
CASE="$HOME/analysis/$IP-$(date -u +%Y%m%dT%H%M%SZ)"
mkdir -p "$CASE/evidence" "$CASE/pcap" "$CASE/artifacts"
```

Keeping each case separate makes later hashing, transfer and documentation easier.

## 3. Record basic context

```bash
{
  echo "target_ip=$IP"
  echo "days=$DAYS"
  echo "since_utc=$SINCE"
  echo "generated_utc=$(date -u +%Y-%m-%dT%H:%M:%SZ)"
  uname -a
} > "$CASE/system-snapshot.txt"
```

## 4. Verify T-Pot paths

Common data root:

```bash
TPOT_DATA="$HOME/tpotce/data"
find "$TPOT_DATA" -maxdepth 3 -type f \( -name '*.json' -o -name '*.json.*' -o -name 'eve.json*' \) 2>/dev/null | head -50
```

Do not delete or modify T-Pot source logs during an investigation.

## Why this stage matters

A reproducible case should always answer:

- which source was investigated,
- which UTC time window was used,
- where exported evidence was stored,
- when the collection was performed.

Continue with [01 - Suricata Evidence Collection](01-Suricata-Evidence-Collection.md).
