# Recording Plan — LearnWithMithran Azure Course

> Pick your next topic from this file. Day numbers match `course_outline.md`. When you record a topic: write the script to `docs/dayXX_topic.md` and add it to `mkdocs.yml`'s nav.

---

## Completed ✅

| Day | Topic | File |
|-----|-------|------|
| Day 1 | Account Setup & Free Tier | `docs/day01_account_setup.md` |
| Day 2 | Core Concepts & Portal | `docs/day02_fundamentals.md` |
| Day 3 | Virtual Machines Part 1 | `docs/day03_vms_part1.md` |
| Day 4 | Web Servers on Azure VMs: Architecture + IIS & Nginx | `docs/day04_web_hosting_architecture.md` |
| Day 5 | VM Management, Availability, Bastion & Backup | `docs/day05_vms_part3.md` |
| Day 6 | Azure App Service | `docs/day06_app_service.md` |
| Day 7 | Azure Storage Account: Blob Storage, Static Websites & Versioning | `docs/day07_storage_account.md` |
| Day 8 | Azure Storage: Files, Queues, Tables & Storage Explorer | `docs/day08_storage_files_queue_tables.md` |
| Day 9 | Introduction to Networking: IP Addresses, Binary, CIDR & Subnet Classes | `docs/day09_intro_to_networking.md` |
| Day 10 | Azure Virtual Network: Address Spaces, Subnets & NSGs | `docs/day10_virtual_network.md` |
| Day 11 | VNet Advanced: Peering, Service & Private Endpoints, and Bastion | `docs/day11_vnet_advanced.md` |
| Day 12 | Load Balancer, VM Scale Sets & Application Gateway | `docs/day12_load_balancer_vmss.md` |
| Day 13 | Azure DNS — Public & Private Zones | `docs/day13_azure_dns.md` |
| Day 14 | Traffic Manager, Front Door, CDN & WAF | `docs/day14_traffic_frontdoor_cdn_waf.md` |
| Day 15 | VPN Gateway & ExpressRoute | `docs/day15_vpn_expressroute.md` |
| Day 16 | Azure SQL Database + Other Databases | `docs/day16_azure_sql_database.md` |
| Day 17 | Microsoft Entra ID & Azure RBAC (merged) | `docs/day17_entra_id_rbac.md` |
| Day 18 | Azure Key Vault | `docs/day18_key_vault.md` |
| Day 19 | Azure Monitor, Log Analytics & Alerts | `docs/day19_azure_monitor.md` |

---

## ⚠️ Structure Change — Entra ID + RBAC Merged

Entra ID and Azure RBAC were previously Day 17 and Day 18. **They are now a single video: Day 17 — Microsoft Entra ID & Azure RBAC** (`days/entra_id_rbac.md`).

Everything after it shifted down by one. The course is now **31 days + 3 optional bonus days**.

| Old | New | Topic |
|-----|-----|-------|
| Day 17 + Day 18 | **Day 17** | Microsoft Entra ID & Azure RBAC (merged) |
| Day 19 | Day 18 | Azure Key Vault |
| Day 20 | Day 19 | Azure Monitor & Alerts |
| Day 21–25 | Day 20–24 | Azure DevOps |
| Day 26–27 | Day 25–26 | IaC (Bicep, Terraform) |
| Day 28–30 | Day 27–29 | Containers + AKS |
| Day 31–32 | Day 30–31 | Capstone |
| Day 33–35 | Day 32–34 | Optional Bonus |

**This change is now complete.** `docs/day17_entra_id_rbac.md` holds the combined script (**14 parts**, identity in Parts 1–7 then authorisation in Parts 8–14, with a marked halfway point before Part 8). The identity-only `docs/day17_entra_id.md` has been removed, `days/entra_id.md` and `days/rbac.md` have been merged into `days/entra_id_rbac.md`, and the `mkdocs.yml` nav points at the merged title. Days 1–16 are unaffected.

---

## Up Next — Recording Order

Day numbers are fixed by `course_outline.md`. Record in order — dependencies are already baked into the numbering. Where a phase has no internal ordering constraint, that's noted below.

Phases 3 and 4 are fully recorded (Days 9–16), and Day 17 (Entra ID + RBAC) is written. Azure Functions and API Management are **not** in the numbered sequence — both live in the Optional Bonus block at the end. Next up:

### Phase 5 — Identity, Security & Monitoring

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| ~~Day 17~~ | ~~Microsoft Entra ID & Azure RBAC~~ — **written** | `entra_id_rbac.md` | — |
| ~~Day 18~~ | ~~Azure Key Vault~~ — **written** | `key_vault.md` | Day 17 |
| ~~Day 19~~ | ~~Azure Monitor & Alerts~~ — **written** | `azure_monitor.md` | None |

**Phase 5 is complete.** Days 17–19 are written. Phase 6 (Azure DevOps) is next.

### Phase 6 — Azure DevOps

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 20 | Azure DevOps Introduction | `devops_intro.md` | None |
| Day 21 | Azure Repos | `azure_repos.md` | DevOps Intro |
| Day 22 | Pipelines: CI | `pipelines_ci.md` | Azure Repos |
| Day 23 | Pipelines: CD | `pipelines_cd.md` | Pipelines CI |
| Day 24 | Azure Artifacts | `artifacts.md` | DevOps Intro |

### Phase 7 — Infrastructure as Code

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 25 | ARM Templates & Bicep | `arm_bicep.md` | Phases 1–4 recommended (so demos have services to deploy) |
| Day 26 | Terraform on Azure | `terraform.md` | ARM & Bicep |

### Phase 8 — Containers & AKS

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 27 | ACR & Docker | `acr_docker.md` | None |
| Day 28 | AKS Setup | `aks_setup.md` | ACR & Docker |
| Day 29 | AKS Advanced | `aks_advanced.md` | AKS Setup |

### Phase 9 — Capstone Project

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 30 | Capstone Part 1 | `capstone_part1.md` | Phases 1–6 |
| Day 31 | Capstone Part 2 | `capstone_part2.md` | Capstone Part 1 |

### Optional Bonus

Record these after Day 31 — no main-course day depends on them.

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 32 | Cosmos DB (Optional) | `cosmos_db_optional.md` | Phase 2 (Storage) |
| Day 33 | Azure Functions & Serverless (Optional) | `functions.md` | Day 6 (App Service recommended) |
| Day 34 | Azure API Management (Optional) | `api_management.md` | Day 33 — the APIM demo fronts the Function App built there |

---

## What's Next to Record

**Day 20 — Azure DevOps Introduction** (`days/devops_intro.md`) — next up. **Phase 5 is complete** (Days 17–19 written), and Day 20 starts Phase 6.

This is the biggest tonal shift in the course: nineteen days of portal clicking, and Phase 6 is where that stops. Day 19's closing section already sets this up explicitly — *"you can't automate what you don't understand"* — so open on that rather than re-justifying it.

### What Phase 5 hands forward

Three things are now fully established and should be **used, not re-taught**, when Pipelines arrives on Days 22–23:

- **Service principal + federated credential.** Day 17 Part 5 registers an app, creates a secret, and explicitly shows the **Federated credentials** tab with the *GitHub Actions* / *Kubernetes* scenarios, saying "bookmark this for Day 22." Day 22 owes that payoff — a pipeline that authenticates to Azure with **no stored secret**.
- **Key Vault + variable groups.** Day 18 Part 10 builds managed identity → data-plane role → reference, and Part 10's closing tip names **Azure DevOps variable groups linked to a vault** as "the same shape, that's Day 22."
- **Azure Monitor.** Day 19's closing section promises both of the above by name. Deployments should emit telemetry somewhere the student already knows how to query.

Also reusable: `Priya Sharma`, `grp-finance-team` and `db-lwm-demo` are still alive after all three cleanups, kept deliberately for Day 30's capstone.

### Standard to hold

Days 17, 18 and 19 all landed at **14 parts / ~12k words** — roughly 90–100 minutes, inside the 2-hour cap, student-friendly depth, free-or-near-free labs, paid features as concepts only. Keep Phase 6 to the same shape.

**Research the portal steps before writing.** All three Phase 5 days were verified against current Microsoft documentation and all three turned up material changes — see the *Portal Currency* tables in `days/entra_id_rbac.md`, `days/key_vault.md` and `days/azure_monitor.md`. Azure DevOps is a fast-moving product with its own UI refresh cadence and its own pricing (free tier: 5 users, parallel job grants that changed), so this matters at least as much for Phase 6.

### Source-file drift found so far

Three `days/*.md` files disagreed with `course_outline.md` or with reality. The outline wins on scope, per CLAUDE.md; factual errors get corrected in both places:

| File | Problem | Resolution |
|---|---|---|
| `days/entra_id_rbac.md` | Was two separate days | Merged into one — Day 17 |
| `days/key_vault.md` | Titled "Key Vault & Security Center"; Defender + Sentinel had equal billing | Key Vault is Parts 1–13; Defender + Sentinel get one concept-level closing part |
| `days/azure_monitor.md` | **Factual error:** claimed the Log Analytics free grant is "5GB per day". It is **5 GB per billing account per month** (~30x). Also claimed App Insights has its own separate 5 GB/month free tier — it is workspace-based and draws on the same grant. | Corrected in the script (a `!!! danger` box, the gotchas list and the interview table) and in the source file |

### Drift check on Phase 6 — done, and it found a lab-breaker

Ran the check rather than leaving it as a note. `days/devops_intro.md` and `days/pipelines_ci.md` both carried a claim that would have blocked students for a working week:

| File | Problem | Resolution |
|---|---|---|
| `days/devops_intro.md` | Said the free tier includes *"1 Microsoft-hosted pipeline with 1,800 minutes/month"* as though it arrives with a new org. **It does not.** Microsoft docs: *"you must enable the Microsoft-hosted free tier."* A new org has **zero** hosted parallelism and fails with `No hosted parallelism has been purchased or granted`. Fix is linking billing; if the grant is withheld for anti-abuse reasons, the request form takes **~4–5 business days**. Also missed: **public projects are retired** (no new ones; existing convert to private in 2027), killing the old "go public for free minutes" workaround. | Corrected in the file with a ⚠️ section. **The billing-link step must be Part 1–2 of Day 20, not a Day 22 footnote** — students need days of lead time. |
| `days/pipelines_ci.md` | Same 1,800-minutes claim, stated as "all steps below are free tier" | Corrected with a `!!! danger` box; Day 22 must open by verifying Parallel jobs shows an available Microsoft-hosted job |
| `days/aks_setup.md` | Said the AKS control plane is managed *"at no charge"* with no tier qualification | Corrected: **free on the Free tier only (no SLA)**; Standard is **/usr/bin/bash.10/cluster/hour (~3/month)**, Premium /usr/bin/bash.60. The demo must select Free explicitly |

**Verified as correct, no change needed:** Cosmos DB free tier (1,000 RU/s + 25 GB, one per subscription, lifetime), Azure Functions Consumption (1M executions/month), Azure Artifacts (2 GB), unlimited private repos, 5 free Basic users.

**Keep doing this check.** Four of the last four source files inspected had either scope drift or a factual error, and two of those errors would have cost students money or a week of blocked progress.
