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

**This change is now complete.** `docs/day17_entra_id_rbac.md` holds the combined script (24 parts, identity then authorisation, with a marked halfway point before Part 12). The identity-only `docs/day17_entra_id.md` has been removed, `days/entra_id.md` and `days/rbac.md` have been merged into `days/entra_id_rbac.md`, and the `mkdocs.yml` nav points at the merged title. Days 1–16 are unaffected.

---

## Up Next — Recording Order

Day numbers are fixed by `course_outline.md`. Record in order — dependencies are already baked into the numbering. Where a phase has no internal ordering constraint, that's noted below.

Phases 3 and 4 are fully recorded (Days 9–16), and Day 17 (Entra ID + RBAC) is written. Azure Functions and API Management are **not** in the numbered sequence — both live in the Optional Bonus block at the end. Next up:

### Phase 5 — Identity, Security & Monitoring

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| ~~Day 17~~ | ~~Microsoft Entra ID & Azure RBAC~~ — **written** | `entra_id_rbac.md` | — |
| **Day 18 (next)** | Azure Key Vault | `key_vault.md` | Day 17 |
| Day 19 | Azure Monitor & Alerts | `azure_monitor.md` | None |

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

**Day 18 — Azure Key Vault** (`days/key_vault.md`) — next up now that Day 17 is written.

Day 17 deliberately hands Key Vault a loaded starting position, so lean on it rather than re-teaching it:

- `Priya Sharma` and `grp-finance-team` are left alive at Day 17 cleanup. Reuse them for vault access instead of creating new principals.
- `db-lwm-demo` is left alive for the connection-string secret demo, and again for Day 30 (Capstone).
- The **control plane vs data plane** split is already taught (Day 17 Part 16, with the storage-account demo). Key Vault is the second instance of the same pattern — *Key Vault Contributor* manages the vault, *Key Vault Secrets User* reads a secret. Call back to the blob demo explicitly; do not re-derive it.
- **Key Vault Data Access Administrator** and its built-in ABAC condition are named in Day 17 Part 19 as a forward reference. Day 18 owns the hands-on.
- **Managed identity → resource** is fully established (Day 17 Parts 8 and 17). Day 18 should go straight to wiring an identity to a vault, not re-explain what a managed identity is.

The one genuinely new axis for Day 18 is the **access policy vs RBAC** permission-model choice on the vault itself, plus soft delete, purge protection, and private endpoints.

Recording note: Day 17 runs long (~2.5 h) by design — it is the only over-length video in the course. Days 18 and 19 are back to normal length.
