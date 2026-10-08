# RDPHoneypot Investigation

This guide covers RDP activity captured by T-Pot's `rdphoneypot` service on TCP/3389.

Use it after Suricata triage shows RDP traffic or the T-Pot attack map identifies `RDPHoneypot` activity.

---

## 1. Locate RDPHoneypot events

Typical log path:

```bash
$HOME/tpotce/data/rdphoneypot/log/rdphoneypot.json
```

Search a source IP:

```bash
IP="<source-ip>"

sudo zcat -f "$HOME/tpotce/data/rdphoneypot/log/rdphoneypot.json"* 2>/dev/null |
jq -c --arg ip "$IP" 'select(.src_ip == $ip)'
```

Important event IDs are:

```text
rdphoneypot.session.connect
rdphoneypot.login
rdphoneypot.session.closed
```

---

## 2. Count sessions and login events

```bash
sudo zcat -f "$HOME/tpotce/data/rdphoneypot/log/rdphoneypot.json"* 2>/dev/null |
jq -r --arg ip "$IP" 'select(.src_ip == $ip) | .eventid' |
sort | uniq -c | sort -nr
```

Record:

- first and last seen,
- connection count,
- login-event count,
- unique sessions,
- unique source ports,
- session-close count.

Large numbers of very short sessions are often consistent with automated scanning or credential activity, but volume alone does not prove successful authentication.

---

## 3. Review NLA / NTLM authentication telemetry

RDPHoneypot may record fields such as:

```text
auth_method
derived_domain
domain
username
hostname
nt_challenge
nt_proof
nt_response
hashcat_line
```

For example, count usernames and authentication methods:

```bash
sudo zcat -f "$HOME/tpotce/data/rdphoneypot/log/rdphoneypot.json"* 2>/dev/null |
jq -r --arg ip "$IP" '
  select(.src_ip == $ip and .eventid == "rdphoneypot.login") |
  [(.auth_method // ""), (.derived_domain // .domain // ""), (.username // "")] |
  @tsv
' |
sort | uniq -c | sort -nr
```

### Important interpretation rule

An empty `password` field in an NLA event does **not** mean the attacker tried a blank plaintext password.

RDP NLA uses NTLM challenge-response authentication. Fields such as `nt_challenge`, `nt_proof`, `nt_response` and `hashcat_line` are authentication telemetry, not plaintext passwords.

Do not report them as malware file hashes either.

A safe report wording is:

```text
Authentication: NLA / NTLM challenge-response observed
Plaintext password: not captured
```

---

## 4. Analyze session durations

```bash
sudo zcat -f "$HOME/tpotce/data/rdphoneypot/log/rdphoneypot.json"* 2>/dev/null |
jq -r --arg ip "$IP" '
  select(.src_ip == $ip and .eventid == "rdphoneypot.session.closed") |
  (.duration // "unknown")
' |
sort -n | uniq -c | sort -nr
```

A high proportion of 0-1 second sessions can support an automated-scanning assessment when combined with high connection/login volume.

---

## 5. Correlate with Suricata

Search Suricata for the same source IP and TCP/3389.

Useful RDP-related events and signatures may include:

```text
RDP connection request
MS Remote Desktop Request RDP
Remote Desktop Administrator Login Request
Unusually fast Terminal Server Traffic
RDP Authentication Bypass Attempt
```

Treat Suricata signatures as detections, not proof that the described action succeeded. For example, an authentication-bypass signature match does not by itself prove that authentication was bypassed.

Correlate timestamps between Suricata and RDPHoneypot before drawing a conclusion.

---

## 6. Collector v3.4

The `tpot-ip-report` v3.4 change adds a dedicated `RDPHONEYPOT ANALYSIS` section with:

- RDP event counts,
- first/last seen,
- unique sessions and source ports,
- authentication methods,
- domains and usernames,
- session durations,
- NTLM/NLA capture counts,
- RDP-specific behavior flags.

It also labels the password field as `<NOT_CAPTURED>` for RDP NLA events instead of incorrectly treating it as an empty plaintext password.

This public documentation snapshot records the v3.4 behavior changes; the collector source diff is not included here.

---

## 7. Final assessment

A defensible RDP conclusion should separate observation from inference.

Example:

```text
The source generated repeated TCP/3389 connections and RDP NLA login events.
RDPHoneypot recorded repeated Administrator targeting and NTLM challenge-response telemetry.
Most sessions were short-lived, and Suricata independently recorded RDP scan/login-related signatures.
The pattern is consistent with automated RDP scanning or credential activity.
No successful Windows login was demonstrated by the honeypot evidence.
```

Do not infer attacker identity from the source IP alone.
