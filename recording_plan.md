# Recording Plan — LearnWithMithran Azure Course

> Pick your next topic from this file. Topics within the same group can be recorded in any order. Topics marked with a dependency must come after their prerequisite.
>
> When you record a topic: move the file from `days_not_yet_taken/` → `days_taken/dayXX_topic.md`, add it to `docs/`, and update `mkdocs.yml`.

---

## Completed ✅

| Day | Topic | File |
|-----|-------|------|
| Day 1 | Account Setup & Free Tier | `days_taken/day01_account_setup.md` |
| Day 2 | Core Concepts & Portal | `days_taken/day02_fundamentals.md` |
| Day 3 | Virtual Machines Part 1 | `days_taken/day03_vms_part1.md` |
| Day 4 | Web Servers on Azure VMs: Architecture + IIS & Nginx | `days_taken/day04_web_hosting_architecture.md` |

---

## Remaining Topics — Suggested Recording Order

### Group 1 — Compute (finish the VM + serverless track)
> Record these before moving to Networking. No strict order within this group.

| Priority | Topic | File | Depends On |
|----------|-------|------|------------|
| Next | VM Management, VMSS & Backups | `vms_part3.md` | Day 3 ✅ |
| | Azure App Service | `app_service.md` | Day 3 ✅ |
| | Azure Functions & Serverless | `functions.md` | App Service (recommended) |

---

### Group 2 — Networking
> VNet must come before Load Balancer and VPN. App Gateway and Traffic Manager can come after.

| Priority | Topic | File | Depends On |
|----------|-------|------|------------|
| First | Virtual Network + DNS | `virtual_network.md` | Day 3 ✅ |
| After VNet | Load Balancer & VMSS | `load_balancer.md` | VNet |
| After VNet | VPN Gateway & ExpressRoute | `vpn_expressroute.md` | VNet |
| After VNet | Application Gateway & WAF | `application_gateway.md` | VNet |
| Anytime | Traffic Manager, Front Door & CDN | `traffic_frontdoor_cdn.md` | None |

---

### Group 3 — Storage & Databases
> All independent — record in any order.

| Priority | Topic | File | Depends On |
|----------|-------|------|------------|
| Anytime | Storage Accounts | `storage_account.md` | None |
| Anytime | Azure SQL + Other Databases | `sql_database.md` | None |

---

### Group 4 — Identity, Security & Monitoring
> Entra ID first, then RBAC and Key Vault build on it. Monitor is independent.

| Priority | Topic | File | Depends On |
|----------|-------|------|------------|
| First | Microsoft Entra ID | `entra_id.md` | None |
| After Entra | Azure RBAC | `rbac.md` | Entra ID |
| After Entra | Azure Key Vault | `key_vault.md` | Entra ID |
| Anytime | Azure Monitor & Alerts | `azure_monitor.md` | None |

---

### Group 5 — Azure DevOps
> Independent track — can be recorded in parallel with any other group. Must follow internal order.

| Priority | Topic | File | Depends On |
|----------|-------|------|------------|
| First | DevOps Introduction | `devops_intro.md` | None |
| 2nd | Azure Repos | `azure_repos.md` | DevOps Intro |
| 3rd | Pipelines: CI | `pipelines_ci.md` | Azure Repos |
| 4th | Pipelines: CD | `pipelines_cd.md` | Pipelines CI |
| Last | Azure Artifacts | `artifacts.md` | DevOps Intro |

---

### Group 6 — Infrastructure as Code
> Record after you've covered a few Azure services so demos make sense.

| Priority | Topic | File | Depends On |
|----------|-------|------|------------|
| First | ARM Templates & Bicep | `arm_bicep.md` | Groups 1–3 recommended |
| After ARM | Terraform on Azure | `terraform.md` | ARM & Bicep |

---

### Group 7 — Containers & AKS
> Docker/ACR first, then AKS builds on it.

| Priority | Topic | File | Depends On |
|----------|-------|------|------------|
| First | ACR & Docker | `acr_docker.md` | None |
| After ACR | AKS Setup | `aks_setup.md` | ACR & Docker |
| After AKS | AKS Advanced | `aks_advanced.md` | AKS Setup |

---

### Group 8 — Capstone Project
> Record last. Needs Compute, Networking, Storage, Identity, and DevOps done first.

| Priority | Topic | File | Depends On |
|----------|-------|------|------------|
| First | Capstone Part 1 | `capstone_part1.md` | Groups 1–5 |
| After Part 1 | Capstone Part 2 | `capstone_part2.md` | Capstone Part 1 |

---

### Optional Bonus

| Topic | File | Depends On |
|-------|------|------------|
| Cosmos DB (Optional) | `cosmos_db_optional.md` | Storage (Group 3) |

---

## Quick Reference — What Can I Record Next?

Any topic with **None** or a ✅ completed day in the Depends On column is ready to record right now:

- `vms_part3.md` — VM Management, VMSS & Backups
- `app_service.md` — Azure App Service
- `virtual_network.md` — Virtual Network + DNS
- `storage_account.md` — Storage Accounts
- `sql_database.md` — Azure SQL + Other Databases
- `entra_id.md` — Microsoft Entra ID
- `azure_monitor.md` — Azure Monitor & Alerts
- `devops_intro.md` — Azure DevOps Introduction
- `traffic_frontdoor_cdn.md` — Traffic Manager, Front Door & CDN
- `acr_docker.md` — ACR & Docker
- `functions.md` — Azure Functions (after App Service)
