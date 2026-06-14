# Changelog

All notable changes to this pack are documented here. Format based on [Keep a Changelog](https://keepachangelog.com/).

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
