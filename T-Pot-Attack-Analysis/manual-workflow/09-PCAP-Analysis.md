# 09 - PCAP Analysis

PCAP is optional. Use it only when packet capture exists for the relevant time window and source IP.

## Locate captures

```bash
find "$HOME/tpotce/data" -type f \( -name '*.pcap' -o -name '*.pcapng' \) 2>/dev/null
```

## Filter by source IP

With `tshark`:

```bash
tshark -r <capture.pcap> -Y "ip.addr == $IP" -w "$CASE/pcap/$IP-filtered.pcap"
```

or with `tcpdump`:

```bash
tcpdump -r <capture.pcap> -w "$CASE/pcap/$IP-filtered.pcap" "host $IP"
```

## Inspect metadata

```bash
tshark -r "$CASE/pcap/$IP-filtered.pcap" \
  -T fields \
  -e frame.time_epoch \
  -e ip.src -e tcp.srcport \
  -e ip.dst -e tcp.dstport \
  -e _ws.col.Protocol |
head -100
```

## What PCAP can add

- TCP handshake/retransmission details,
- exact packet timing,
- protocol negotiation,
- plaintext application content where available,
- payload delivery evidence,
- verification of log-derived assumptions.

## Limitations

If no relevant PCAP exists, record `not captured / not available` rather than treating packet-level evidence as absent activity.

Encrypted TLS application content normally cannot be reconstructed without suitable decryption material.

For deeper packet analysis, see `../05-PCAP-Network-Analysis.md`.
