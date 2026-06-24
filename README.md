# Tenable VM Dashboard

A **100% browser-local** dashboard for Tenable Security Center (Tenable.sc) vulnerability data. Drop in your `cumulative` and `mitigated` vulnerability-detail exports and get high-value vulnerability-management reports instantly — **the data never leaves your browser**. No server, no upload, no SaaS.

> Open your browser's Network tab while you use it: there are zero outbound requests with your data. Everything is parsed and rendered client-side.

**Live page:** https://cloudanimal.github.io/tenable-vm-dashboard/

## Why

Security teams often can't push raw scan data to a third-party tool for analysis — it's sensitive, and policy frequently forbids it. This page does the analysis entirely in the browser using JavaScript, so you get the convenience of a hosted tool with the privacy of a local one.

## What it reports

- **Executive KPIs** — assets analyzed, open findings, exploitable open, exploitable-per-asset vs SLA, mitigated count, remediation rate, mean time to remediate (MTTR).
- **Exploitable-vulnerability SLA by business unit** — the headline metric: exploitable (`exploitAvailable = Yes`) vulnerabilities per asset against a configurable **0.25 SLA**, with within/over-SLA status. Asset counts are editable per BU.
- **Severity distribution** of open findings.
- **Top exploitable vulnerabilities** — plugin, severity, CVE, exploit framework, instance + host counts.
- **Exposure by operating system and by cloud** (Azure / AWS).
- **Remediation throughput** — mitigated findings by month.
- **Most-exposed hosts** by exploitable finding count.

## Input formats

Provide either:

1. **One Excel workbook** (`.xlsx`) containing worksheets whose names include **`cumulative`** and **`mitigated`**, or
2. **Separate CSV or JSON files** — each set is auto-classified (records carrying a `mitigatedDate` are treated as the mitigated database; everything else as cumulative).

The columns expected are the standard Tenable.sc `/analysis` `vulndetails` fields (`pluginID`, `pluginName`, `severity`, `exploitAvailable`, `repository`, `ip`, `dnsName`, `operatingSystem`, `cloudProvider`, `firstSeen`, `mitigatedDate`, `cve`, `vprScore`, …). Extra columns are ignored; missing ones degrade gracefully.

## Try it

Click **Load sample data** on the page, or upload [`sample-data/Tenable_SC_VulnDetail_Sample.xlsx`](sample-data/Tenable_SC_VulnDetail_Sample.xlsx) — a fully synthetic dataset (1,000 hosts across 5 business units, cumulative + mitigated, full field schema).

## Export

- **Export PDF** — print-optimized report of the whole dashboard.
- **Export CSV** — the per-BU SLA summary.

## Run locally

```bash
git clone https://github.com/cloudanimal/tenable-vm-dashboard
cd tenable-vm-dashboard
python3 -m http.server 8000   # serve so "Load sample data" can fetch the workbook
# open http://localhost:8000
```

Uploading your own files works even from `file://`; only the bundled **Load sample data** button needs the page served over HTTP.

## Tech

Single static `index.html`. [SheetJS](https://sheetjs.com) for XLSX/CSV parsing, [Chart.js](https://www.chartjs.org) for visualization. No build step, no dependencies to install.

## Note

The bundled sample data is entirely synthetic and does not represent any real organization or environment.

## License

MIT
