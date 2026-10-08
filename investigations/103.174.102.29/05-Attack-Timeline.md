## 5. Attack Timeline

The activity from source IP `103.174.102.29` was observed for approximately 10 hours and 13 minutes on 22 September 2026.

| Time (UTC) | Event |
|---|---|
| `09:57:04` | First SSH activity from `103.174.102.29` observed by Cowrie. |
| `09:59:54` | SSH connection from source port `58394` established. Suricata identified the client as `SSH-2.0-Go`. |
| `09:59:55` | Authentication succeeded using the credentials `root / <REDACTED>`. |
| `09:59:56` | The command `echo xsec` was executed. |
| `09:59:56` | The successful SSH session was closed shortly afterwards. |
| `~09:57–13:46` | Repeated credential attempts primarily targeted the `root` account. |
| `~13:47` | Credential testing shifted from `root` to `ubuntu`. |
| `13:47–20:10` | Structured and repeated SSH credential attempts continued against the `ubuntu` account. |
| `20:10:35` | Last activity from the source IP observed in the Cowrie dataset. |

### Successful Session

The only successful authentication used:

- **Username:** `root`
- **Password:** `<REDACTED>`
- **Session ID:** `db841bd9b899`
- **SSH client:** `SSH-2.0-Go`
- **Source port:** `58394`

After authentication, the only observed command was:

```bash
echo xsec

No additional post-authentication commands were observed before the session ended.
```
