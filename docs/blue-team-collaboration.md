# Blue-Team Collaboration — choosing the right visual monitoring workbooks

This is the working agreement for deciding *which* workbooks to build, prioritize, and tune with the blue team. The goal is visual monitoring that drives action, not dashboards nobody opens.

## Principles

1. **Detection follows data.** Only build a workbook for a data source you actually ingest and trust. No connector → no workbook (yet).
2. **Map every tile to ATT&CK.** If a visual doesn't map to a technique or a concrete question, cut it.
3. **Each workbook answers a hunt question**, not "here's everything." (See per-workbook "Hunt question" below.)
4. **Tune before you trust.** A workbook ships with thresholds marked "tune against your data"; the blue team owns tuning.

## Prioritization workshop (run this with the blue team)

Score each candidate workbook 1–5 on:

| Criterion | Question |
|-----------|----------|
| Data readiness | Is the source ingested and parsed correctly? |
| Threat relevance | Does it map to a technique in our threat model / top risks? |
| Actionability | If a tile lights up, do we know the next step? |
| FP burden | Will it generate noise we can't triage? |
| Coverage gap | Does it cover a tactic we're currently blind to? |

Build highest **(readiness + relevance + actionability − FP burden + gap)** first. Use workbook **09 (MITRE Coverage)** to see gaps.

## The current bench and its hunt question

| Workbook | Hunt question | Owner check |
|----------|---------------|-------------|
| 01 Meraki Network Anomaly | "Is a host scanning or beaconing externally?" | Network team validates Meraki fields |
| 02 Anomalous Travel | "Did one identity appear in two impossible places?" | Tune speed for VPN users |
| 03 Sysmon Process Hunting | "Is something running encoded/LOLBin commands?" | Confirm Sysmon config & noise |
| 04 Syslog Monitoring | "What's abnormal in appliance/Linux logs?" | Set per-host volume baselines |
| 05 Entra Identity Compromise | "Is an account being brute-forced/sprayed?" | Map result codes to your policy |
| 06 Sign-in Risk & CA Gaps | "Where is Conditional Access NOT protecting us?" | IAM team reviews CA policy |
| 07 Lateral Movement | "Is one account hopping across hosts?" | Define normal admin fan-out |
| 08 Persistence & PrivEsc | "Did privileges/accounts change unexpectedly?" | Pair with change management |
| 09 MITRE Coverage | "Where are we blind?" | Quarterly review |

## Cadence

- **Daily:** triage incidents from analytics rules + the scheduled hunt digest.
- **Weekly:** review FP rates, tune thresholds, run the manual hunting queries.
- **Quarterly:** revisit workbook 09 coverage, retire low-value tiles, add for new data sources.

## Backlog candidates (add when data exists)

DNS anomaly (if DNS logs), Defender for Endpoint device timeline, OAuth app-consent abuse (`AuditLogs`), data-exfil via cloud apps (MCAS/Defender for Cloud Apps), email threat trends (Defender for Office 365).
