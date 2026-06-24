# Roadmap

Planned and considered features. Everything here keeps the project's hard constraint: **100% client-side, data never leaves the browser.**

## Near-term

- **Real-export column mapping** — tolerate Tenable.sc column-name and format variations across versions / custom `/analysis` views, with a mapping step when headers don't match expectations.
- **True asset-count import** — let the user supply an authoritative host inventory so per-asset metrics (the 0.25 SLA) use a real denominator instead of distinct IPs seen in scan data.
- **True remediation-time MTTR** — falls out of compare mode (below): a finding open in the prior export and gone in the current one was remediated between scans. Compute per-severity. The single-upload page deliberately shows only **open age** instead (see [methodology.md](methodology.md#open-age--and-why-theres-no-mttr-yet)).
- **KEV / EPSS enrichment** — flag CISA KEV and show EPSS for prioritization. Would require either a bundled snapshot (to stay offline) or an optional, clearly-opt-in lookup.

## Month-over-month diff (compare mode)

Upload this month's and last month's exports and switch between periods to show **deltas**:

- New vs remediated vs persisting findings.
- Per-segment / per-severity change.
- Trend arrows on KPIs (▲/▼ vs prior period), net change in past-SLA counts.

Likely a "compare mode" toggle with current + prior snapshot slots, reusing the existing cumulative/mitigated model.

## Agent-coverage report

Reconcile vulnerability data against an authoritative inventory to find unmanaged hosts:

- **Active Directory** as source of truth (`Get-ADComputer -Filter * -Properties * | ConvertTo-Json`).
- **ManageEngine Endpoint Central** managed-endpoint export.
- Optional **CrowdStrike** host CSV.
- Output: per-agent coverage % vs AD inventory + gap lists (machines missing each agent), sliceable by segment / OS.

## Companion API fetcher (out of the page)

A pure static browser page **cannot** pull from the Tenable.sc / Tenable.io APIs directly — CORS blocks a `github.io` origin from calling on-prem SC or `cloud.tenable.com`, self-signed SC certs block `fetch`, and embedding API keys in a web page is a credential-exposure problem.

The privacy-preserving bridge is a thin **companion fetcher** (Python `pyTenable` or PowerShell using `X-ApiKeys`) that pulls the cumulative + patched `/analysis` exports and writes the two CSVs this page consumes. A browser extension (which can bypass CORS) is an alternative if in-page fetch is desired.
