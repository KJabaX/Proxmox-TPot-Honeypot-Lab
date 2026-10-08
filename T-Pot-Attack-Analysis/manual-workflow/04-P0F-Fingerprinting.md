# 04 - p0f Fingerprinting

p0f provides passive TCP fingerprint context for connections observed by T-Pot.

## Export matching rows

First locate the p0f log files used by the installation:

```bash
find "$HOME/tpotce/data" -maxdepth 4 -type f -iname '*p0f*' 2>/dev/null
```

Then search only the files that contain structured p0f output:

```bash
sudo zcat -f <p0f-log-files> 2>/dev/null |
grep -F "$IP" > "$CASE/evidence/p0f-matches.txt"
```

If the source is JSON, normalize it with `jq` and save as JSONL instead.

## What to inspect

Record, when available:

- OS guess,
- TCP/IP stack signature,
- distance/TTL estimate,
- link type,
- NAT or proxy hints,
- first/last matching observation.

## Interpretation

p0f is **heuristic context**, not definitive attribution. A fingerprint may be affected by NAT, proxies, middleboxes, virtualization or incomplete traffic.

Use p0f mainly for comparing whether repeated sources show similar network behavior. Do not claim a specific host or operator based only on p0f output.
