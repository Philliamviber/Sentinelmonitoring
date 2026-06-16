# Changelog

All notable changes to this pack are documented here. Format based on [Keep a Changelog](https://keepachangelog.com/).

## [0.2.0] - 2026-06-15

### Added
- **20-workbook Azure network observability pack** (`workbooks/13`–`32`), 4 per theme:
  - **Network availability** (13–16): Heartbeat up/down, Connection Monitor reachability & latency, VPN/ExpressRoute gateway health, endpoint availability SLO rollup.
  - **Syslog traffic monitoring** (17–20): volume/anomaly trends, facility×severity & error bursts, network-appliance parsing, ingestion health & silent-host detection.
  - **Inter-region egress** (21–24): cross-region overview, egress-cost (GB) estimator, spike/exfil anomaly, VNet-peering & gateway bandwidth.
  - **Virtual WAN traffic** (25–28): hub health/routing, site-to-site VPN gateway, ExpressRoute gateway, flow analysis & top talkers.
  - **Windows Firewall deny** (29–32): deny overview (`WindowsFirewall` + `SecurityEvent`), 5157 blocked-connection parsing, 5152 blocked-packet parsing, policy/rule-change monitoring.
- **`workbooks/parsers/WindowsFirewallDeny.kql`** — ASIM-aligned parser unioning `SecurityEvent` 5152/5157 and the `WindowsFirewall` table into a normalized deny schema.
- **Docs**: `azure-monitor-network-workbooks-plan.md` (tech-lead build spec) and `community-workbooks-research.md` (reusable community/Microsoft workbook references).

### Notes
- KQL is starting templates; `_CL` column names (Traffic Analytics) and table availability are connector-dependent — verify before alerting.

## [0.1.0] - 2026-06-14

### Added
- **9 Sentinel workbooks** (MITRE ATT&CK-mapped): Meraki network anomaly, anomalous/impossible travel (Azure Maps), Sysmon process hunting, syslog monitoring, Entra identity compromise, sign-in risk & Conditional Access gaps, lateral movement, persistence & privilege escalation, and a MITRE coverage overview.
- **5 scheduled analytics rules** (ARM): password spray, impossible travel, Meraki port scan, Sysmon encoded PowerShell, Entra privileged-role addition.
- **Sentinel-native SOAR**: `Notify-SOC` (incident-triggered) and `Scheduled-Hunt-Digest` (recurrence) playbooks, plus automation-rule guidance.
- **3 hunting queries** (YAML): new legacy-auth user, Meraki beaconing, rare Sysmon process.
- **Entra ID setup**: tenant-scope diagnostic-settings ARM template + hunting/hardening guide.
- **Docs**: deployment guide, blue-team collaboration playbook, SOAR runbook, MITRE coverage map.
- **CI**: JSON validation workflow.

### Notes
- All detections are starting templates and require threshold tuning against real telemetry.
- SOAR is Microsoft Sentinel-native (no Cortex XSOAR). Geo uses Azure Maps (Bing Maps for Enterprise is deprecated).
