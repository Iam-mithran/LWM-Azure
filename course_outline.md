# LearnWithMithran — Azure Cloud & Azure DevOps Course Outline
> Master reference file for Claude Code sessions. Read this file at the start of every session.

---

## Channel & Course Context

- **Channel:** LearnWithMithran (YouTube)
- **Course:** Azure Cloud + Azure DevOps — A Complete Beginner's Guide
- **Total Videos:** 32 (Day 1 through Day 32) + 3 Optional Bonus (Day 33 — Cosmos DB, Day 34 — Azure Functions, Day 35 — API Management)
- **Total Phases:** 10 (Phase 0 through Phase 9) + Optional Bonus
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
| Phase 1 | Day 3 – Day 6 | Compute (VMs & App Service) |
| Phase 2 | Day 7 – Day 8 | Storage |
| Phase 3 | Day 9 – Day 15 | Networking |
| Phase 4 | Day 16 | Databases |
| Phase 5 | Day 17 – Day 20 | Identity, Security + Monitoring |
| Phase 6 | Day 21 – Day 25 | Azure DevOps |
| Phase 7 | Day 26 – Day 27 | Infrastructure as Code (IaC) |
| Phase 8 | Day 28 – Day 30 | Containers + AKS |
| Phase 9 | Day 31 – Day 32 | Capstone Project |
| Optional | Day 33 – Day 35 | Bonus — Cosmos DB, Azure Functions & Serverless, API Management |

---

## Full Day-by-Day Breakdown

### Phase 0 — Foundations + Account Setup

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 1 | Account Setup & Azure Free Tier | Azure Portal, Subscriptions, Free Tier | ✅ Free |
| Day 2 | Core Concepts & Portal Deep Dive | ARM, Resource Groups, Cloud Shell, Pricing Calculator, SLAs, Advisor, Service Health, Locks, Dashboards | ✅ Free |

### Phase 1 — Compute (VMs & App Service)

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 3 | Virtual Machines Part 1 — Linux & Windows VMs | Azure VMs, VM Series & Sizing, Pricing Models, Managed Disks, Public IP & DNS Labels, NSG, SSH, Windows VM + RDP | ✅ Free |
| Day 4 | Web Servers on Azure VMs: Architecture + IIS & Nginx | Static vs Dynamic Sites, 1/2/3-Tier Architecture, Web Servers, IIS (Windows VM), Nginx (Ubuntu VM) | ✅ Free |
| Day 5 | VM Management, Availability, Bastion & Backup | Availability Sets/Zones, **VM Scale Sets** (intro), Disk Management & Data Disks, Snapshots, Custom Images, **Azure Backup**, Recovery Services Vault | ✅ / 💳 |
| Day 6 | Azure App Service & Web Apps | App Service Plans, Web Apps, Runtimes, ZIP Deploy, App Settings, Custom Domains & SSL, Deployment Slots, Auto-Scale | ✅ / 💳 |

### Phase 2 — Storage

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 7 | Azure Storage Account: Blob Storage, Static Websites & Versioning | Storage Accounts, Blob Storage, Access Tiers, Lifecycle Management, SAS Tokens, Blob Versioning, Static Website Hosting | ✅ |
| Day 8 | Azure Storage: Files, Queues, Tables & Storage Explorer | Azure Files, File Share Snapshots, Soft Delete, Queue Storage, Table Storage, Azure File Sync, Storage Explorer | ✅ |

### Phase 3 — Networking

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 9 | Introduction to Networking: IP Addresses, Binary, CIDR & Subnet Classes | IPv4 Addressing, Binary/Decimal Conversion, Public vs Private IP, NAT, CIDR Notation, IP Address Classes | ✅ |
| Day 10 | Azure Virtual Network: Address Spaces, Subnets & NSGs | VNet, Address Space (CIDR), Subnets, Azure-Reserved Addresses, Network Security Groups, Application Security Groups (ASGs) | ✅ |
| Day 11 | VNet Advanced: Peering, Service & Private Endpoints, and Bastion | VNet Peering, Service Endpoints, Private Endpoints, **Azure Bastion**, Route Tables (UDR), **NAT Gateway** | ✅ / 💳 |
| Day 12 | Load Balancer, VM Scale Sets & Application Gateway | **Azure Load Balancer** (Standard), Frontend IP, Backend Pools, Health Probes, Inbound NAT Rules, VMSS Autoscale, **Application Gateway v2**, Path-based routing, Multi-site hosting, SSL termination | 💳 |
| Day 13 | Azure DNS: Public & Private Zones | Azure DNS Public Zones, DNS Record Types, TTL, Alias Records, Azure DNS Private Zones, VNet Links & Autoregistration, Azure DNS Private Resolver, DNSSEC | ✅ |
| Day 14 | Traffic Manager, Front Door, CDN & WAF | **Azure Traffic Manager**, **Azure Front Door**, **Azure CDN**, global routing methods, edge caching, **Web Application Firewall (WAF)**, OWASP CRS | 💳 |
| Day 15 | VPN Gateway & ExpressRoute | VPN Gateway, Site-to-Site VPN, Point-to-Site VPN, ExpressRoute, Azure Virtual WAN | 💳 |

### Phase 4 — Databases

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 16 | Azure SQL Database + Other Databases | Azure SQL (Serverless), Elastic Pools, Managed Instance, **PostgreSQL & MySQL Flexible Server (overview)** | ✅ |

### Phase 5 — Identity, Security + Monitoring

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 17 | Microsoft Entra ID | Users, Groups, MFA, Conditional Access, App Registrations, Managed Identities | ✅ |
| Day 18 | Azure RBAC | Roles, Assignments, Custom Roles, Privileged Identity Management (PIM) | ✅ |
| Day 19 | Azure Key Vault | Secrets, Keys, Certificates, Access Policies, RBAC, Soft Delete, Private Endpoints | ✅ |
| Day 20 | Azure Monitor & Alerts | Log Analytics, Metrics, Alerts, Action Groups, Application Insights, Workbooks | ✅ |

### Phase 6 — Azure DevOps

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 21 | Azure DevOps Introduction | Organizations, Projects, Boards, Agile workflow | ✅ |
| Day 22 | Azure Repos | Git repos, Branches, Pull Requests, Branch policies | ✅ |
| Day 23 | Azure Pipelines — CI | Build pipelines, YAML, triggers, agents, artifacts | ✅ |
| Day 24 | Azure Pipelines — CD | Release pipelines, deployment gates, environments, approvals | ✅ |
| Day 25 | Azure Artifacts | Feed creation, npm/NuGet packages, upstream sources | ✅ |

### Phase 7 — Infrastructure as Code

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 26 | ARM Templates & Bicep | ARM JSON templates, Bicep language, parameters, modules, what-if deployments | ✅ |
| Day 27 | Terraform on Azure | Terraform CLI, AzureRM provider, state management, plan/apply/destroy | ✅ |

### Phase 8 — Containers + AKS

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 28 | Azure Container Registry & Docker | ACR, Docker images, push/pull, tasks, geo-replication | ✅ / 💳 |
| Day 29 | AKS Setup | Azure Kubernetes Service, node pools, kubectl, deployments, services | 💳 |
| Day 30 | AKS Advanced | Ingress, HPA, cluster autoscaler, Azure Monitor for containers, RBAC | 💳 |

### Phase 9 — Capstone Project

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 31 | Capstone Part 1 — Architecture & Build | Multi-tier app: VNet + VM/App Service + SQL Database + Key Vault + Blob Storage | 💳 |
| Day 32 | Capstone Part 2 — DevOps + IaC | CI/CD pipeline, Bicep/Terraform deployment, monitoring, alerts, cleanup | 💳 |

### Optional Bonus

These days sit outside the core 32-day curriculum. They are not required for AZ-900 or AZ-104, and none of the main-course days depend on them. Record and release them after Day 32.

| Day | Title | Key Services | Tier |
|-----|-------|-------------|------|
| Day 33 | Cosmos DB — NoSQL at Global Scale (Optional) | Cosmos DB Core (SQL) API, Free Tier, global replication, RUs, partitioning | ✅ |
| Day 34 | Azure Functions & Serverless (Optional) | Function App, Triggers, Bindings, Consumption plan, Durable Functions | ✅ |
| Day 35 | Azure API Management (Optional) | APIM, Products & Subscriptions, Subscription Keys, Policies (rate limiting, quotas, JWT validation, caching, transformation), Developer Portal, Versions & Revisions, Consumption vs Developer tier, Self-hosted Gateway | ✅ / 💳 |

> **Day 35 depends on Day 34** — the APIM demo fronts the Function App built in the Functions bonus day. Record them in that order.

---

## Topic Coverage Checklist

| Topic | Covered In |
|-------|-----------|
| Web Hosting Architecture (1/2/3-Tier, IIS/Nginx) | Day 4 |
| App Service / Web Apps | Day 6 |
| Storage Account (Blob, Files, Queues, Tables) | Day 7–8 |
| Networking Fundamentals (IP, Binary, CIDR) | Day 9 |
| Virtual Network (VNet) | Day 10–11 |
| Application Security Groups (ASGs) | Day 10 |
| Route Tables / User Defined Routes (UDR) | Day 11 |
| NAT Gateway | Day 11 |
| Microsoft Entra ID | Day 17 |
| Azure Monitor | Day 20 |
| Application Gateway | Day 12 |
| WAF (Web Application Firewall) | Day 14 |
| Load Balancer | Day 12 |
| VM Scale Sets (VMSS) | Day 5 + Day 12 |
| Traffic Manager | Day 14 |
| Azure Front Door | Day 14 |
| Azure SQL Database | Day 16 |
| Azure CDN | Day 14 |
| Key Vault | Day 19 |
| Backup Service (Azure Backup) | Day 5 |
| ARM Templates | Day 26 |
| Azure DNS | Day 13 |
| VPN Gateway / ExpressRoute | Day 15 |
| Azure Kubernetes Service | Day 29–30 |
| Containers / ACR | Day 28 |
| Azure DevOps | Day 21–25 |
| Bicep / Terraform | Day 26–27 |
| Cosmos DB | Day 33 (Optional Bonus) |
| Function App / Serverless | Day 34 (Optional Bonus) |
| API Management (API Gateway) | Day 35 (Optional Bonus) |

---

## Instructions for Claude Code Sessions

When starting a new session to write course content:

1. **Read this file first** to load course context.
2. **Read the specific day file** from `days/` for the day you are working on (e.g., `days/functions.md`).
3. **Output format for each day's script:**
   - File name: `dayXX_topic_name.md` saved to `docs/`
   - Sections: Hook → Core Concepts (with Hands-On after each section) → Summary → Key Takeaways
   - All Portal Demo steps must carry ✅ or 💳 tier labels
4. **No AWS, GCP, or on-premise comparisons** — Azure-native explanations only.
5. **Scope per session:** Write ONE day at a time.
6. **Tone:** Conversational, enthusiastic, instructor-speaking-to-camera. Not dry documentation.

---

*Last updated: August 2026 | LearnWithMithran | Azure Cloud + Azure DevOps Course*
