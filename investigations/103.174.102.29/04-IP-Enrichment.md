## 4. IP Enrichment

Passive OSINT sources were used to collect additional information about the source IP `103.174.102.29`. No active scanning or direct connection attempts were performed against the investigated host.

### 4.1 Network and ASN Information

The IP address `103.174.102.29` belongs to the network `103.174.102.0/23` and is associated with:

* **ASN:** AS133719
* **Organization:** IDIGITALCAMP WEB SERVICES
* **Registered country:** India

The registered country represents the location of the network registration and does not necessarily indicate the physical location of the person or system responsible for the observed activity.

### 4.2 Shodan InternetDB

Shodan InternetDB returned the following information for `103.174.102.29`:

* Open ports observed: `22`, `80`, `443`, `4000`, `8000`
* Detected technologies included:

  * Ubuntu Linux
  * OpenSSH 8.9p1
  * Apache HTTP Server 2.4.52
  * Python
  * Uvicorn
* Associated hostname:

  * `stage-zoho.api.mytyles.digital`

Shodan also returned several CVE identifiers associated with the detected software versions. These were treated only as enrichment information. The vulnerabilities were not independently verified against the host and therefore are not considered confirmed vulnerabilities.

### 4.3 Hostname Verification

The hostname reported by Shodan was verified using a DNS A-record lookup:

```text
stage-zoho.api.mytyles.digital -> 43.204.226.112
```
The hostname does not currently resolve to the investigated IP address `103.174.102.29`.
This suggests that the hostname association in Shodan may be historical or outdated. The hostname was therefore not treated as evidence directly linking the domain to the observed honeypot activity.

### 4.4 External Reputation

The source IP `103.174.102.29` was also observed by the SANS Internet Storm Center / DShield sensor network.
At the time of investigation, the IP appeared among active scanners targeting TCP port `2222`, with **3,258 observations**.
TCP port `2222` is commonly used as an alternative SSH port and is also associated with SSH honeypot deployments such as Cowrie.
This independent observation is consistent with the repeated automated SSH activity identified in the local Cowrie and Suricata datasets.
The DShield information was treated as external enrichment and does not by itself identify the operator of the source system.

### 4.5 AbuseIPDB

AbuseIPDB reported a 100% Abuse Confidence Score for 103.174.102.29, with 248 reports from 69 reporters. The IP was associated
with IDIGITALCAMP WEB SERVICES and AS133719. This supports the conclusion that the source IP has a history of reported abusive
activity, but the data is treated as external reputation information rather than direct evidence from the honeypot.

### 4.6 VirusTotal

VirusTotal was used to check the reputation of the source IP 103.174.102.29.

At the time of investigation:

4 of 89 security vendors classified the IP as malicious
Several additional vendors classified the IP as suspicious
The IP was associated with AS133719 / IDIGITALCAMP WEB SERVICES
Network: 103.174.102.0/23
Registered country: India
The most recent VirusTotal analysis shown was approximately two days old

The vendors flagging the IP as malicious included Fortinet, GreyNoise, MalwareURL and SOCRadar.

The VirusTotal result provides additional reputation evidence that the IP has been associated with potentially malicious
activity. However, most vendors did not classify the IP as malicious, so the VirusTotal result was treated as supporting
enrichment rather than definitive evidence.

The primary evidence for this investigation remains the activity recorded directly by the Cowrie and Suricata sensors.

### 4. Enrichment Assessment

The external enrichment sources provide additional context for the source IP 103.174.102.29.

WHOIS and related network information associate the IP with AS133719 and IDIGITALCAMP WEB SERVICES, a data center and web hosting
provider.

Shodan InternetDB identified several Internet-facing services and associated the host with Ubuntu, OpenSSH, Apache, Python and
Uvicorn. A hostname reported by Shodan was checked separately and no longer resolved to the investigated IP, indicating that
the hostname association may be historical.

AbuseIPDB reported a 100% Abuse Confidence Score with 248 reports from 69 reporters. SANS ISC / DShield had also observed the
IP participating in scanning activity. VirusTotal showed that 4 of 89 security vendors classified the IP as malicious,
with several additional vendors marking it as suspicious.

Taken together, the external reputation data is consistent with the automated SSH activity observed independently in the
Cowrie and Suricata logs.

The enrichment information does not identify the individual responsible for the activity and does not establish the physical
location of the operator. The local Cowrie and Suricata logs remain the primary evidence for the investigation.
