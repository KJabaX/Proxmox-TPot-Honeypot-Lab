# Cowrie Attack Investigation

This document contains commands and notes for investigating SSH/Telnet attacks captured by the Cowrie honeypot in T-Pot.

Cowrie can provide information about:

* Connections
* Login attempts
* Credentials
* Sessions
* Successful logins
* Commands
* File downloads

---

## 1. Define the source IP

```bash
IP="<source-ip>"
```

Check the variable:

```bash
echo "$IP"
```

### What does this do?

Stores the source IP address in a shell variable.

Instead of writing the IP manually into every command, `$IP` can be reused throughout the investigation.

---

## 2. Define Cowrie directories

```bash
LOGDIR="$HOME/tpotce/data/cowrie/log"
DOWNLOADDIR="$HOME/tpotce/data/cowrie/downloads"
```

### What does this do?

`LOGDIR` points to Cowrie log files.

`DOWNLOADDIR` points to files captured by Cowrie.

---

## 3. Create a case directory

```bash
CASE="$HOME/analysis/$IP-$(date -u +%Y%m%dT%H%M%SZ)"

mkdir -p "$CASE"
```

Check:

```bash
echo "$CASE"
```

Example:

```text
/home/<user>/analysis/<source-ip>-<timestamp>
```

### Why?

Each investigation gets its own directory.

Investigation results such as timelines, commands and network events can be stored there.

---

## 4. Export all Cowrie events for the source IP

```bash
sudo zcat -f "$LOGDIR"/cowrie.json* 2>/dev/null |
jq -c --arg ip "$IP" '
select(.src_ip == $ip)
' > "$CASE/events.jsonl"
```

### What does this do?

Reads current and compressed Cowrie JSON logs and keeps only events where:

```text
src_ip = source IP
```

The results are saved to:

```text
events.jsonl
```

This becomes the main Cowrie dataset for the investigation.

---

## 5. Count events

```bash
wc -l "$CASE/events.jsonl"
```

Example:

```text
2837 events.jsonl
```

Each line represents one Cowrie JSON event.

---

## 6. Calculate an evidence hash

```bash
sha256sum "$CASE/events.jsonl" |
tee "$CASE/events.sha256"
```

### Why?

Creates a SHA-256 hash that can later be used to verify that the investigation dataset has not changed.

---

## 7. Find first seen and last seen

```bash
jq -r '.timestamp // empty' "$CASE/events.jsonl" |
sort |
sed -n '1p;$p'
```

Example:

```text
2026-09-22T09:57:04Z
2026-09-22T11:43:51Z
```

These timestamps define the observed attack window.

---

## 8. Count event types

```bash
jq -r '.eventid // "unknown"' "$CASE/events.jsonl" |
sort |
uniq -c |
sort -nr
```

Example:

```text
709 cowrie.session.connect
709 cowrie.session.closed
709 cowrie.client.version
708 cowrie.login.failed
1   cowrie.login.success
1   cowrie.command.input
```

### Important Cowrie events

```text
cowrie.session.connect
```

New connection.

```text
cowrie.login.failed
```

Failed login attempt.

```text
cowrie.login.success
```

Successful authentication.

```text
cowrie.command.input
```

Command entered by the attacker.

```text
cowrie.session.file_download
```

File downloaded during the session.

```text
cowrie.session.closed
```

Session ended.

---

## 9. List unique sessions

```bash
jq -r '.session // empty' "$CASE/events.jsonl" |
sort -u
```

Count sessions:

```bash
jq -r '.session // empty' "$CASE/events.jsonl" |
sort -u |
wc -l
```

### Why?

One source IP can create hundreds of separate connections.

The session ID groups events belonging to the same connection.

---

## 10. Create the full Cowrie timeline

```bash
jq -r '
[
    .timestamp,
    (.session // ""),
    (.eventid // ""),
    (.username // ""),
    (.password // ""),
    (.input // ""),
    (.url // ""),
    (.message // "")
]
| @tsv
' "$CASE/events.jsonl" |
sort |
tee "$CASE/timeline.tsv"
```

The timeline contains:

```text
timestamp
session
event
username
password
command
URL
message
```

---

## 11. Read the timeline

```bash
column -ts $'\t' "$CASE/timeline.tsv" |
less -S
```

Exit with:

```text
q
```

---

## 12. Investigate one specific session

Set the session:

```bash
SESSION="c9eb5ae140d6"
```

Investigate it:

```bash
jq -r --arg session "$SESSION" '
select(.session == $session) |
[
    .timestamp,
    .eventid,
    (.username // ""),
    (.password // ""),
    (.input // ""),
    (.url // ""),
    (.message // "")
] |
map(
    if . == null then ""
    elif type == "array" then join(",")
    elif type == "object" then tojson
    else tostring
    end
) |
@tsv
' "$CASE/events.jsonl" |
sort |
column -ts $'\t'
```

### When should this be used?

Especially after finding:

```text
cowrie.login.success
```

Take the session ID from the successful login and inspect the entire session.

---

## 13. Show login attempts

```bash
jq -r '
select(
    .eventid == "cowrie.login.failed"
    or
    .eventid == "cowrie.login.success"
) |
[
    .timestamp,
    .session,
    .eventid,
    (.username // ""),
    (.password // "")
]
| @tsv
' "$CASE/events.jsonl" |
sort
```

Shows:

* Timestamp
* Session
* Success/failure
* Username
* Password

Useful for identifying brute-force and dictionary attacks.

---

## 14. Count username/password combinations

```bash
jq -r '
select(
    .eventid == "cowrie.login.failed"
    or
    .eventid == "cowrie.login.success"
) |
[
    (.username // ""),
    (.password // "")
]
| @tsv
' "$CASE/events.jsonl" |
sort |
uniq -c |
sort -nr
```

Example:

```text
60 root    root
42 root    admin
15 admin   admin
```

---

## 15. Find successful logins

```bash
jq -r '
select(.eventid == "cowrie.login.success") |
[
    .timestamp,
    .session,
    (.username // ""),
    (.password // "")
]
| @tsv
' "$CASE/events.jsonl"
```

### Next step

If a successful login exists:

```text
session → commands → downloads
```

---

## 16. Show attacker commands

```bash
jq -r '
select(.eventid == "cowrie.command.input") |
[
    .timestamp,
    .session,
    (.input // "")
]
| @tsv
' "$CASE/events.jsonl" |
sort |
tee "$CASE/commands.tsv"
```

The results are saved to:

```text
commands.tsv
```

Typical commands may include:

```text
uname -a
id
whoami
cat /etc/os-release
wget ...
chmod +x ...
```

---

## 17. Count common commands

```bash
jq -r '
select(.eventid == "cowrie.command.input") |
.input // empty
' "$CASE/events.jsonl" |
sort |
uniq -c |
sort -nr
```

Useful when a bot repeats commands across many sessions.

---

## 18. Search for download commands

```bash
jq -r '
select(.eventid == "cowrie.command.input") |
.input // empty
' "$CASE/events.jsonl" |
grep -Ei 'wget|curl|tftp|ftp|scp|http://|https://'
```

Look especially for:

```text
wget
curl
tftp
ftp
scp
```

These are often used to download payloads.

---

## 19. Extract URLs

```bash
jq -r '
select(.eventid == "cowrie.command.input") |
.input // empty
' "$CASE/events.jsonl" |
grep -Eo 'https?://[^[:space:]";]+' |
sort -u
```

URLs may reveal:

* Payload servers
* Malware hosting
* Additional infrastructure
* Command-and-control infrastructure

Record interesting URLs as IOCs.

---

## 20. Find Cowrie file downloads

```bash
jq -r '
select(.eventid == "cowrie.session.file_download") |
[
    .timestamp,
    .session,
    (.url // ""),
    (.outfile // .destfile // ""),
    (.shasum // ""),
    (.message // "")
]
| @tsv
' "$CASE/events.jsonl" |
tee "$CASE/downloads.tsv"
```

Results are stored in:

```text
downloads.tsv
```

If Cowrie captured a file, continue with:

```text
03-Malware-Static-Analysis.md
```

---

## Cowrie Investigation Order

```text
1. Define source IP
2. Export attacker events
3. Calculate hash
4. Determine first/last seen
5. Count event types
6. Identify sessions
7. Create timeline
8. Analyze login attempts
9. Find successful login
10. Investigate successful session
11. Analyze commands
12. Extract URLs
13. Identify file downloads
14. Continue with Suricata / malware analysis
```

---
