# PCAP Network Analysis

This document contains commands for capturing and analyzing network traffic associated with an investigation.

Tools used:

```text
ss
tcpdump
tshark
```

PCAP analysis is useful across all exposed services and should include both TCP and UDP.

---

## 1. Check current connections

```bash
sudo ss -tunap
```

Search for the source IP:

```bash
sudo ss -tunap |
grep -F "$IP"
```

---

## 2. Capture traffic for the source

```bash
sudo tcpdump -i any -nn host "$IP" -w "$CASE/$IP.pcap"
```

Stop with:

```text
Ctrl+C
```

---

## 3. Create a PCAP hash

```bash
sha256sum "$CASE/$IP.pcap" |
tee "$CASE/$IP.pcap.sha256"
```

---

## 4. Read the capture

```bash
tshark -r "$CASE/$IP.pcap"
```

Filter to the source:

```bash
tshark -r "$CASE/$IP.pcap" -Y "ip.addr == $IP"
```

---

## 5. Show ports and protocols

```bash
tshark -r "$CASE/$IP.pcap" -Y "ip.addr == $IP" -T fields -e frame.time -e ip.src -e ip.dst -e tcp.srcport -e tcp.dstport -e udp.srcport -e udp.dstport -e _ws.col.Protocol
```

This is useful for identifying which exposed services were contacted.

---

## 6. TCP conversations

```bash
tshark -r "$CASE/$IP.pcap" -q -z conv,tcp
```

---

## 7. UDP conversations

```bash
tshark -r "$CASE/$IP.pcap" -q -z conv,udp
```

UDP analysis is important for services such as SIP and TFTP.

---

## 8. DNS traffic

```bash
tshark -r "$CASE/$IP.pcap" -Y "dns"
```

---

## 9. HTTP traffic

```bash
tshark -r "$CASE/$IP.pcap" -Y "http"
```

Look for:

```text
HTTP methods
request paths
Host headers
User-Agent
payload downloads
scanner fingerprints
```

---

## 10. TLS traffic

```bash
tshark -r "$CASE/$IP.pcap" -Y "tls"
```

Encrypted application payloads are not normally visible, but metadata may still be useful.

---

## 11. SSH traffic

```bash
tshark -r "$CASE/$IP.pcap" -Y "ssh"
```

Compare SSH timestamps with Cowrie.

---

## 12. SIP traffic

```bash
tshark -r "$CASE/$IP.pcap" -Y "sip"
```

Useful for SentryPeer cases involving REGISTER, INVITE or OPTIONS activity.

---

## 13. SMB traffic

```bash
tshark -r "$CASE/$IP.pcap" -Y "smb || smb2"
```

Useful when investigating traffic targeting TCP/445.

---

## 14. Filter a specific destination port

Example:

```bash
PORT=445

tshark -r "$CASE/$IP.pcap" -Y "ip.addr == $IP && (tcp.port == $PORT || udp.port == $PORT)"
```

This can be reused for any exposed port.

---

## 15. Record network indicators

Record useful evidence such as:

```text
Source IP
Destination IP
Destination port
Protocol
Domain
URL
User-Agent
Additional contacted host
Suricata signature
```

---

## PCAP Investigation Order

```text
1. Check active connections
2. Capture traffic if useful
3. Hash the PCAP
4. Identify ports and protocols
5. Review TCP conversations
6. Review UDP conversations
7. Inspect relevant application protocols
8. Correlate with Suricata
9. Correlate with honeypot logs
10. Extract additional IOCs
```
