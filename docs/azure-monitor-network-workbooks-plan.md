# Azure Monitor Network Workbooks Plan (13–32)

> Plan produced by the `tech-lead` planner; built in-thread. This pack extends the
> identity/Meraki/Sysmon pack (01–12) into **Azure network observability**.

## 1. Rationale + the 20 workbooks

The existing pack (01–12) is identity- and Meraki/Sysmon-focused. This pack extends the repo into
**Azure network observability**: availability, syslog scale-out, cross-region/egress
cost-and-exfil signal, Virtual WAN, and Windows host firewall denies. Every file follows the
house format established by `04-syslog-monitoring.workbook.json` and
`10-meraki-sdwan-site-to-site-vpn.workbook.json`:

- Root object: `{"version": "Notebook/1.0", "items": [...], "$schema": "...workbook.json"}`
- `items[0]` = a `type: 1` Markdown title block with a one-line purpose, data-source note, and a
  "starting template — tune thresholds" caveat. ATT&CK tags only where security-relevant
  (firewall denies, egress exfil); availability/traffic workbooks use an operational framing.
- `items[1]` = a `type: 9` `TimeRange` parameter pill.
- Remaining items = `type: 3` `KqlItem/1.0` tiles, `queryType: 0`,
  `resourceType: microsoft.operationalinsights/workspaces`, each with a `title`, `timeContext`,
  and a `visualization` (`table` / `timechart` / `barchart` / `tiles`).
- Filenames: `NN-kebab-case.workbook.json` in `workbooks/`.

**Data-source caveat baked into every title block:** these tables only exist if the matching
connector/diagnostic is configured. `AzureNetworkAnalytics_CL` requires Traffic Analytics on NSG
flow logs; `NWConnectionMonitorTestResult`/`NetworkMonitoring` requires Connection Monitor;
`WindowsFirewall` requires the Windows Firewall DCR. Empty tiles usually mean the source isn't
ingesting yet, not a broken query. KQL is a tunable starting template, not production-tuned.

### Theme 1 — Network availability (13–16)

| # | File | Purpose | Primary table(s) | Panels |
|---|------|---------|------------------|--------|
| 13 | `13-heartbeat-availability.workbook.json` | VM/agent up-down and missing-heartbeat detection | `Heartbeat` | last-seen per Computer + stale (>15m); heartbeat count timechart by OSType; agents that stopped reporting; heartbeat by version |
| 14 | `14-connection-monitor-availability.workbook.json` | Connection Monitor reachability & latency | `NWConnectionMonitorTestResult`, fallback `NetworkMonitoring` | pass/fail % by test/destination; RTT (avg/p95) timechart; failed checks by source→dest; checks with packet loss |
| 15 | `15-gateway-availability.workbook.json` | VPN/ExpressRoute gateway tunnel health & throughput | `AzureDiagnostics` (gateway), `AzureMetrics` | tunnel up/down events; gateway bytes timechart; disconnect/renegotiation messages; bandwidth utilization |
| 16 | `16-endpoint-availability-slo.workbook.json` | Availability SLO rollup | `Heartbeat`, `NWConnectionMonitorTestResult` | composite availability % tiles; SLO trend; worst-performing endpoints; downtime-minutes |

### Theme 2 — Syslog traffic monitoring (17–20)

> Complements existing `04-syslog-monitoring` (auth/severity focus); these focus on traffic/volume.

| # | File | Purpose | Primary table(s) | Panels |
|---|------|---------|------------------|--------|
| 17 | `17-syslog-volume-trends.workbook.json` | Ingestion volume & per-host anomaly | `Syslog` | events/hour by Computer; top talkers; z-score spike vs 7d baseline; new hosts first-seen |
| 18 | `18-syslog-facility-severity.workbook.json` | Facility × severity breakdown & error bursts | `Syslog` | facility × severity matrix; err/crit burst timechart; top ProcessName by errors; severity mix per host |
| 19 | `19-syslog-network-appliance.workbook.json` | Network-device syslog (firewall/router) | `Syslog` | events by Facility local0–7; deny/drop keyword extraction; interface link-flap; top message patterns |
| 20 | `20-syslog-ingestion-health.workbook.json` | Pipeline health / silent-host detection | `Syslog`, `Heartbeat` | syslog vs heartbeat-only gap; hosts gone silent; ingestion latency; facility coverage per host |

### Theme 3 — Inter-region egress monitoring (21–24)

| # | File | Purpose | Primary table(s) | Panels |
|---|------|---------|------------------|--------|
| 21 | `21-interregion-egress-overview.workbook.json` | Cross-region traffic volume & top flows | `AzureNetworkAnalytics_CL` | bytes by src→dest region (cross-region only); egress timechart; top region pairs; outbound breakdown |
| 22 | `22-interregion-egress-cost.workbook.json` | Egress-cost estimator (bytes → GB) | `AzureNetworkAnalytics_CL` | GB per region pair; trend; top egressing workloads/subnets; total cross-region GB tiles |
| 23 | `23-interregion-egress-anomaly.workbook.json` | Egress spike / data-exfil signal | `AzureNetworkAnalytics_CL` | z-score spike per region pair; new region-pair flows; outbound bytes to internet; top source IPs — **ATT&CK T1567 / TA0010 Exfiltration** |
| 24 | `24-peering-bandwidth.workbook.json` | VNet peering / gateway bandwidth | `AzureMetrics`, `AzureNetworkAnalytics_CL` | peering in/out timechart; utilization vs limit; denied-by-peering flows; top peering links |

### Theme 4 — Virtual WAN (vWAN) traffic analysis (25–28)

| # | File | Purpose | Primary table(s) | Panels |
|---|------|---------|------------------|--------|
| 25 | `25-vwan-hub-overview.workbook.json` | Virtual hub health & routing overview | `AzureDiagnostics` (`VIRTUALHUBS`) | events by hub; route/BGP status; hub throughput timechart; per-hub category breakdown |
| 26 | `26-vwan-vpn-gateway.workbook.json` | vWAN site-to-site VPN gateway traffic | `AzureDiagnostics` (`VPNGATEWAYS`) | tunnel up/down per connection; bytes in/out timechart; IKE/renegotiation; top connections |
| 27 | `27-vwan-expressroute.workbook.json` | vWAN ExpressRoute gateway traffic | `AzureDiagnostics` (`EXPRESSROUTEGATEWAYS`) | ER bits in/out timechart; ARP/BGP route status; scale-unit utilization; top circuits |
| 28 | `28-vwan-flow-analysis.workbook.json` | vWAN flow-log traffic & top talkers | vWAN flow logs via `AzureNetworkAnalytics_CL` / `AzureDiagnostics` | allowed vs denied flows; top src/dest by bytes; flow volume by hub; cross-hub traffic matrix |

### Theme 5 — Windows Firewall deny-traffic parsing (29–32)

| # | File | Purpose | Primary table(s) | Panels |
|---|------|---------|------------------|--------|
| 29 | `29-winfw-deny-overview.workbook.json` | Dropped/blocked traffic overview | `WindowsFirewall` (Action Drop/Block), `SecurityEvent` | denies by Computer; deny timechart; top blocked dest ports; inbound vs outbound — **ATT&CK T1562.004** |
| 30 | `30-winfw-blocked-connections.workbook.json` | EventID 5157 blocked-connection parsing | `SecurityEvent` (5157) | blocked connections by SourceAddress→DestPort; timechart; top blocking processes; external source IPs |
| 31 | `31-winfw-blocked-packets.workbook.json` | EventID 5152 WFP packet-block parsing | `SecurityEvent` (5152) | blocked packets by Protocol/DestPort; timechart; top filter origins; repeated-block source IPs (scan signal) |
| 32 | `32-winfw-rule-changes.workbook.json` | Firewall policy/rule-change monitoring | `SecurityEvent` (5031 + 4946/4947/4950) | rule add/modify/delete by user; app-blocked-from-accepting (5031); policy-change timechart; firewall-disabled events — **ATT&CK T1562.004** |

## 2. Delegation plan (as planned by tech-lead)

| Step | Specialist | Task | Depends on | Parallel? |
|------|-----------|------|------------|-----------|
| 1 | requirements-analyst | Confirm scope: 20 files, themes, table assumptions, acceptance criteria | — | No |
| 2 | detection-engineer | Draft/validate KQL for all 20 (correct tables/columns; z-score/baseline patterns) | 1 | with 3 |
| 3 | tech-writer | Draft per-workbook title-block Markdown (purpose, caveat, ATT&CK) | 1 | with 2 |
| 4 | implementer | Build 20 `Notebook/1.0` JSON files, slot in KQL + title text | 2, 3 | No |
| 5 | detection-engineer | KQL sanity re-pass inside assembled JSON | 4 | with 6 |
| 6 | code-reviewer | Validate JSON parses, format/naming parity | 4 | with 5 |
| 7 | docs-maintainer | Update README + CHANGELOG `[0.2.0]`; save this plan | 5, 6 | No |

> Execution note: the main thread built the 20 JSON files in-thread (matching the 04/10 house
> format) and ran the JSON validation that mirrors `.github/workflows/validate.yml`, rather than
> dispatching every step as a separate cold subagent.

## 3. Acceptance criteria

1. **Valid JSON** — all 20 files parse (repo CI passes); query newlines encoded as `\n`.
2. **Format parity** — root `version` `"Notebook/1.0"`, `items[0]` Markdown title, `items[1]`
   `TimeRange` pill, then `type: 3` KQL tiles (`queryType: 0`,
   `resourceType: microsoft.operationalinsights/workspaces`).
3. **Coverage** — files `13`–`32`, 4 per theme, each 2–4 KQL tiles with `title` + `visualization`.
4. **KQL as templates** — correct primary table(s); thresholds called out as tunable;
   `_CL` columns flagged as connector-dependent.
5. **Portal-importable** — renders in Sentinel → Workbooks → Advanced Editor (empty tiles OK when
   the source connector isn't ingesting).
6. **Docs updated** — README count/table + CHANGELOG reflect the new pack.

### Open questions flagged by tech-lead (defaults applied for this build)
1. **vWAN flow logs (WB 28):** defaulted to `AzureNetworkAnalytics_CL` (with `AzureDiagnostics` note).
2. **WindowsFirewall vs SecurityEvent:** WB 29 assumes both; 30–32 use `SecurityEvent` EventIDs.
3. **Watchlist reuse:** self-contained for this pack (no `HostInventory` dependency).
4. **CHANGELOG:** bumped to `[0.2.0]`.
