# MITRE ATT&CK Coverage

Reference map of what this pack detects. Workbook 09 renders this live; this file is the static, reviewable version.

| Tactic | Technique | ID | Covered by | Data source |
|--------|-----------|----|-----------|-------------|
| Reconnaissance / Discovery | Network Service Discovery | T1046 | WB01, `meraki-port-scan` | CommonSecurityLog |
| Command and Control | Application Layer Protocol | T1071 | WB01, `meraki-beaconing` | CommonSecurityLog |
| Initial Access | Valid Accounts | T1078 | WB02, WB05, `impossible-travel` | SigninLogs |
| Lateral Movement | Remote Services (RDP) | T1021.001 | WB07 | SecurityEvent |
| Lateral Movement | Remote Services (SMB) | T1021.002 | WB07 | SecurityEvent |
| Execution | Command & Scripting Interpreter | T1059 | WB03, `sysmon-encoded-powershell` | Event (Sysmon) |
| Defense Evasion | System Binary Proxy Execution | T1218 | WB03, `sysmon-rare-parent-child` | Event (Sysmon) |
| Credential Access | OS Credential Dumping | T1003 | WB03 | Event (Sysmon) |
| Credential Access | Brute Force | T1110 | WB05, WB07, `password-spray` | SigninLogs / SecurityEvent |
| Persistence / Defense Evasion | Modify Authentication Process | T1556 | WB05, WB06, `new-legacy-auth-user` | SigninLogs |
| Persistence | Create Account | T1136 | WB08 | SecurityEvent |
| Persistence / PrivEsc | Account Manipulation | T1098 | WB08, `entra-privileged-role-added` | SecurityEvent / AuditLogs |
| Persistence | Create or Modify System Process | T1543.003 | WB08 | Event |
| Privilege Escalation | Valid Accounts: Cloud | T1078.004 | WB08 | AuditLogs |

## Known blind spots (backlog)

- **Exfiltration** (T1041 / T1567) — needs cloud-app or DLP telemetry.
- **Collection / Email** (T1114) — needs Defender for Office 365.
- **DNS-based C2** (T1071.004) — needs DNS query logs.
- **Container / cloud control-plane** — needs Azure Activity / AKS logs.

Review this table with the blue team quarterly (see [blue-team-collaboration.md](blue-team-collaboration.md)).
