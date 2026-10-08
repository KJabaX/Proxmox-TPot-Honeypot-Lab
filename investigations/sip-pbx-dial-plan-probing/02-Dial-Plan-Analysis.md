# 2. Dial-Plan Analysis

## Observed Pattern

SentryPeer recorded repeated attempts to reach the same underlying destination using multiple dialing formats.

The public version redacts the exact destination number. Representative variants are shown only as patterns:

```text
+44...
0044...
01144...
44...
90044...
00044...
```

The six dominant variants produced the following final counts:

| Dialing pattern | Events |
|---|---:|
| `+44...` | 1,145 |
| `01144...` | 1,139 |
| `0044...` | 1,139 |
| `00044...` | 1,139 |
| `44...` | 1,138 |
| `90044...` | 1,138 |

These six variants account for **6,838 of 7,023 events (97.37%)**.

The final dataset contained **25 unique called-number variants**.

## Why the Pattern Is Significant

The important observation is not the destination itself.

The evidence is the combination of:

1. the same underlying destination being repeated,
2. systematic prefix variation,
3. nearly identical high-volume counts across the main variants,
4. smaller fixed-size batches of additional variants,
5. sustained automated timing.

This behavior is strongly consistent with a tool iterating through possible PBX dial rules.

## Example ES|QL Pattern

```text
FROM logstash-*
| WHERE app_name == "sentrypeer"
| STATS events = COUNT(*) BY called_number
| SORT events DESC
```

The private case used additional source scoping. Exact numbers are intentionally omitted from this public version.

## Assessment

The dial-plan distribution is strong evidence of:

> automated SIP/PBX dial-plan probing consistent with toll-fraud reconnaissance.

Because the target was a honeypot, the case documents attempted behavior rather than successful real-world call routing.
