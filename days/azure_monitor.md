# Day 19 — Azure Monitor, Log Analytics & Alerts

> Phase 5 — Identity, Security + Monitoring
>
> **No scope conflict.** Unlike Days 17 and 18, this file and `course_outline.md` already agreed.
> The outline lists Log Analytics, Metrics, Alerts, Action Groups, Application Insights and
> Workbooks; this file added Diagnostic Settings and Dashboards, which fit inside the same day.
> Everything below is kept, plus the agent/DCR story the original file was missing entirely.

## Scope Rules for This Day

Same standard as Days 17 and 18 — student-friendly depth, not a reference manual:

- **14 parts.** Roughly 12,000 words. Around 90–100 minutes of video.
- **This is the first day where a careless lab genuinely costs money**, so cost is taught explicitly
  and early rather than waved at. One deliberate ~$0.50/month step, clearly flagged and deleted.
- **Interview and real-job relevance is the filter.**
- **Cut deliberately:** Azure Monitor managed Prometheus and managed Grafana beyond a mention,
  dedicated clusters, customer-managed keys for workspaces, the full KQL language, Sentinel (owned
  by Day 18 Part 14), VM Insights and Container Insights beyond naming them, multi-step web tests,
  the common alert schema, and workbook authoring beyond one parameter.

## ⚠️ Correction Carried Into the Script

The previous version of this file said:

> *"First 5GB of data ingested per day is free"*

**That is wrong.** The free grant is **5 GB per billing account per month** — a ~30x difference. A
student who believes the old figure could enable verbose diagnostics across several resources,
ingest ~4 GB/day assuming it's covered, and receive an invoice around **$270** for the month.

The script calls this out explicitly in a `!!! danger` box in *Before We Begin*, names it as one of
the most common surprise bills in Azure, and repeats it in the gotchas and the interview table. It
also corrects the related claim that Application Insights has its own 5 GB/month free tier — App
Insights is **workspace-based** and bills as Log Analytics ingestion, so it draws on the same grant.

## Portal Currency — Verified September 2026

Researched against current Microsoft documentation before writing. What shaped the script:

| Finding | Detail | Where it lands |
|---|---|---|
| **Free grant is monthly** | 5 GB **per billing account per month** for Analytics-plan ingestion, ~$2.30/GB after. Analytics queries are free. | *Before We Begin*, Parts 7 and 14 |
| **What is genuinely free** | Docs are explicit: platform metric collection/analysis **and activity log collection *and alerting*** incur no charge. Metric alerts include **10 free time series**. | Part 9 — shapes which demos are free |
| **Log alerts cost** | ~$0.50/month first time series, $0.05 each additional; splitting by dimensions multiplies it. | Part 11 — the one flagged paid step |
| **Data Collector API retires 14 Sept 2026** | Days after this script was written. Replacement is the DCR-based **Logs Ingestion API** (Entra auth, not shared keys). | Part 8 |
| **Agent landscape** | **MMA retired 31 Aug 2024**; Azure Diagnostics extension **deprecated 31 Mar 2026**; Dependency Agent retires 30 June 2028. **AMA + DCR is the only current answer.** | Part 8 |
| **App Insights** | Classic **retired 29 Feb 2024** — workspace-based only. **Instrumentation key ingestion support ended 31 Mar 2025** — connection strings now. | Part 12 |
| **Table plans** | Analytics / Basic / Auxiliary, with Basic and Auxiliary **charged per GB scanned to query**. Retention split into **interactive** (31 days free) and **total/long-term** (12 years), configurable **per table**. Default Analytics retention 30 days. | Part 7 |

## Goal

Know what your resources are doing, and be told when they stop doing it — without either missing the
incident or receiving a surprise bill for the privilege of watching.

---

## Part One — Collect and Query (Parts 1–7)

**1. What Azure Monitor Actually Is**
Four data types — metrics, logs, traces, changes — in one table with cost and retention. **The free
column is bigger than beginners expect; the bill is one column: logs.** ASCII diagram of
sources → diagnostic settings → stores → consumers, which the rest of the day walks through.

**2. Metrics: The Data You Already Have**
Free, 93 days, near real-time, already collected on everything. Aggregation / granularity /
dimensions — the three things the alert wizard will ask about. **Average hides spikes; Max finds them.**
*Demo: create the Free F1 App Service we monitor all day, generate traffic, then work Metric Explorer
properly — splitting, filters, multiple metrics, pin, and the "New alert rule" shortcut.*

**3. The Activity Log: Who Did That?**
Free, 90 days, control plane only — the third outing for the control/data-plane split. The seven
categories. **The `Caller` field in the JSON tab is the answer to "who did that?"**
*Demo: filter to Delete operations; read the JSON.*

**4. The Log Analytics Workspace**
What it is, and that Sentinel, Defender, App Insights and the Insights experiences all sit on it.
**How many should you have** — start with one; split for sovereignty, access isolation or retention;
the anti-pattern is one-per-application. Resource-context RBAC as the middle path.
*Demo: create `law-lwm-day19`; tour the left menu as a map of the day.*

**5. Diagnostic Settings: The Hinge**
**The most transferable idea in the day** — one mechanism, identical for every Azure service, four
destinations. **The Off-By-Default Rule:** resource logs are never retroactive, so this belongs in
the build checklist, enforced at scale with **Azure Policy `DeployIfNotExists`**.
*Demo: route the subscription activity log to the workspace, then App Service HTTP + console logs.
Includes why NOT to tick `AllMetrics` by habit.*

**6. KQL**
Built up one operator at a time — `take`, `where`, `project`, `summarize`, `bin`, `render`, `extend`,
`let`, and `union`/`join` named. Six runnable queries against the student's own data.
*Demo: query editor, table discovery, the prebuilt Queries library. **Filter early; `take 10` first.***

**7. Table Plans, Retention and Not Getting a Shocking Bill**
**No demo — the most financially valuable ten minutes.** Analytics/Basic/Auxiliary trade-off and the
trap of moving a frequently-queried table to Basic. Interactive vs total retention. **The four ways
people overspend.** Daily cap, commitment tiers, and the `Usage` query to run on any inherited workspace.

---

## Part Two — Detect and Respond (Parts 8–14)

**8. Getting Data Off a Machine: Agents and DCRs**
The retirement table. **Why DCRs are better than the old workspace-level config** — many-to-many,
deployable, governable. **The 14 Sept 2026 Data Collector API retirement.** Transformations as
cost control at ingestion.
*Demo: walk the DCR wizard and cancel — no VM, so the lab stays free.*

**9. Alerts: The Three Types, and What They Cost**
Comparison table with **cost per type** as a first-class column. Severity as a routing decision and
**alert fatigue as a self-inflicted outage**. Stateful vs stateless. Alert processing rules for
maintenance windows.

**10. Action Groups**
Notifications vs actions. **Email Azure Resource Manager Role** as Day 17's RBAC doing useful work.
Actions as the on-ramp to auto-remediation.
*Demo: create `ag-lwm-oncall`; the confirmation email doubles as a free test.*

**11. Building Alerts That Actually Fire**
*Demo 1 — metric alert on Http 5xx (**free**, within the 10-series allowance), then stop the app to
make it fire and receive a real email.*
*Demo 2 — activity log alert on Delete Resource Group (**completely free**).*
*Demo 3 — log search alert (**~$0.50/month, flagged and deleted at cleanup**), built query-first.*

**12. Application Insights**
The different class of question — not *is it up* but *why is it slow*. **Dependency tracking is
usually where the answer is.** Classic retirement and connection strings.
*Demo: auto-instrument the App Service into the existing workspace; Live Metrics, Failures,
Performance, Application Map; then query `requests` in the same workspace as the infra logs.*

**13. Visualising It**
Dashboard vs Workbook vs Insights vs Managed Grafana. **A workbook can encode an investigation, not
just a picture.**
*Demo: build a small workbook with a text block, a query and a time-range parameter; contrast with
pinning a metric tile to a dashboard.*

**14. Cost Control, Gotchas and Interview Prep**
*Demo: Usage and estimated costs, the `Usage` billing query, set a daily cap.* The gotcha list and a
40-row interview/exam keyword table.

**Cleanup** — including **deleting the subscription-level diagnostic setting**, which the resource
group delete does not remove. Same shape of trap as Day 18's soft-deleted vault, called out as such.

---

## Hands-On Demo Summary

**Account requirements:** everything is free except one deliberately-flagged step. Platform metrics,
the activity log, activity log alerts, and Analytics-plan queries are free outright. Metric alerts
are within the 10-free-time-series allowance. Ingestion is a few MB against a 5 GB/month grant.

- ✅ Create the Free F1 App Service and generate traffic
- ✅ Metric Explorer — splitting, filters, multiple metrics, pin to dashboard
- ✅ Activity log — filter to deletes, read `Caller` in the JSON
- ✅ Create `law-lwm-day19`
- ✅ Diagnostic setting: subscription activity log → workspace
- ✅ Diagnostic setting: App Service HTTP + console logs → workspace
- ✅ Six KQL queries against the student's own data
- ✅ Create action group `ag-lwm-oncall` (confirmation email is a free test)
- ✅ **Metric alert** on Http 5xx → stop the app → receive a real email
- ✅ **Activity log alert** on Delete Resource Group — free
- ⚠️ **Log search alert** — ~$0.50/month, built to learn the mechanics, **deleted at cleanup**
- ✅ Enable Application Insights into the existing workspace; Live Metrics, Failures, App Map
- ✅ Query `requests` beside `AppServiceHTTPLogs` — app and infra in one workspace
- ✅ Build a workbook with a time-range parameter
- ✅ `Usage` billing query; set a daily cap
- ✅ Cleanup — **including the subscription-level diagnostic setting**

**Explained but not demonstrated (paid):** SMS/voice notifications, availability tests, Managed
Grafana, commitment tiers, Basic/Auxiliary table plans, VM Insights and Container Insights.

## Cleanup

Delete the three alert rules (the log alert first — it's the only recurring charge), **delete the
subscription-level diagnostic setting `activity-to-law`**, then delete `rg-day19-demo`. **Keep**
`Priya Sharma`, `grp-finance-team` and `db-lwm-demo` for Day 30.

## Summary

Metrics and the activity log were free and already running the whole time. Logs are the part that
costs money, routed by **diagnostic settings** — one mechanism for every Azure service — into a
**Log Analytics workspace** billed per GB, with **5 GB free per billing account per month**. Because
logs are off by default and never retroactive, enabling them belongs in the build checklist and is
enforced with Azure Policy. **KQL** is the skill that transfers furthest. Then alerts: metric alerts
fast and effectively free, activity log alerts free outright, log alerts billed per rule — all
attached to action groups named for who they reach.

This closes Phase 5. Across three days: **who someone is** (Entra ID), **what they may do** (RBAC),
**where the secrets live** (Key Vault), and **how to know what's happening** (Azure Monitor) — the
layer that separates "I can deploy this" from "I can run this."
