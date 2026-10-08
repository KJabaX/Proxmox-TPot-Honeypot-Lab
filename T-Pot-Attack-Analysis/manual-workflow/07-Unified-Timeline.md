# 07 - Unified Timeline

A unified timeline combines multiple evidence sources into one chronological view. This is one of the most important stages because it prevents conclusions from being based on a single log source.

## Recommended columns

```text
timestamp
source
service
event_type
src_ip
src_port
dst_ip
dst_port
protocol
username
password
detail
```

## Build source-specific TSV rows

Normalize each source separately. Example Suricata pattern:

```bash
zcat "$CASE/evidence/suricata.jsonl.gz" |
jq -r '[
  (.timestamp // ""),
  "suricata",
  (.app_proto // ""),
  (.event_type // ""),
  (.src_ip // ""),
  (.src_port // ""),
  (.dest_ip // ""),
  (.dest_port // ""),
  (.proto // ""),
  "",
  "",
  (.alert.signature // "")
] | @tsv'
```

Create equivalent rows for Cowrie, Dionaea and relevant other honeypots, then concatenate and sort:

```bash
cat "$CASE"/timeline-*.tsv |
sort -t $'\t' -k1,1 > "$CASE/timeline.tsv"

gzip -1 "$CASE/timeline.tsv"
```

## Questions the timeline should answer

- When did the source first appear?
- When did it stop?
- Was there reconnaissance before credential guessing?
- Did the source move between services?
- Do IDS alerts align with honeypot application events?
- Were downloads or commands preceded by successful emulated logins?

## Important distinction

Different products may record slightly different timestamps for the same network interaction. Treat close timestamps as correlated observations when the flow/session context supports it; do not assume every timestamp represents a separate attack action.
