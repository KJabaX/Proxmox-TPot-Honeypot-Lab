# 12 - Evidence Integrity and Bundle

This stage finalizes the case evidence for transfer, review and later verification.

## 1. Create SHA-256 manifest

From the case directory:

```bash
cd "$CASE"
find . -type f ! -name 'SHA256SUMS' -print0 |
sort -z |
xargs -0 sha256sum > SHA256SUMS
```

Verify later with:

```bash
sha256sum -c SHA256SUMS
```

## 2. Review expected outputs

A complete case may contain:

```text
report.txt
timeline.tsv.gz
iocs.tsv
credentials.tsv
campaign-context.tsv
campaign-password-similarity.tsv
evidence/
pcap/
artifacts/
pcap-manifest.tsv
artifact-references.tsv
artifact-metadata.tsv
docker-ps.txt
system-snapshot.txt
elasticsearch-fallback.json
SHA256SUMS
```

Missing optional files should be explained in `report.txt` rather than silently omitted when their absence matters.

## 3. Package the case

From the parent directory:

```bash
tar -cf - "$(basename "$CASE")" | gzip -1 > "${CASE}-FULL.tar.gz"
```

The low gzip level reduces CPU pressure on the honeypot VM while still producing a convenient transfer archive.

## 4. Transfer safely

Prefer copying the archive from the isolated T-Pot VM to the trusted workstation over the management path, then review/upload it from the workstation.

Do not execute files from `artifacts/` after transfer.

## Evidence principle

Hashing proves that the bundled file has not changed since the manifest was generated. It does not prove that the original telemetry was complete or that no logs were lost before collection.
