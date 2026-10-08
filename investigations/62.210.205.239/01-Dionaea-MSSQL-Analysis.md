# 1. Dionaea MSSQL Analysis

## Observed Service

All **250,538 Dionaea events** associated with `62.210.205.239` targeted the MSSQL honeypot:

```text
Protocol: mssqld
Transport: TCP
Destination port: 1433
Username: sa
```

No Dionaea evidence showed the source switching to another Dionaea-emulated service during the collected window.

## Credential Activity

Dionaea recorded **250,529 credential-bearing events**.

| Metric | Value |
|---|---:|
| Dionaea events | 250,538 |
| Credential-bearing events | 250,529 |
| Observed usernames | 1 |
| Username | `sa` |
| Unique password values | 250,304 |
| Empty-password attempts | 31 |

Approximately **99.9% of the observed password values were unique**, which is consistent with a very large dictionary or credential corpus rather than repeated testing of a small set of default passwords.

Frequently repeated values included an empty password and several common administrative/password patterns, but nearly all password candidates appeared only once.

Examples included values such as:

```text
P@ssw0rd
Admin123
Admin123!
Passw0rd
qwerty
sa@123456
sql!123
masterkey
12345678
```

These examples are attack telemetry, not credentials belonging to the honeypot host.

## Attack Rate

Credential-bearing events occurred during **1,237 active UTC minutes**, giving an average of approximately **203 credential events per active minute**.

The highest observed minute was:

```text
2026-10-02T18:56 UTC
1223 credential events / minute
```

This corresponds to more than 20 attempts per second during that minute and strongly supports automated execution.

## Activity Pattern

The source was active in several distinct bursts rather than continuously for the entire observation period.

| Period (UTC) | Approximate Dionaea events |
|---|---:|
| `2026-10-02 18:43:04–18:43:08` | 10 |
| `2026-10-02 18:53:17–2026-10-03 00:37:56` | 76,163 |
| `2026-10-03 06:41:24–06:41:27` | 10 |
| `2026-10-03 06:52:35–13:54:40` | 87,445 |
| `2026-10-03 14:05:51–21:51:50` | 86,910 |

The short ten-event bursts used empty passwords and were followed roughly ten minutes later by large dictionary runs. This repeated structure is consistent with staged automated credential testing.

## Interpretation of `connection.type=accept`

Dionaea records the connection as:

```json
"connection": {
  "protocol": "mssqld",
  "transport": "tcp",
  "type": "accept"
}
```

`accept` means that Dionaea accepted the network/application connection. It does **not** establish that the supplied `sa` password would have authenticated successfully against a real SQL Server.

## Assessment

The combination of:

- a single administrative username (`sa`),
- approximately 250,000 password candidates,
- very high request rate,
- repeated burst structure,
- exclusive TCP/1433 targeting,

is consistent with a **high-volume automated MSSQL dictionary brute-force / credential-guessing attack**.

No evidence in the collected Dionaea telemetry indicates post-authentication command execution, database manipulation or payload delivery.
