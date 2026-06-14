# Deployment Guide

Order matters: get data flowing first, then content.

## Prerequisites

- A Log Analytics workspace with **Microsoft Sentinel** enabled.
- **Azure CLI** (`az`) signed in, or use the portal.
- An **Azure Maps account** (for workbook 02's map tile).
- Roles: *Microsoft Sentinel Contributor* on the workspace; *Global Administrator* or *Security Administrator* for the tenant-scope Entra diagnostic settings.

## 1. Connect data sources

| Source | How |
|--------|-----|
| Entra ID | Deploy `entra/diagnostic-settings.json` (tenant scope) |
| Cisco Meraki | Forward syslog to a Linux collector running AMA with the CEF/Syslog DCR → `CommonSecurityLog` |
| Sysmon | AMA DCR collecting `Microsoft-Windows-Sysmon/Operational` → `Event` |
| Linux / appliances | AMA Syslog DCR → `Syslog` |
| Windows Security | AMA Security Events DCR → `SecurityEvent` |

## 2. Deploy analytics rules

```bash
for f in analytics-rules/*.json; do
  az deployment group create --resource-group <rg> \
    --template-file "$f" --parameters workspace=<workspace-name>
done
```

## 3. Import workbooks

Portal: **Sentinel → Workbooks → Add workbook → Edit → `</>` Advanced Editor** → paste a `workbooks/0X-*.workbook.json` → **Apply → Save** (name it, pick the workspace).

Workbook 02 will prompt to bind to your Azure Maps account on first map render.

## 4. Deploy SOAR playbooks

```bash
az deployment group create --resource-group <rg> \
  --template-file playbooks/notify-soc.logicapp.json \
  --parameters teamsChannelWebhook=<webhook-url>
```

Then **authorize the API connections** (portal → the connection → *Authorize*) and grant the playbook's managed identity **Microsoft Sentinel Responder**. Bind it with an **automation rule** (see [`playbooks/README.md`](../playbooks/README.md)).

## 5. Add hunting queries

Portal: **Sentinel → Hunting → New query**, paste from `hunting-queries/*.yaml`. Or import via the Sentinel `repositories` feature / API.

## Validation

- Confirm each table has data: `SigninLogs | take 5`, `CommonSecurityLog | take 5`, etc.
- Open each workbook and confirm tiles render (empty tiles usually mean the data source isn't connected yet).
- Let analytics rules run one cycle, then check **Sentinel → Incidents**.

## Tuning

Start every rule **disabled or at low severity**, watch a week of results, then adjust thresholds in the rule's `query`/`triggerThreshold`. See per-file comments.
