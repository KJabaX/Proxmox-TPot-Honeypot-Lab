# 06 - System and Elasticsearch Context

This stage records the state of the T-Pot host and optionally checks indexed data without replacing the raw evidence sources.

## Docker snapshot

```bash
sudo docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}' > "$CASE/docker-ps.txt"
```

Optional resource snapshot:

```bash
{
  date -u
  uptime
  free -h
  df -h
  sudo docker stats --no-stream
} >> "$CASE/system-snapshot.txt"
```

This helps explain missing services, container restarts or resource pressure during collection.

## Elasticsearch fallback

If the local Elasticsearch endpoint is accessible without adding credentials to the investigation script, a simple count can confirm whether indexed records exist for the IP.

Example pattern:

```bash
curl -s 'http://127.0.0.1:64298/_count' \
  -H 'Content-Type: application/json' \
  -d "{\"query\":{\"query_string\":{\"query\":\"$IP\"}}}" \
  > "$CASE/elasticsearch-fallback.json"
```

The exact endpoint/index configuration may differ between T-Pot versions. Verify locally before relying on this command.

## Evidence priority

Indexed Elasticsearch results are useful for discovery and fast cross-checking, but the exported honeypot and Suricata logs remain the primary evidence for a case.

Do not store Elasticsearch credentials or other secrets in public documentation or case bundles.
