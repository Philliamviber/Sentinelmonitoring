# Community Azure Monitor / Sentinel Workbooks — research

Survey of existing community + Microsoft-published workbooks worth reusing or borrowing from,
mapped to our five themes. Use these as references for KQL patterns and visualizations rather than
reinventing them. (Links current as of June 2026.)

## Primary source repos

| Source | What it is | Why it matters here |
|--------|-----------|---------------------|
| [Azure/Azure-Sentinel → `Workbooks/`](https://github.com/Azure/Azure-Sentinel/tree/master/Workbooks) | Microsoft's official Sentinel content repo; hundreds of workbook JSONs + `WorkbooksMetadata.json` catalog | The canonical reference for the exact `Notebook/1.0` format we mirror; search it for firewall/network/VWAN templates |
| [microsoft/Application-Insights-Workbooks](https://github.com/microsoft/Application-Insights-Workbooks) | Open-source workbook templates engine + samples | Ground truth for the workbook schema/`$schema` URL and item types |
| [Azure Monitor Network Insights](https://learn.microsoft.com/en-us/azure/network-watcher/network-insights-overview) | Built-in, zero-config network workbooks (topology, health, metrics) | Free baseline for **availability** (Connection Monitor), **egress** (NSG/VNet flow logs + Traffic Analytics) — onboard before building custom |
| [Tobias Zimmergren — open-source workbooks roundup](https://zimmergren.net/open-source-azure-monitor-microsoft-sentinel-workbooks/) | Curated list of security-focused community workbooks | Good "nice-to-have" shortlist and links |
| [MYoussef23/Azure-Sentinel-Workbooks](https://github.com/MYoussef23/Azure-Sentinel-Workbooks) | Community collection of custom Sentinel workbooks | Extra patterns for security monitoring/visualization |

## Theme-by-theme mapping

### Network availability (our 13–16)
- **Azure Monitor Network Insights** built-in workbook — topology + health + metrics, no config; integrates Connection Monitor connectivity insights and Traffic Analytics. *Onboard this first; our 13–16 add Heartbeat/SLO rollups it doesn't focus on.*
- WAN-monitoring solution workbooks expose **VPN Status, Device Availability, Device Performance, Device Bandwidth, WAN Latency** — pattern source for WB 14/16.

### Syslog traffic monitoring (our 17–20)
- No single canonical community "syslog traffic" workbook; closest references are generic
  Sentinel **Workspace Usage / Data Collection Health** workbooks in `Azure-Sentinel/Workbooks`
  (ingestion-volume + silent-host patterns reused in WB 20). Our pack is largely original here.

### Inter-region egress monitoring (our 21–24)
- **Traffic Analytics** (part of Network Insights) over `AzureNetworkAnalytics_CL` is the
  authoritative source for cross-region/egress flows — our 21–24 query the same table with a
  region-pair + cost + anomaly lens.
- **Network Firewall Events** community workbook (visualizes `CommonSecurityLog` from Azure
  Firewall, Palo Alto, Cisco ASA, Check Point, Fortinet) — pattern source for top-talker/action
  filtering reused in WB 24/28.

### Virtual WAN traffic analysis (our 25–28)
- [Monitoring Virtual WAN using Azure Monitor Insights](https://learn.microsoft.com/en-us/azure/virtual-wan/azure-monitor-insights) —
  Microsoft's **pre-packaged Virtual WAN metrics workbook** showing metrics at VWAN/hub/gateway/
  connection levels, plus an autodiscovered topology map. *Deploy this alongside our 25–27, which
  add log-based (`AzureDiagnostics`) routing/tunnel detail the metrics workbook lacks.*

### Windows Firewall deny-traffic parsing (our 29–32)
- [Azure-Sentinel `EventAnalyzer.json` workbook](https://github.com/Azure/Azure-Sentinel/blob/master/Workbooks/EventAnalyzer.json) —
  queries `SecurityEvent` where `EventID in (5031, 5150, 5151, 5154, 5155, 5156, 5157, 5158, 5159)`;
  direct pattern source for WB 30/31/32.
- [ASIM Network Session parser for Windows Firewall](https://github.com/Azure/Azure-Sentinel/blob/master/Parsers/ASimNetworkSession/Parsers/ASimNetworkSessionMicrosoftWindowsEventFirewall.yaml) —
  normalizes WindowsEvent/SecurityEvent firewall events (5150–5159) to the ASIM `NetworkSession`
  schema. *Reference for our `parsers/WindowsFirewallDeny.kql` ASIM-style parser.*
- [Charbel Nemnom — collecting Windows Firewall events to Sentinel](https://charbelnemnom.com/collect-windows-firewall-events-to-sentinel/) —
  DCR setup guidance for getting the deny events flowing (prereq for WB 29–32).

## Takeaways
1. **Deploy the free built-ins first** (Network Insights, Virtual WAN Insights, Traffic Analytics) —
   they cover topology/metrics our custom log-based workbooks intentionally don't duplicate.
2. **Borrow KQL, not files** — `EventAnalyzer.json` and the ASIM firewall parser are the strongest
   direct references for the Windows Firewall theme.
3. Our pack's differentiators: SLO rollups (16), syslog ingestion-health (20), egress **cost +
   anomaly/exfil** lens (22/23), and log-level vWAN routing/tunnel detail (25–28).

## Sources
- [Azure/Azure-Sentinel Workbooks](https://github.com/Azure/Azure-Sentinel/tree/master/Workbooks) · [WorkbooksMetadata.json](https://github.com/Azure/Azure-Sentinel/blob/master/Workbooks/WorkbooksMetadata.json)
- [microsoft/Application-Insights-Workbooks](https://github.com/microsoft/Application-Insights-Workbooks)
- [Network Insights overview](https://learn.microsoft.com/en-us/azure/network-watcher/network-insights-overview)
- [Monitoring Virtual WAN using Azure Monitor Insights](https://learn.microsoft.com/en-us/azure/virtual-wan/azure-monitor-insights)
- [Azure-Sentinel EventAnalyzer workbook](https://github.com/Azure/Azure-Sentinel/blob/master/Workbooks/EventAnalyzer.json)
- [ASIM NetworkSession Windows Firewall parser](https://github.com/Azure/Azure-Sentinel/blob/master/Parsers/ASimNetworkSession/Parsers/ASimNetworkSessionMicrosoftWindowsEventFirewall.yaml)
- [Visualize your data using workbooks in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/monitor-your-data)
- [Tobias Zimmergren — open-source workbooks](https://zimmergren.net/open-source-azure-monitor-microsoft-sentinel-workbooks/)
- [MYoussef23/Azure-Sentinel-Workbooks](https://github.com/MYoussef23/Azure-Sentinel-Workbooks)
- [Charbel Nemnom — Windows Firewall events to Sentinel](https://charbelnemnom.com/collect-windows-firewall-events-to-sentinel/)
