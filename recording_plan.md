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

---

## Up Next — Recording Order

Day numbers are fixed by `course_outline.md`. Record in order — dependencies are already baked into the numbering. Where a phase has no internal ordering constraint, that's noted below.

### Phase 3 — Networking (continued)

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| **Day 12 (next)** | Azure DNS — Public & Private Zones | `azure_dns.md` | Days 10–11 ✅ |
| Day 13 | Load Balancer & VM Scale Sets | `load_balancer.md` | Days 10–11 ✅ |
| Day 14 | VPN Gateway & ExpressRoute | `vpn_expressroute.md` | Days 10–11 ✅ |
| Day 15 | Application Gateway & WAF | `application_gateway.md` | Days 10–11 ✅ |
| Day 16 | Traffic Manager, Front Door & CDN | `traffic_frontdoor_cdn.md` | None |

> Days 13–16 have no ordering constraint relative to each other — only Day 12 (DNS) needs to come first since Day 11's "What's Next" promises it.

### Phase 4 — Serverless & Databases

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 17 | Azure Functions & Serverless | `functions.md` | Day 6 ✅ (App Service recommended); slotted here so Compute wraps up after Networking |
| Day 18 | Azure SQL Database + Other Databases | `sql_database.md` | None |

### Phase 5 — Identity, Security & Monitoring

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 19 | Microsoft Entra ID | `entra_id.md` | None |
| Day 20 | Azure RBAC | `rbac.md` | Entra ID |
| Day 21 | Azure Key Vault | `key_vault.md` | Entra ID |
| Day 22 | Azure Monitor & Alerts | `azure_monitor.md` | None |

> Day 20 and Day 21 only depend on Day 19, not on each other — order between them is flexible.

### Phase 6 — Azure DevOps

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 23 | Azure DevOps Introduction | `devops_intro.md` | None |
| Day 24 | Azure Repos | `azure_repos.md` | DevOps Intro |
| Day 25 | Pipelines: CI | `pipelines_ci.md` | Azure Repos |
| Day 26 | Pipelines: CD | `pipelines_cd.md` | Pipelines CI |
| Day 27 | Azure Artifacts | `artifacts.md` | DevOps Intro |

### Phase 7 — Infrastructure as Code

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 28 | ARM Templates & Bicep | `arm_bicep.md` | Phases 1–4 recommended (so demos have services to deploy) |
| Day 29 | Terraform on Azure | `terraform.md` | ARM & Bicep |

### Phase 8 — Containers & AKS

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 30 | ACR & Docker | `acr_docker.md` | None |
| Day 31 | AKS Setup | `aks_setup.md` | ACR & Docker |
| Day 32 | AKS Advanced | `aks_advanced.md` | AKS Setup |

### Phase 9 — Capstone Project

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 33 | Capstone Part 1 | `capstone_part1.md` | Phases 1–6 |
| Day 34 | Capstone Part 2 | `capstone_part2.md` | Capstone Part 1 |

### Optional Bonus

| Day | Topic | File | Depends On |
|-----|-------|------|------------|
| Day 35 | Cosmos DB (Optional) | `cosmos_db_optional.md` | Phase 2 (Storage) |

---

## What's Next to Record

**Day 12 — Azure DNS: Public & Private Zones** (`days/azure_dns.md`) — completes the VNet/DNS arc started in Days 9–11, as promised in Day 11's "What's Next" section.
