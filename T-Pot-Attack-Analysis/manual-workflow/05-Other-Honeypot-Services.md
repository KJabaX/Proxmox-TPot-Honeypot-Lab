# 05 - Other Honeypot Services

Not every source belongs to Cowrie or Dionaea. This stage checks the remaining T-Pot service logs after Suricata has shown which ports and protocols matter.

## Identify likely services

Use the Suricata destination ports and the current runtime mapping:

```bash
sudo docker ps --format 'table {{.Names}}\t{{.Ports}}'
```

Typical examples include Mailoney, SentryPeer, SNARE/Tanner, Heralding, Redis honeypots, RDPHoneypot and HTTPS honeypots.

## Locate service logs

```bash
find "$HOME/tpotce/data" -maxdepth 4 -type f 2>/dev/null |
grep -Ei 'mailoney|sentrypeer|snare|tanner|heralding|redis|rdp|honey'
```

## Search for the source IP

For a known service log directory:

```bash
sudo zcat -f <service-log-files> 2>/dev/null |
grep -F "$IP"
```

For JSON logs, prefer structured filtering:

```bash
sudo zcat -f <service-json-files> 2>/dev/null |
jq -c --arg ip "$IP" --arg since "$SINCE" '
  select((.src_ip == $ip or .source_ip == $ip) and ((.timestamp // "") >= $since))
'
```

## RDPHoneypot

For TCP/3389, inspect `rdphoneypot` separately instead of treating it as a generic username/password service.

Typical event IDs:

```text
rdphoneypot.session.connect
rdphoneypot.login
rdphoneypot.session.closed
```

Useful fields include:

```text
auth_method
derived_domain
username
hostname
duration
nt_challenge
nt_proof
nt_response
hashcat_line
```

Important: NLA/NTLM challenge-response values are authentication telemetry. An empty `password` field does not mean a blank plaintext password was attempted, and NTLM response values should not be promoted to malware hash IOCs.

For deeper RDP analysis see [`../07-RDPHoneypot-Investigation.md`](../07-RDPHoneypot-Investigation.md).

## What to extract

Depending on the protocol, record:

- timestamps,
- source/destination ports,
- usernames and authentication metadata,
- SMTP envelope fields,
- SIP methods and identities,
- HTTP paths, methods, user agents and hosts,
- protocol commands,
- payload/file references.

Only call a value a plaintext password when the honeypot actually captured it as plaintext.

Save useful matches under `evidence/` and include them in the final timeline.

Avoid treating generic string matches from unrelated initialization/service logs as attack evidence; confirm the surrounding schema and service first.
