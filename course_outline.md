# LearnWithMithran — Azure Cloud & Azure DevOps Course Outline
> Master reference file for Claude Code sessions. Read this file at the start of every session.

---

## Channel & Course Context

- **Channel:** LearnWithMithran (YouTube)
- **Course:** Azure Cloud + Azure DevOps — A Complete Beginner's Guide
- **Total Videos:** 29 (Day 1 through Day 29) + 1 Optional Bonus (Day 30 — Cosmos DB)
- **Total Phases:** 9 (Phase 0 through Phase 8) + Optional Bonus
- **Max Video Length:** 2 hours per video
- **Target Audience:** Complete beginners to cloud — no prior AWS, GCP, or on-premise experience assumed. Also suitable for viewers with some prior cloud background.
- **Instructor Account:** Paid Azure account (used during teaching to demonstrate all features including paid-only ones)
- **Student Account:** Azure Free Tier (assumed for most viewers; some may choose paid for practice)

---

## Teaching Philosophy & Ground Rules

1. **Azure-Native Teaching** — Explain every concept from first principles as if the viewer has never used any cloud platform. No comparisons to AWS, GCP, or on-premise.
2. **Console-First Always** — Every concept is demonstrated via the Azure Portal (GUI). Students learn by clicking, not typing commands.
3. **CLI is Contextual Only** — Azure CLI is shown only where it genuinely saves time (e.g., Bicep/Terraform deploys). CLI usage is capped at 3–5% of total content. No standalone CLI-only video.
4. **Hands-On Every Day** — Every video includes at least one practical demo or lab the viewer can follow along.
5. **No Assumed Cloud Knowledge** — Treat every viewer as starting from zero. Define all terms clearly.
6. **Account Tier Transparency** — The instructor teaches on a paid Azure account. Students are assumed to be on the Azure Free Tier. Each day's Hands-On Demo must clearly label every step:
   - ✅ **Free Tier** — students can follow along
   - 💳 **Paid (Instructor Demo)** — instructor demonstrates; students should watch, skip, or activate a free trial to practice
7. **Script Structure Per Video:**
   - Hook (what problem does this solve?) — ~2 min
   - Core concept explanation — ~10–15 min per section
   - Portal demo / hands-on — immediately after each section; **no time cap**; cover every important feature completely
   - Summary + what's next — ~2–3 min
8. **Deep Dive, Not High Level** — This is a technical hands-on course. Cover all key features, configuration options, and real-world use cases for each service. Do not skip steps or features for brevity.

---

## Phase Overview

| Phase | Days | Topic |
|-------|------|-------|
| Phase 0 | Day 1 – Day 2 | Foundations + Account Setup |
| Phase 1 | Day 3 – Day 6 | Compute |
| Phase 2 | Day 7 – Day 9 | Networking |
| Phase 3 | Day 10 – Day 13 | Storage, Databases & Global Delivery |
| Phase 4 | Day 14 – Day 17 | Identity, Security + Monitoring |
| Phase 5 | Day 18 – Day 22 | Azure DevOps |
| Phase 6 | Day 23 – Day 24 | Infrastructure as Code (IaC) |
| Phase 7 | Day 25 – Day 27 | Containers + AKS |
| Phase 8 | Day 28 – Day 29 | Capstone Project |
| Optional | Day 30 | Cosmos DB (Bonus — Optional) |

---

## Full Day-by-Day Breakdown

### Phase 0 — Foundations + Account Setup

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 1 | Account Setup & Azure Free Tier | Azure Portal, Subscriptions, Free Tier | ✅ Free |
| Day 2 | Core Concepts & Portal Deep Dive | ARM, Resource Groups, Cloud Shell, Pricing Calculator, SLAs, Advisor, Service Health, Locks, Dashboards | ✅ Free |

### Phase 1 — Compute

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 3 | Virtual Machines Part 1 — Linux & Windows VMs | Azure VMs, NSG, Managed Disks, Public IP, SSH, Windows VM + RDP | ✅ Free |
| Day 4 | Virtual Machines Part 2 — Management, VMSS & Backups | Availability Sets/Zones, **VM Scale Sets**, Snapshots, Custom Images, Bastion, **Azure Backup**, Recovery Services Vault | ✅ / 💳 |
| Day 5 | Azure App Service & Web Apps | App Service Plans, Web Apps, Deployment slots, Custom domains, Auto-scaling | ✅ / 💳 |
| Day 6 | Azure Functions & Serverless | Function App, Triggers, Bindings, Consumption plan, Durable Functions | ✅ |

### Phase 2 — Networking

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 7 | Virtual Network + Azure DNS | VNet, Subnets, NSG, VNet Peering, Service Endpoints, Private Endpoints, **Azure DNS Public & Private Zones** | ✅ |
| Day 8 | Load Balancer & VM Scale Sets | **Azure Load Balancer** (Standard), VMSS, Health Probes, Autoscale Rules | 💳 |
| Day 9 | VPN Gateway & ExpressRoute | VPN Gateway, Site-to-Site VPN, Point-to-Site VPN, ExpressRoute | 💳 |

### Phase 3 — Storage, Databases & Global Delivery

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 10 | Azure Storage Account | Blob Storage, File Storage, Queue Storage, Table Storage, Access Tiers, SAS tokens, Storage Explorer | ✅ |
| Day 11 | Azure SQL Database + Other Databases | Azure SQL (Serverless), Elastic Pools, Managed Instance, **PostgreSQL & MySQL Flexible Server (overview)** | ✅ |
| Day 12 | Application Gateway & WAF | **Application Gateway v2**, Path-based routing, Multi-site hosting, SSL termination, **WAF**, OWASP CRS | 💳 |
| Day 13 | Traffic Manager, Front Door & CDN | **Azure Traffic Manager**, **Azure Front Door**, **Azure CDN**, global routing methods, edge caching | 💳 |

### Phase 4 — Identity, Security + Monitoring

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 14 | Microsoft Entra ID | Users, Groups, MFA, Conditional Access, App Registrations, Managed Identities | ✅ |
| Day 15 | Azure RBAC | Roles, Assignments, Custom Roles, Privileged Identity Management (PIM) | ✅ |
| Day 16 | Azure Key Vault | Secrets, Keys, Certificates, Access Policies, RBAC, Soft Delete, Private Endpoints | ✅ |
| Day 17 | Azure Monitor & Alerts | Log Analytics, Metrics, Alerts, Action Groups, Application Insights, Workbooks | ✅ |

### Phase 5 — Azure DevOps

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 18 | Azure DevOps Introduction | Organizations, Projects, Boards, Agile workflow | ✅ |
| Day 19 | Azure Repos | Git repos, Branches, Pull Requests, Branch policies | ✅ |
| Day 20 | Azure Pipelines — CI | Build pipelines, YAML, triggers, agents, artifacts | ✅ |
| Day 21 | Azure Pipelines — CD | Release pipelines, deployment gates, environments, approvals | ✅ |
| Day 22 | Azure Artifacts | Feed creation, npm/NuGet packages, upstream sources | ✅ |

### Phase 6 — Infrastructure as Code

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 23 | ARM Templates & Bicep | ARM JSON templates, Bicep language, parameters, modules, what-if deployments | ✅ |
| Day 24 | Terraform on Azure | Terraform CLI, AzureRM provider, state management, plan/apply/destroy | ✅ |

### Phase 7 — Containers + AKS

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 25 | Azure Container Registry & Docker | ACR, Docker images, push/pull, tasks, geo-replication | ✅ / 💳 |
| Day 26 | AKS Setup | Azure Kubernetes Service, node pools, kubectl, deployments, services | 💳 |
| Day 27 | AKS Advanced | Ingress, HPA, cluster autoscaler, Azure Monitor for containers, RBAC | 💳 |

### Phase 8 — Capstone Project

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 28 | Capstone Part 1 — Architecture & Build | Multi-tier app: VNet + VM/App Service + SQL Database + Key Vault + Blob Storage | 💳 |
| Day 29 | Capstone Part 2 — DevOps + IaC | CI/CD pipeline, Bicep/Terraform deployment, monitoring, alerts, cleanup | 💳 |

### Optional Bonus

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 30 | Cosmos DB — NoSQL at Global Scale (Optional) | Cosmos DB Core (SQL) API, Free Tier, global replication, RUs, partitioning | ✅ |

---

## Topic Coverage Checklist

| Topic | Covered In |
|-------|-----------|
| App Service / Web Apps | Day 5 |
| Storage Account | Day 10 |
| Microsoft Entra ID | Day 14 |
| Virtual Network (VNet) | Day 7 |
| Azure Monitor | Day 17 |
| Application Gateway + WAF | Day 12 |
| Load Balancer | Day 8 |
| VM Scale Sets (VMSS) | Day 4 + Day 8 |
| Traffic Manager | Day 13 |
| Azure Front Door | Day 13 |
| Azure SQL Database | Day 11 |
| Azure CDN | Day 13 |
| Key Vault | Day 16 |
| Backup Service (Azure Backup) | Day 4 |
| Function App | Day 6 |
| ARM Templates | Day 23 |
| Azure DNS | Day 7 |
| VPN Gateway / ExpressRoute | Day 9 |
| Azure Kubernetes Service | Day 26–27 |
| Containers / ACR | Day 25 |
| Azure DevOps | Day 18–22 |
| Bicep / Terraform | Day 23–24 |

---

## Instructions for Claude Code Sessions

When starting a new session to write course content:

1. **Read this file first** to load course context.
2. **Read the specific day file** from `days/` for the day you are working on (e.g., `days/day06_functions.md`).
3. **Output format for each day's script:**
   - File name: `dayXX_topic_name.md` saved to `docs/`
   - Sections: Hook → Core Concepts (with Hands-On after each section) → Summary → Key Takeaways
   - All Portal Demo steps must carry ✅ or 💳 tier labels
4. **No AWS, GCP, or on-premise comparisons** — Azure-native explanations only.
5. **Scope per session:** Write ONE day at a time.
6. **Tone:** Conversational, enthusiastic, instructor-speaking-to-camera. Not dry documentation.

---

*Last updated: May 2026 | LearnWithMithran | Azure Cloud + Azure DevOps Course*
