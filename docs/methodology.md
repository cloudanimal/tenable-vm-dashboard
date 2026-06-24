# Methodology

How each metric is computed. The goal is to be **honest about what two point-in-time Tenable.sc exports can and cannot support** — every number here is bounded by that.

## SLA model

Two distinct SLA ideas run side by side:

### 1. Remediation-time SLA (per-severity day targets)

A finding is **past SLA** when the time since its vulnerability publish date exceeds the configured day target for its severity:

```
pastSLA(finding) = (now − vulnPubDate) > slaDays[severity]
```

- Aged from **`vulnPubDate`** (publish date), not first-seen — i.e. "how long has a fix been available and still not applied."
- Day targets are editable live in the **Configure SLAs** panel. Defaults are a placeholder **30 days for every severity** (`Critical/High/Medium/Low = 30`) — replace with your org's real targets.
- **Every past-SLA metric recalculates instantly** when you edit a target: the KPI cards, the per-severity counts, the plugin × segment matrix, "Top vulnerabilities past SLA," the OS stacked chart, and the **Most-exposed hosts** ranking all re-derive from `pastSLA()`.

### 2. Exposure SLA (exploitable findings per host)

A what-if target of **0.25 exploitable findings per host** on average:

```
Avg Past SLA = (exploitable findings past SLA) ÷ (unique IPs)
```

The card turns **red above 0.25**. The "Exploitable-vulnerability SLA by segment" panel is an editable heat-map: it compares exploitable findings per asset against the 0.25 threshold per segment, with an editable rollup row.

## Key KPIs

| KPI | Definition |
|---|---|
| **IP Addresses** | Count of unique values in the `ip` column (open + mitigated). |
| **Total Vulnerabilities** | Open finding count. |
| **Total Exploitable** | Open findings with `exploitAvailable = Yes`. |
| **Total / Avg Past SLA** | Findings past their remediation SLA; Avg = exploitable-past-SLA ÷ unique IPs (red > 0.25). |
| **Published > 1 yr** | Open findings with `vulnPubDate` older than a year. |
| **1st seen > 1 yr** | Open findings with `firstSeen` older than a year. |
| **Log4j** | Open findings referencing Log4j (plugin name, synopsis, description, CVE, or output). |
| **Mitigated findings** | Records classified mitigated (`hasBeenMitigated = Yes`). |
| **Open age (median)** | Median of `today − firstSeen` across open findings — see below. |

## Open age — and why there's no MTTR (yet)

```
Open age = median( today − firstSeen )   over open findings   (mean also shown in the tooltip)
```

**Open age** measures how long currently-open findings have been open. It's a backlog-health metric: a rising median means findings are aging in the queue. Median is shown (the mean is pulled up by a long tail of very old findings). It depends on `firstSeen` being true first-detection — rescans or asset re-identification can reset it, which would understate age.

**Why no remediation-time MTTR is displayed.** A true MTTR is `remediated − firstSeen`, but Tenable.sc exports **no remediation timestamp**, and the mitigated export is only a recent window. Any MTTR computed from a single upload is really "MTTR of whatever was remediated recently" — it skews high (long-lived vulns dominate a short remediation window) and isn't an org-wide figure, so it's intentionally **not shown**.

A correct MTTR needs **two scans**: a finding open in the prior cumulative export and absent in the current one was remediated between the scan dates, giving `MTTR ≈ scanDate(current) − firstSeen`. That's the planned **compare mode** ([roadmap](roadmap.md)); MTTR falls out of it for free.

## Most-exposed hosts

Hosts are ranked by **count of open findings past SLA** (respecting the panel's severity filter), so the list reshuffles live as you change SLA day targets. Each row expands to show that host's open findings (severity, plugin, CVE, port, exploit and SLA status).

## Privacy

All computation is client-side. The page makes **zero outbound requests** with your data, and the libraries (Chart.js, SheetJS, PapaParse) are vendored in `vendor/`, so it runs fully offline / air-gapped. Open your browser's Network tab and confirm.
