# 4. Campaign Correlation

## Related High-Volume Sources

The seven-day Dionaea campaign context identified several source IPs with nearly identical high-volume MSSQL behavior.

The most relevant group was:

| Source IP | Events | Credential events | First seen (UTC) | Last seen (UTC) | Protocol | Port | Username |
|---|---:|---:|---|---|---|---:|---|
| `62.210.205.239` | 250,538 | 250,529 | `2026-10-02 18:43:04` | `2026-10-03 21:51:50` | mssqld | 1433 | sa |
| `195.154.176.7` | 248,141 | 248,132 | `2026-10-02 18:47:11` | `2026-10-03 21:52:01` | mssqld | 1433 | sa |
| `62.210.124.129` | 246,284 | 246,277 | `2026-10-02 18:47:16` | `2026-10-03 21:52:02` | mssqld | 1433 | sa |
| `62.210.204.18` | 244,570 | 244,570 | `2026-10-02 18:47:23` | `2026-10-03 21:51:50` | mssqld | 1433 | sa |
| `62.210.205.126` | 243,497 | 243,490 | `2026-10-02 18:47:20` | `2026-10-03 21:52:20` | mssqld | 1433 | sa |
| `195.154.167.21` | 237,522 | 237,519 | `2026-10-02 18:47:14` | `2026-10-03 21:51:54` | mssqld | 1433 | sa |
| `195.154.176.27` | 226,324 | 226,320 | `2026-10-02 18:47:08` | `2026-10-03 21:49:23` | mssqld | 1433 | sa |

Together these seven sources generated **1,696,876 Dionaea events**, of which **1,696,837 contained credentials**.

The six peer sources began within approximately **4 minutes and 19 seconds** of the target source and ended within approximately **2 minutes and 57 seconds** of one another.

Elastic dashboard enrichment associated this group with **ASN 12876 / Scaleway SAS**.

## Behavioral Similarity

The strongest common characteristics are:

- same target service: MSSQL,
- same target port: TCP/1433,
- same username: `sa`,
- nearly identical event volume,
- strongly overlapping activity window,
- similar start and stop times,
- high-rate automated credential testing.

This is strong circumstantial evidence of a **shared automated campaign or common toolset**.

It is not sufficient to attribute all addresses to the same human operator. The addresses may represent rented VPS infrastructure, automation distributed across hosts, compromised systems, or another shared execution environment.

## Password-Similarity Caveat

The v3.3.1 collector generated a bounded password hash sketch to avoid retaining every password from every source in memory.

For the high-volume peers the resulting `password_sketch_similarity` values were `0.0000`. This value should **not** be treated as proof that the password dictionaries were unrelated.

For very large ordered or partitioned wordlists, separate workers can receive different portions of the same corpus and therefore produce little or no direct sample overlap while still belonging to the same campaign.

The current evidence supports campaign correlation primarily through timing, target, username, event volume and infrastructure context rather than password-sketch overlap.

## Other Campaigns in the Same Window

The campaign context also showed unrelated or less-related activity, including:

- other high-volume MSSQL sources on `1433/TCP`,
- SMB traffic targeting `445/TCP`,
- MySQL credential activity targeting `3306/TCP` with username `root`.

These should not automatically be grouped into the same campaign merely because they occurred during the same seven-day period.

## Assessment

The six high-volume peers listed above are **strong candidates for the same distributed MSSQL credential-guessing campaign or common automation framework** as `62.210.205.239`.

Further confirmation would benefit from exact password-sequence comparison, cross-source timing analysis, and infrastructure enrichment for each peer.
