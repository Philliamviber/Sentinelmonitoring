# Entra ID — Threat Hunting & Hardening (fresh-tenant setup)

This is the importable "settings" baseline for Entra ID threat hunting. Apply it first so the workbooks and rules have data.

## 1. Send Entra logs to Sentinel (required)

Deploy [`diagnostic-settings.json`](diagnostic-settings.json) at **tenant scope**. It routes these tables into the workspace:

| Table | Used by |
|-------|---------|
| `SigninLogs` | Identity compromise, anomalous travel, CA gaps |
| `AADNonInteractiveUserSignInLogs` | Token/refresh abuse hunting |
| `AuditLogs` | Persistence / privileged role changes |
| `AADServicePrincipalSignInLogs` | App / workload identity abuse |
| `RiskyUsers`, `UserRiskEvents` | Identity Protection correlation |

```bash
az deployment tenant create \
  --location <azure-region> \
  --template-file entra/diagnostic-settings.json \
  --parameters workspaceResourceId="/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/<ws>"
```

## 2. Hardening checklist (reduces the attack surface these hunts watch for)

- [ ] **Block legacy authentication** via Conditional Access — kills the MFA-bypass path in workbooks 05/06.
- [ ] **Require MFA** for all users (and phishing-resistant MFA for admins).
- [ ] **Enable Identity Protection** risk policies (sign-in + user risk) so `RiskLevelDuringSignIn` is populated.
- [ ] **Enable Continuous Access Evaluation**.
- [ ] **Restrict security defaults / app consent**; review service principals.
- [ ] **Turn on Unified Audit Log** in Microsoft Purview for richer `AuditLogs`.

## 3. Hunting starting points

| Question | Workbook / query |
|----------|------------------|
| Is someone spraying passwords? | Workbook 05 → "Password spray"; rule `password-spray.json` |
| Did a user log in from two impossible places? | Workbook 02; rule `impossible-travel.json` |
| Who is bypassing MFA via legacy auth? | Workbook 06; hunt `new-legacy-auth-user.yaml` |
| Did anyone get added to a privileged role? | Workbook 08; rule `entra-privileged-role-added.json` |

> **Note:** the queries assume default Entra schema. If you rename or filter categories in diagnostic settings, update the table references accordingly.
