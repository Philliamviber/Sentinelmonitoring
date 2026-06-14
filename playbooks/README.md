# SOAR — Sentinel-native automation & scheduling

This folder uses **Microsoft Sentinel's built-in SOAR** (Logic Apps playbooks + automation rules). No third-party SOAR (e.g. Cortex XSOAR) is required.

## What's here

| File | Trigger | Purpose | MITRE link |
|------|---------|---------|------------|
| `notify-soc.logicapp.json` | Sentinel **incident created** | Posts the incident to a SOC Teams channel | (all) |
| `scheduled-hunt-digest.logicapp.json` | **Recurrence** (daily 07:00) | Runs a hunt KQL and emails a digest — this is how you *schedule* threat hunting in Sentinel-native SOAR | T1110 |

## The three Sentinel automation building blocks

1. **Scheduled analytics rules** (`../analytics-rules/`) — run KQL on a frequency (`queryFrequency`) and raise incidents. This is the primary "schedule + target" mechanism.
2. **Automation rules** — bind a playbook to a trigger (e.g. "when any High incident is created, run `Notify-SOC`"). Created in *Sentinel → Automation*. Example ARM resource:

```json
{
  "type": "Microsoft.OperationalInsights/workspaces/providers/automationRules",
  "apiVersion": "2022-11-01-preview",
  "name": "[concat(parameters('workspace'),'/Microsoft.SecurityInsights/', 'a1b2c3d4-0000-0000-0000-000000000001')]",
  "properties": {
    "displayName": "Notify SOC on High incidents",
    "order": 1,
    "triggeringLogic": {
      "isEnabled": true,
      "triggersOn": "Incidents",
      "triggersWhen": "Created",
      "conditions": [
        {"conditionType": "Property", "conditionProperties": {"propertyName": "IncidentSeverity", "operator": "Equals", "propertyValues": ["High"]}}
      ]
    },
    "actions": [
      {"order": 1, "actionType": "RunPlaybook", "actionConfiguration": {"logicAppResourceId": "<resourceId of Notify-SOC>", "tenantId": "<tenantId>"}}
    ]
  }
}
```

3. **Playbooks** (these Logic Apps) — the actions: notify, enrich, or remediate.

## After deploying a playbook

Logic App API connections (`azuresentinel`, `azuremonitorlogs`, `office365`) are created on first deploy but must be **authorized once** in the portal (open the connection → *Authorize*). Grant the playbook's managed identity the **Microsoft Sentinel Responder** role on the workspace so it can read/update incidents.

> These are starting templates. Validate connection authorization and least-privilege before enabling in production.
