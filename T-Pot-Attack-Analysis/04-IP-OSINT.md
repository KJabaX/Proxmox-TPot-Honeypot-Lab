# IP OSINT

This document contains basic commands for investigating infrastructure associated with an source IP address.

The goal is to identify information about the network and infrastructure behind the IP.

IP OSINT does **not** necessarily identify the actual person behind an attack.

---

## 1. Define the IP

```bash
IP="<source-ip>"
```

Check:

```bash
echo "$IP"
```

---

## 2. WHOIS lookup

```bash
whois "$IP"
```

For easier reading:

```bash
whois "$IP" |
less
```

### What are we looking for?

Important fields can include:

```text
NetName
Organisation
Country
CIDR
Origin ASN
Abuse contact
Network range
```

---

## 3. Important WHOIS limitation

WHOIS normally identifies the network that owns or announces an IP address.

It does not prove that the network owner performed the attack.

The attacking host could be:

```text
compromised server
botnet node
VPN
proxy
cloud server
infected IoT device
```

Therefore avoid statements such as:

```text
"The attacker is from country X."
```

A better statement is:

```text
"The source IP is registered to / announced by network X."
```

---

## 4. Reverse DNS lookup

```bash
dig -x "$IP" +short
```

Alternative:

```bash
host "$IP"
```

### What does this do?

Attempts to find a PTR hostname associated with the IP.

Example:

```text
server123.example.net
```

Not every IP has a reverse DNS record.

---

## 5. Save WHOIS output

```bash
whois "$IP" > "$CASE/whois.txt"
```

### Why?

Preserves the WHOIS information used during the investigation.

WHOIS information can change over time.

---

## 6. Save reverse DNS

```bash
dig -x "$IP" +short |
tee "$CASE/reverse-dns.txt"
```

---

## 7. Record infrastructure information

Record at least:

```text
Source IP:

Network owner:

ASN:

Network range:

Registered country:

Reverse DNS:

Abuse contact:

Notes:
```

---

## 8. Interpret results carefully

Useful conclusions:

```text
The IP belongs to a hosting provider.
The IP belongs to a residential ISP.
The IP is part of an ASN associated with cloud infrastructure.
The IP has no PTR record.
```

Avoid unsupported conclusions such as:

```text
This company attacked the server.
This person is the attacker.
The attacker physically lives in this country.
```

---

## IP OSINT Order

```text
1. WHOIS
2. Identify network owner
3. Identify ASN
4. Identify CIDR/network range
5. Check registered country
6. Check reverse DNS
7. Record results
8. Compare with other IOCs
```
