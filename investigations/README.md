# Investigation Cases

This directory contains selected case studies derived from honeypot telemetry.

Public case notes intentionally exclude raw log exports, packet captures, evidence archives, captured binaries, private infrastructure addressing, exact called numbers, management details, plaintext honeypot credentials, and other environment-specific data that is not required to understand the investigation.

Some older cases retain source IP addresses as technical indicators. An IP address identifies an observed network source at a point in time; it does not by itself establish the identity, intent, or ownership of a human actor.

## Cases

- [EternalBlue-like SMBv1 Attempt](smbv1-eternalblue-like-attempt/) - static stream analysis, telemetry correlation and explicit evidence limitations.

- [SIP/PBX Dial-Plan Probing](sip-pbx-dial-plan-probing/) - sanitized public analysis of automated SIP INVITE activity and dial-plan probing.
- [MSSQL Investigation](62.210.205.239/) - high-volume MSSQL credential-guessing analysis.
- [Cowrie Investigation](103.174.102.29/) - SSH honeypot investigation.

The detailed collection and analysis process is documented under [T-Pot-Attack-Analysis](../T-Pot-Attack-Analysis/).
