# Analysis of an EternalBlue-like SMBv1 Exploitation Attempt

## Summary

A connection to an SMB honeypot progressed from SMB1 negotiation and an IPC$ share request to a large transaction with a crafted FEA structure. Static analysis of the application stream found an NT_TRANSACT request followed by 16 TRANS2_SECONDARY messages.

**Assessment:** the combined evidence strongly supports an EternalBlue-like exploitation attempt. Successful code execution, infection and actor attribution are not established.

This sanitized case study demonstrates the progression from an informational network alert to application-stream inspection, while keeping observations separate from inference.

## Evidence

- Selected Suricata SMB, alert and flow records.
- A Dionaea SMB connection-acceptance record.
- One direction-labelled Dionaea bistream, inspected statically.
- A supporting passive fingerprint record, which does not identify the remote client's OS.

The evidence was supplied as selected records and a stream file. This is not a complete endpoint forensic investigation.

## Read the case

1. [Telemetry and relative timeline](01-Telemetry-and-Timeline.md)
2. [SMB stream analysis](02-Bistream-Analysis.md)
3. [Assessment and limitations](03-Assessment-and-Limitations.md)

Raw evidence and environment identifiers are excluded from this public case.
