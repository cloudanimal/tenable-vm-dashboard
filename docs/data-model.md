# Data model

The dashboard consumes **Tenable Security Center (Tenable.sc) `/analysis` vulnerability-detail exports**. Everything is parsed and computed in the browser — no data is uploaded.

## The two inputs

The real-world workflow is **two CSV exports from the same Tenable.sc `/analysis` `vulndetails` view**, changing only the `sourceType`:

| Input | Tenable export | Meaning |
|---|---|---|
| **Cumulative (open)** | `sourceType=cumulative` | Currently open / detected vulnerabilities |
| **Mitigated** | `sourceType=patched` | Findings that are no longer detected (remediated) |

You can also drop in a single `.xlsx` workbook whose worksheet names include `cumulative` and `mitigated`, or JSON. Files in the generic drop zone are auto-classified — any record with `hasBeenMitigated = Yes` is treated as mitigated.

> Why two labelled pickers and not pure auto-detect? A *cumulative* CSV doesn't necessarily carry a field that distinguishes it from a mitigated one, so the labelled pickers (`forcedKind`) guarantee correct classification regardless of column contents.

## Schema

The exports use the standard Tenable.sc `/analysis` `vulndetails` fields (62 columns). Extra columns are ignored; missing ones degrade gracefully. The fields the dashboard actually relies on:

| Field | Used for |
|---|---|
| `pluginID`, `pluginName` | Finding identity; "Top vulnerabilities past SLA" grouping |
| `severity` | Critical / High / Medium / Low / Info — drives SLA targets, donut, filters |
| `exploitAvailable` | `Yes` ⇒ counted as exploitable (the 0.25 per-host SLA model) |
| `cve` | Shown in host detail rows; Log4j detection |
| `ip`, `dnsName` | Host identity (`dnsName` preferred, falls back to `ip`); unique-IP KPI |
| `operatingSystem` | "Vulnerabilities by OS" breakdown |
| `port`, `protocol` | Host detail rows |
| `repository` | **Segment / business unit** — the whole dashboard segments on this |
| `firstSeen`, `lastSeen` | Open-age metric (`today − firstSeen`); "first seen > 1yr" KPI |
| `vulnPubDate` | **SLA aging** — past-SLA is measured from publish date |
| `hasBeenMitigated` | Mitigation classification (`Yes` ⇒ mitigated) |
| `vprScore`, `cvssV3BaseScore` | Available for enrichment / sorting |

### Notable modeling choices

- **Mitigation is identified by `hasBeenMitigated = Yes`**, *not* by a remediation-date column. Tenable.sc does **not** export a literal "remediated on" timestamp, so the dashboard never invents one.
- **Segment = `repository`.** The dashboard reads the distinct `repository` values from your data and renders per-segment views automatically — any number of segments works, no configuration.
- **Host = `dnsName` (or `ip` when DNS is blank).**
- The mitigated export is typically a **recent window** (e.g. ~30 days of remediations). This bounds what MTTR and remediation metrics can mean — see [methodology.md](methodology.md).

## Sample data

`sample-data/` ships a fully synthetic fixture — 18,000 hosts across 5 segments (`bu1`–`bu5`), one IP per host (`10.10–10.50.x.x`). It is **not** based on any real organization. Provided as `cumulative.csv` + `mitigated.csv` (fast load) and as a combined `.xlsx`.
