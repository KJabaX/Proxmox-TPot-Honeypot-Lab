# 11 - Campaign Correlation

Campaign correlation is optional and should be separated from the basic single-IP conclusion.

Use it when multiple sources show similar high-volume behavior, timing, credentials, ports or infrastructure.

## Useful comparisons

Compare peer sources by:

- targeted ports/services,
- first/last seen times,
- per-minute/per-second activity,
- usernames,
- exact password-set overlap,
- credential ordering,
- source-port behavior,
- p0f/TCP fingerprints,
- payload/domain/hash reuse,
- ASN/hosting context.

## Efficient collection

Avoid rescanning a large compressed Dionaea or Suricata history once per peer IP. Prefer:

1. an indexed Elasticsearch query, or
2. one bounded streaming pass that collects all peer sources together.

Example IP set:

```bash
PEERS='["<ip1>","<ip2>","<ip3>"]'
```

Then filter once with `jq` using membership in the peer list.

## Outputs

Useful derived files include:

```text
campaign-context.tsv
campaign-password-similarity.tsv
```

## Interpretation limits

Shared ASN, hosting provider, credential corpus or synchronized timing can support **campaign similarity** or **shared infrastructure context**.

They do not, by themselves, prove:

- common ownership,
- a single threat actor,
- a botnet relationship,
- control of the source system by the apparent operator.

A single-IP case can be complete without this stage. Campaign correlation should normally become a separate investigation when it grows beyond the source being documented.
