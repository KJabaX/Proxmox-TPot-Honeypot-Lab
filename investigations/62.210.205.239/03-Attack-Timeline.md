# 3. Attack Timeline

## Overview

The source `62.210.205.239` was observed for approximately **27 hours and 9 minutes** in the Dionaea dataset.

The activity was not continuous. It consisted of several high-volume attack bursts separated by pauses.

| Time (UTC) | Event |
|---|---|
| `2026-10-02 18:43:04` | First Dionaea MSSQL event from the source. |
| `18:43:04–18:43:08` | Initial burst of 10 `sa` authentication attempts using an empty password. |
| `18:53:17` | Large password-dictionary run begins. |
| `18:55–18:56` | Highest observed attack rate; up to 1,223 credential events in one minute. |
| `2026-10-03 00:37:56` | First large dictionary run ends. |
| `00:37:56–06:41:24` | Approximately six-hour pause in Dionaea activity. |
| `06:41:24–06:41:27` | Second short 10-event empty-password burst. |
| `06:52:35` | Second large dictionary run begins. |
| `13:54:40` | Second large run ends. |
| `14:05:51` | Third large dictionary run begins. |
| `21:51:50` | Last Dionaea event from the source. |
| `21:57:24` | Last related Suricata event in the collected evidence. |

## Attack Bursts

| Period | Approx. events |
|---|---:|
| `18:43:04–18:43:08` | 10 |
| `18:53:17–00:37:56` | 76,163 |
| `06:41:24–06:41:27` | 10 |
| `06:52:35–13:54:40` | 87,445 |
| `14:05:51–21:51:50` | 86,910 |

The two short ten-event bursts used empty passwords. Each was followed roughly ten minutes later by a much larger dictionary run.

## Interpretation

The repeated pattern of:

1. short probe burst,
2. brief delay,
3. sustained high-rate dictionary attack,

is consistent with an automated workflow rather than interactive manual activity.

No evidence in the collected bundle indicates that the source changed attack technique after the credential-guessing phases or delivered a payload after the MSSQL activity.
