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

---

## Up Next — Recording Order

Day numbers are fixed by `course_outline.md`. Record in order — dependencies are already baked into the numbering. Where a phase has no internal ordering constraint, that's noted below.

Phase 3 — Networking is now fully recorded (Days 9–15). Next up:

### Phase 4 — Serverless & Databases

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| **Day 16 (next)** | Azure Functions & Serverless | `functions.md` | Day 6 ✅ (App Service recommended); slotted here so Compute wraps up after Networking |
| Day 17 | Azure SQL Database + Other Databases | `sql_database.md` | None |

### Phase 5 — Identity, Security & Monitoring

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 18 | Microsoft Entra ID | `entra_id.md` | None |
| Day 19 | Azure RBAC | `rbac.md` | Entra ID |
| Day 20 | Azure Key Vault | `key_vault.md` | Entra ID |
| Day 21 | Azure Monitor & Alerts | `azure_monitor.md` | None |

> Day 19 and Day 20 only depend on Day 18, not on each other — order between them is flexible.

### Phase 6 — Azure DevOps

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 22 | Azure DevOps Introduction | `devops_intro.md` | None |
| Day 23 | Azure Repos | `azure_repos.md` | DevOps Intro |
| Day 24 | Pipelines: CI | `pipelines_ci.md` | Azure Repos |
| Day 25 | Pipelines: CD | `pipelines_cd.md` | Pipelines CI |
| Day 26 | Azure Artifacts | `artifacts.md` | DevOps Intro |

### Phase 7 — Infrastructure as Code

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 27 | ARM Templates & Bicep | `arm_bicep.md` | Phases 1–4 recommended (so demos have services to deploy) |
| Day 28 | Terraform on Azure | `terraform.md` | ARM & Bicep |

### Phase 8 — Containers & AKS

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 29 | ACR & Docker | `acr_docker.md` | None |
| Day 30 | AKS Setup | `aks_setup.md` | ACR & Docker |
| Day 31 | AKS Advanced | `aks_advanced.md` | AKS Setup |

### Phase 9 — Capstone Project

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 32 | Capstone Part 1 | `capstone_part1.md` | Phases 1–6 |
| Day 33 | Capstone Part 2 | `capstone_part2.md` | Capstone Part 1 |

### Optional Bonus

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 34 | Cosmos DB (Optional) | `cosmos_db_optional.md` | Phase 2 (Storage) |

---

## What's Next to Record

**Day 16 — Azure Functions & Serverless** (`days/functions.md`) — next up now that Phase 3 (Networking) is fully recorded.
