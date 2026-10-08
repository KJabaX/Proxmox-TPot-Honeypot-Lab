## Cowrie Analysis

### Authentication Activity

The source IP `103.174.102.29` generated **709 SSH sessions** during the observed activity window.

Authentication results:

* **708 failed authentication attempts**
* **1 successful authentication**
* **706 unique username/password combinations**

The activity followed a structured credential-testing pattern. The attack initially targeted the `root` account using a large password dictionary and later switched to the `ubuntu` account.

The password list contained common passwords, keyboard patterns, numerical sequences, product/vendor-related passwords, and variations using special characters.

Examples included patterns based on:

* `root`
* `ubuntu`
* `admin`
* `password`
* numerical sequences
* keyboard patterns such as `qwerty` and `1qaz2wsx`

Most username/password combinations were attempted only once.

Three credential pairs targeting the `ubuntu` account were each observed twice.

The plaintext password values are redacted from the public portfolio snapshot.

The regular timing between authentication attempts, repeated SSH fingerprint, structured password list and separate SSH sessions are consistent with an automated SSH credential dictionary attack.

---

### Successful Authentication

One authentication attempt succeeded:

```text
Timestamp: 2026-09-22T09:59:55.128260Z
Session:   db841bd9b899
Username:  root
Password:  <REDACTED>
```

The SSH client identified itself as:

```text
SSH-2.0-Go
```

The observed HASSH fingerprint was:

```text
98ddc5604ef6a1006a2b49a58759fbe6
```

Shortly after authentication, the following command was submitted:

```bash
echo xsec
```

No additional commands were observed in the successful session.

The command does not perform system modification or discovery. In the context of the very short automated session, it is consistent with a simple command-execution or shell-access check.

---

### File and URL Activity

The Cowrie dataset was searched for common file-transfer commands including:

```text
wget
curl
tftp
ftp
scp
http://
https://
```

No matching commands were observed.

No URLs were extracted from attacker commands.

No `cowrie.session.file_download` events were observed.

Therefore, there is no Cowrie evidence in this dataset of a payload being downloaded during the observed attack.

---

### Cowrie Assessment

The observed activity is consistent with an automated SSH credential dictionary attack.

The source repeatedly established new SSH sessions and attempted username/password combinations from a structured credential list.

One credential combination, `root / <REDACTED>`, resulted in successful authentication. The session immediately executed `echo xsec` and disconnected.

Despite obtaining successful authentication, the source continued credential testing afterward.

No persistence commands, system discovery commands, download commands, URLs or captured payloads were observed in the available Cowrie dataset.
