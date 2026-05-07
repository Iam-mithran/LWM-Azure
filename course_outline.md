# LearnWithMithran — Azure Cloud & Azure DevOps Course Outline
> Master reference file for Claude Code sessions. Read this file at the start of every session.

---

## Channel & Course Context

- **Channel:** LearnWithMithran (YouTube)
- **Course:** Azure Cloud + Azure DevOps — A Complete Beginner's Guide
- **Total Videos:** 29 (Day 0 through Day 28)
- **Total Phases:** 9 (Phase 0 through Phase 8)
- **Max Video Length:** 2 hours per video
- **Target Audience:** Complete beginners to Azure — but existing subscribers who already know AWS, DevOps fundamentals (Docker, Kubernetes, Jenkins, Git, CI/CD)

---

## Teaching Philosophy & Ground Rules

1. **AWS-to-Azure Comparisons First** — Always anchor new Azure concepts by comparing to the AWS equivalent. Example: "Azure VM = AWS EC2", "Azure Blob Storage = AWS S3", "Azure AD = AWS IAM". This is the #1 pedagogical device throughout the course.
2. **Console-First Always** — Every concept is demonstrated via the Azure Portal (GUI). Students learn by clicking, not typing commands.
3. **CLI is Contextual Only** — Azure CLI is shown only where it genuinely saves time (e.g., repetitive tasks, quick lookups). CLI usage is capped at 3–5% of total content. There is NO standalone CLI-only video.
4. **Hands-On Every Day** — Every video includes at least one practical demo or lab the viewer can follow along.
5. **No Assumed Azure Knowledge** — Treat every viewer as Azure-zero. Assume they know AWS/DevOps but nothing about Azure's naming, portal, or billing.
6. **Script Structure Per Video:**
   - Hook (what problem does this solve?) — ~2 min
   - AWS-to-Azure comparison — ~3–5 min
   - Core concept explanation — ~10–15 min
   - Portal demo / hands-on — ~20–40 min
   - Summary + what's next — ~2–3 min

---

## Phase Overview

| Phase | Days | Topic |
|-------|------|-------|
| Phase 0 | Day 0 – Day 1 | Foundations + Account Setup |
| Phase 1 | Day 2 – Day 5 | Compute |
| Phase 2 | Day 6 – Day 8 | Networking |
| Phase 3 | Day 9 – Day 12 | Storage + Databases |
| Phase 4 | Day 13 – Day 16 | Identity, Security + Monitoring |
| Phase 5 | Day 17 – Day 21 | Azure DevOps |
| Phase 6 | Day 22 – Day 23 | Infrastructure as Code (IaC) |
| Phase 7 | Day 24 – Day 26 | Containers + AKS |
| Phase 8 | Day 27 – Day 28 | Capstone Project |

---

## Detailed Day-by-Day Syllabus

---

### Phase 0 — Foundations + Account Setup (Day 0–1)

---

#### Day 0 — Azure Account Setup & Free Tier Walkthrough

**Goal:** Get every viewer set up with a free Azure account, understand the free tier, and feel confident before spending a rupee.

**Key Topics:**
- What is Azure? (Microsoft's cloud — AWS equivalent overview)
- Azure vs AWS: High-level positioning
- Creating a free Azure account at portal.azure.com
  - $200 free credit for 30 days
  - 12 months of free services after credit expires
  - Always-free services (25+ services that are always free)
- Credit card requirement — reassurance for Indian students (card is for identity verification; you won't be charged without upgrading)
- Azure for Students option (if applicable — no credit card needed)
- Setting up Budget Alerts to avoid surprise bills
- Tour of the Azure Portal (home screen, search bar, resource groups, subscriptions)
- Azure regions — what they are and how to pick one (compare to AWS regions/AZs)

**AWS Comparison:** AWS Free Tier vs Azure Free Account structure

**Hands-On Demo:**
- Create Azure free account live
- Set a $1 budget alert
- Navigate the portal: find Subscriptions, Resource Groups, Cost Management

---

#### Day 1 — Azure Fundamentals: Core Concepts & Portal Deep Dive

**Goal:** Build a mental model of how Azure is organized before touching any services.

**Key Topics:**
- Azure Global Infrastructure: Regions, Availability Zones, Region Pairs
  - Compare: AWS Regions → Azure Regions, AWS AZs → Azure Availability Zones
- Azure Resource Hierarchy:
  - Management Groups → Subscriptions → Resource Groups → Resources
  - Compare: AWS Organizations → AWS Accounts → (no RG equivalent) → Resources
- What is a Resource Group? Why it matters (logical container for related resources)
- Azure Resource Manager (ARM) — the control plane behind everything
  - Compare: AWS CloudFormation / AWS API → ARM
- Naming conventions and tagging best practices
- Azure Marketplace overview
- Azure documentation and learning resources (Microsoft Learn)

**AWS Comparison:** AWS account structure vs Azure subscription + resource group model

**Hands-On Demo:**
- Create a Resource Group
- Apply tags to it
- Explore Azure Marketplace
- Browse Microsoft Learn

---

### Phase 1 — Compute (Day 2–5)

---

#### Day 2 — Azure Virtual Machines (VMs) — Part 1: Creating Your First VM

**Goal:** Create and connect to an Azure VM from scratch.

**Key Topics:**
- What is Azure Virtual Machines? (Compare: AWS EC2)
- VM sizes and series (General Purpose, Compute Optimized, Memory Optimized)
  - Compare: AWS instance types (t3, m5, c5, r5 → Azure B-series, D-series, F-series, E-series)
- VM pricing: Pay-as-you-go, Reserved Instances, Spot VMs
  - Compare: AWS On-Demand, Reserved, Spot
- OS options: Windows Server, Ubuntu, RHEL, etc.
- Managed Disks: OS disk, Data disk (Compare: AWS EBS)
- Public IP addresses and DNS
- Network Security Groups (NSG) — intro (Compare: AWS Security Groups)

**Hands-On Demo:**
- Create an Ubuntu VM via Azure Portal
- Configure size, disk, NSG rules (open port 22)
- Connect via SSH
- Stop and deallocate the VM (explain cost saving)

---

#### Day 3 — Azure Virtual Machines — Part 2: VM Management & Availability

**Goal:** Manage VMs like a professional — availability, scaling, snapshots.

**Key Topics:**
- Availability Sets vs Availability Zones (Compare: AWS placement groups / Multi-AZ)
- VM Scale Sets — auto scaling group equivalent (Compare: AWS Auto Scaling Groups)
- VM snapshots and images (Compare: AWS AMIs)
- Custom Script Extensions — run scripts on VMs post-deployment
- Azure Bastion — secure browser-based SSH/RDP without public IP (Compare: AWS Session Manager)
- VM monitoring basics: CPU, memory, disk metrics in portal
- Shutting down vs deallocating (the cost trap beginners fall into)

**Hands-On Demo:**
- Create a VM snapshot
- Enable Azure Bastion and connect via browser
- Set up a basic alert on CPU > 80%

---

#### Day 4 — Azure App Service — Deploying Web Apps Without Managing Servers

**Goal:** Deploy a web application to App Service (PaaS) and understand when to choose it over VMs.

**Key Topics:**
- What is Azure App Service? (Compare: AWS Elastic Beanstalk / AWS App Runner)
- App Service Plans: Free, Shared, Basic, Standard, Premium, Isolated
  - Compare: EC2 instance sizing under Beanstalk
- Supported runtimes: Node.js, Python, Java, .NET, PHP, Ruby
- Deployment methods: ZIP deploy, GitHub Actions, Azure DevOps, FTP
- Custom domains and SSL certificates
- App Service Environment (ASE) — brief mention for enterprise context
- Scaling: Manual scale, Auto scale rules
- Application settings and environment variables (Compare: AWS Elastic Beanstalk env vars)
- Deployment slots: staging vs production (Compare: AWS Beanstalk environments)

**Hands-On Demo:**
- Create an App Service Plan
- Deploy a simple Node.js or Python app via ZIP deploy
- Set environment variables
- Scale the app manually

---

#### Day 5 — Azure Functions — Serverless Computing

**Goal:** Build and deploy a serverless function and understand the event-driven model.

**Key Topics:**
- What is serverless? (Compare: AWS Lambda)
- Azure Functions vs AWS Lambda: triggers, bindings, pricing model
- Triggers: HTTP, Timer, Blob Storage, Queue, Event Hub
- Bindings: input and output bindings concept
- Function App vs individual Functions
- Hosting plans: Consumption (serverless), Premium, Dedicated (App Service)
  - Compare: AWS Lambda provisioned concurrency
- Cold start problem — what it is, how to mitigate
- Durable Functions — brief intro (stateful serverless, compare to AWS Step Functions)
- Monitoring Functions with Application Insights

**Hands-On Demo:**
- Create a Function App
- Write an HTTP-triggered function in the portal editor
- Test it with a browser request
- View execution logs in Application Insights

---

### Phase 2 — Networking (Day 6–8)

---

#### Day 6 — Azure Virtual Network (VNet) — Core Networking

**Goal:** Understand Azure networking from scratch and create a working VNet with subnets.

**Key Topics:**
- What is a Virtual Network (VNet)? (Compare: AWS VPC)
- VNet address space, subnets (Compare: AWS CIDR blocks and subnets)
- Public vs Private subnets
- Network Security Groups (NSG) — detailed walkthrough (Compare: AWS Security Groups + NACLs combined)
- NSG rules: inbound and outbound, priority, deny/allow
- VNet Peering — connecting two VNets (Compare: AWS VPC Peering)
- Service Endpoints vs Private Endpoints (Compare: AWS VPC Endpoints)
- Azure DNS basics

**Hands-On Demo:**
- Create a VNet with 2 subnets (public and private)
- Attach NSG to subnet
- Create NSG rules for SSH and HTTP
- Deploy a VM into the private subnet

---

#### Day 7 — Azure Load Balancer & Application Gateway

**Goal:** Distribute traffic across VMs and understand Layer 4 vs Layer 7 load balancing.

**Key Topics:**
- Azure Load Balancer (Layer 4) — Compare: AWS Network Load Balancer (NLB)
- Application Gateway (Layer 7) — Compare: AWS Application Load Balancer (ALB)
- Load Balancer SKUs: Basic vs Standard
- Backend pools, health probes, load balancing rules
- Application Gateway: URL-based routing, SSL termination, WAF integration
- Azure Front Door — global load balancing + CDN (Compare: AWS CloudFront + Global Accelerator)
- Traffic Manager — DNS-based global routing (Compare: AWS Route 53 routing policies)

**Hands-On Demo:**
- Create a Standard Load Balancer
- Add two VMs to a backend pool
- Configure a health probe and load balancing rule
- Test traffic distribution

---

#### Day 8 — VPN Gateway & ExpressRoute

**Goal:** Connect on-premises networks to Azure securely.

**Key Topics:**
- Hybrid networking concepts (Compare: AWS hybrid networking)
- Azure VPN Gateway: Site-to-Site VPN (Compare: AWS Site-to-Site VPN)
- Point-to-Site VPN (Connect your laptop to Azure VNet — Compare: AWS Client VPN)
- ExpressRoute: dedicated private connection (Compare: AWS Direct Connect)
- VPN Gateway SKUs and performance tiers
- When to use VPN vs ExpressRoute (cost, latency, reliability tradeoffs)
- Azure Virtual WAN — brief intro for large-scale networking

**Hands-On Demo:**
- Create a VPN Gateway (note: takes 30–45 mins to deploy; explain while waiting)
- Walk through Point-to-Site VPN configuration
- Show ExpressRoute setup in portal (conceptual walkthrough, no physical line needed)

---

### Phase 3 — Storage + Databases (Day 9–12)

---

#### Day 9 — Azure Storage Account — Blob, Files, Queues, Tables

**Goal:** Understand Azure's unified storage offering and work with Blob Storage.

**Key Topics:**
- Azure Storage Account — the container for all storage types (no AWS direct equivalent — AWS splits these into S3, EFS, SQS, DynamoDB)
- Storage types within one account:
  - Blob Storage (Compare: AWS S3)
  - Azure Files (Compare: AWS EFS)
  - Queue Storage (Compare: AWS SQS)
  - Table Storage (Compare: AWS DynamoDB — basic key-value)
- Storage Account types: LRS, ZRS, GRS, GZRS (replication options — Compare: AWS S3 replication classes)
- Access tiers: Hot, Cool, Cold, Archive (Compare: AWS S3 Standard, IA, Glacier)
- Storage Account performance tiers: Standard vs Premium
- Containers, blobs, and blob types (Block, Append, Page blobs)
- Shared Access Signatures (SAS) for secure, time-limited access (Compare: AWS pre-signed URLs)
- Storage lifecycle management policies

**Hands-On Demo:**
- Create a Storage Account
- Upload files to Blob Storage via portal
- Generate a SAS URL and access via browser
- Set up a lifecycle policy (move to Cool after 30 days)

---

#### Day 10 — Azure SQL Database — Managed Relational Database

**Goal:** Deploy and connect to a fully managed SQL database on Azure.

**Key Topics:**
- Azure SQL Database — fully managed PaaS (Compare: AWS RDS)
- Deployment models: Single Database, Elastic Pool, Managed Instance
  - Compare: AWS RDS Single, Aurora Serverless, RDS on EC2
- Service tiers: Basic, Standard, Premium, General Purpose, Business Critical, Hyperscale
- DTU vs vCore purchasing model
- Serverless compute tier (auto-pause, auto-scale)
- Backups, geo-restore, point-in-time restore (Compare: AWS RDS automated backups)
- Firewall rules and private endpoints
- Azure SQL vs SQL Server on VM: when to choose each
- Connection strings and Azure Data Studio

**Hands-On Demo:**
- Create an Azure SQL Database (single database)
- Configure firewall to allow your IP
- Connect using Azure Data Studio or Query Editor in portal
- Run basic SQL queries

---

#### Day 11 — Azure Cosmos DB — NoSQL at Global Scale

**Goal:** Understand multi-model NoSQL and when to choose Cosmos DB.

**Key Topics:**
- What is Cosmos DB? (Compare: AWS DynamoDB — but Cosmos is multi-model)
- APIs supported: Core (SQL), MongoDB, Cassandra, Gremlin, Table
- Global distribution and multi-region writes (Compare: AWS DynamoDB Global Tables)
- Consistency levels: Strong, Bounded Staleness, Session, Consistent Prefix, Eventual
- Request Units (RUs) — Cosmos DB's capacity unit
- Partitioning strategy: choosing a partition key
- Serverless vs Provisioned throughput
- Cosmos DB Free Tier (400 RU/s + 5GB always free)
- Change Feed — event-driven data processing (Compare: DynamoDB Streams)

**Hands-On Demo:**
- Create a Cosmos DB account with Core (SQL) API
- Create a database and container
- Insert and query documents via Data Explorer
- Observe RU consumption

---

#### Day 12 — Other Azure Databases + Storage Deep Dive

**Goal:** Survey remaining database options and advanced storage features.

**Key Topics:**
- Azure Database for PostgreSQL (Compare: AWS RDS PostgreSQL / Aurora PostgreSQL)
- Azure Database for MySQL (Compare: AWS RDS MySQL / Aurora MySQL)
- Azure Cache for Redis (Compare: AWS ElastiCache Redis)
- Azure Disk Storage: Managed Disks — Standard HDD, Standard SSD, Premium SSD, Ultra Disk (Compare: AWS EBS volume types)
- Azure NetApp Files — enterprise NFS (Compare: AWS FSx)
- Storage security: encryption at rest (default), encryption in transit, CMK with Azure Key Vault
- Azure Storage Explorer — desktop tool walkthrough

**Hands-On Demo:**
- Create an Azure Database for PostgreSQL — Flexible Server
- Connect via psql or pgAdmin
- Enable Azure Cache for Redis and test basic set/get via portal console

---

### Phase 4 — Identity, Security + Monitoring (Day 13–16)

---

#### Day 13 — Microsoft Entra ID (Azure Active Directory) — Identity Foundation

**Goal:** Understand Azure's identity system and how it differs fundamentally from AWS IAM.

**Key Topics:**
- What is Microsoft Entra ID (formerly Azure Active Directory)?
  - Compare: AWS IAM — but explain the critical difference: Entra ID is for human identities (users/groups), IAM is for both humans and services
- Tenants, directories, and subscriptions — how they relate
- Users: cloud-only vs synced from on-prem AD
- Groups: Security groups vs Microsoft 365 groups
- Azure AD Free vs Premium P1 vs Premium P2 tiers
- Service Principals & App Registrations (Compare: AWS IAM Roles for applications)
- Managed Identities — let Azure services authenticate without credentials (Compare: AWS IAM Instance Profiles / IAM Roles for services)
- Multi-Factor Authentication (MFA) setup
- Self-Service Password Reset (SSPR)
- Conditional Access — access policies based on conditions (no AWS equivalent)

**Hands-On Demo:**
- Create a user in Entra ID
- Create a security group and add user
- Enable MFA for the user
- Create a Managed Identity and assign it to a VM

---

#### Day 14 — Azure RBAC — Role-Based Access Control

**Goal:** Control who can do what in Azure using roles and assignments.

**Key Topics:**
- Azure RBAC model: Scope → Role → Principal
  - Scope: Management Group > Subscription > Resource Group > Resource
  - Compare: AWS IAM Policies attached to Users/Roles/Groups
- Built-in roles: Owner, Contributor, Reader, and 70+ service-specific roles (Compare: AWS managed policies)
- Custom roles: when and how to create them (Compare: AWS custom IAM policies)
- Role assignments: how to assign a role to a user/group/service principal at a scope
- Deny assignments (special — no AWS equivalent for explicit deny at assignment level)
- Azure Policy: enforce governance rules across subscriptions (Compare: AWS SCPs / AWS Config Rules)
- Azure Blueprints — brief mention (governance templates)
- Principle of Least Privilege in practice

**Hands-On Demo:**
- Assign the Reader role to a user at Resource Group scope
- Create a custom role with specific permissions
- Create an Azure Policy to enforce a tag on all resources
- Verify policy enforcement by attempting to create an untagged resource

---

#### Day 15 — Azure Key Vault & Security Center

**Goal:** Store secrets securely and understand Azure's security posture tools.

**Key Topics:**
- Azure Key Vault: secrets, keys, and certificates (Compare: AWS Secrets Manager + KMS combined)
  - Secrets: API keys, passwords, connection strings
  - Keys: cryptographic keys for encryption
  - Certificates: SSL/TLS certificate management
- Key Vault access policies vs RBAC model
- Soft delete and purge protection
- Integrating Key Vault with App Service / Azure Functions (reference secrets without hardcoding)
- Microsoft Defender for Cloud (formerly Security Center) — Compare: AWS Security Hub + AWS GuardDuty
  - Secure Score: what it is and how to improve it
  - Security recommendations
  - Regulatory compliance dashboard
- Microsoft Sentinel — brief intro (SIEM — Compare: AWS Security Lake / third-party SIEMs)

**Hands-On Demo:**
- Create a Key Vault
- Store a secret (database connection string)
- Retrieve the secret from an App Service using Managed Identity (no hardcoded credentials)
- Review Defender for Cloud Secure Score

---

#### Day 16 — Azure Monitor, Log Analytics & Alerts

**Goal:** Set up monitoring, logging, and alerting for Azure resources.

**Key Topics:**
- Azure Monitor — the unified monitoring platform (Compare: AWS CloudWatch)
- Metrics vs Logs: what's the difference
- Log Analytics Workspace — centralized log storage and querying (Compare: AWS CloudWatch Logs + Insights)
- Kusto Query Language (KQL) — basics for querying logs (Compare: CloudWatch Logs Insights query syntax)
- Application Insights — APM for apps (Compare: AWS X-Ray + CloudWatch Application Insights)
- Diagnostic Settings — how to route logs from any resource to Log Analytics / Storage / Event Hub
- Azure Monitor Alerts: metric alerts, log alerts, activity log alerts (Compare: AWS CloudWatch Alarms + EventBridge)
- Action Groups: email, SMS, webhook, runbook (Compare: AWS SNS + Lambda)
- Azure Dashboard — building a custom monitoring dashboard
- Azure Workbooks — interactive report templates

**Hands-On Demo:**
- Enable Diagnostic Settings on a VM → send to Log Analytics
- Write a basic KQL query (show CPU usage over last hour)
- Create a metric alert (CPU > 80%) with email action group
- Build a simple Azure Dashboard with 3–4 pinned metrics

---

### Phase 5 — Azure DevOps (Day 17–21)

---

#### Day 17 — Azure DevOps Introduction & Organization Setup

**Goal:** Set up Azure DevOps from zero and understand its components.

**Key Topics:**
- What is Azure DevOps? (Compare: GitHub + Jenkins + Jira combined into one Microsoft platform)
- Azure DevOps services overview:
  - Boards (Compare: Jira)
  - Repos (Compare: GitHub / GitLab)
  - Pipelines (Compare: Jenkins / GitHub Actions)
  - Test Plans (Compare: TestRail)
  - Artifacts (Compare: AWS CodeArtifact / Nexus)
- Azure DevOps vs GitHub Actions: when to use which
- Creating an Azure DevOps organization and project
- Linking Azure DevOps to an Azure Subscription
- Free tier limits (5 users free, 1 Microsoft-hosted agent free)
- Navigating the Azure DevOps portal

**Hands-On Demo:**
- Create Azure DevOps organization at dev.azure.com
- Create a project (Agile template)
- Set up Boards: create epics, user stories, tasks
- Invite a team member (or simulate with a second account)

---

#### Day 18 — Azure Repos — Git in Azure DevOps

**Goal:** Use Azure Repos for version control and understand branching strategies.

**Key Topics:**
- Azure Repos: Git repositories in Azure DevOps (Compare: GitHub repos)
- Git vs TFVC — always choose Git
- Creating and cloning a repository
- Branching strategies: feature branches, GitFlow, trunk-based development
- Branch policies: require PR, minimum reviewers, linked work items, build validation (Compare: GitHub branch protection rules)
- Pull Requests in Azure DevOps: creating, reviewing, approving, completing
- Code review best practices
- Connecting local VS Code to Azure Repos

**Hands-On Demo:**
- Create a repo in Azure DevOps
- Push a sample app (Node.js or Python) from local machine
- Create a feature branch, make a change, open a PR
- Set up a branch policy requiring 1 reviewer
- Complete the PR and merge

---

#### Day 19 — Azure Pipelines — CI (Continuous Integration)

**Goal:** Build an automated CI pipeline that runs on every code push.

**Key Topics:**
- What is CI/CD? Quick refresher (audience knows Jenkins)
- Azure Pipelines vs Jenkins: key differences (Compare in depth)
- YAML pipelines vs Classic (GUI) pipelines — always use YAML
- Pipeline anatomy: triggers, stages, jobs, steps, tasks
- Microsoft-hosted agents vs self-hosted agents (Compare: Jenkins agents/nodes)
- Common tasks: Checkout, build (npm/maven/dotnet), test, publish artifacts
- Pipeline variables and variable groups
- Secrets in pipelines: using Key Vault linked variable groups
- Build artifacts and artifact publishing

**Hands-On Demo:**
- Create a YAML pipeline from scratch (azure-pipelines.yml)
- Trigger: on push to main branch
- Steps: checkout → install dependencies → run tests → publish artifact
- View pipeline run results, logs, and test reports

---

#### Day 20 — Azure Pipelines — CD (Continuous Delivery/Deployment)

**Goal:** Deploy automatically to Azure App Service or a VM using release pipelines.

**Key Topics:**
- CD concepts: Continuous Delivery vs Continuous Deployment
- Environments in Azure Pipelines (Compare: Jenkins environments / stages)
- Deployment strategies: Rolling, Blue-Green, Canary (Compare: Jenkins pipeline stages)
- Azure Pipeline deployment jobs
- Approvals and gates: manual approval before production deployment
- Deploying to Azure App Service from pipeline
- Deploying to Azure VM using deployment groups
- Rollback strategies
- Multi-stage YAML pipeline: CI + CD in one file

**Hands-On Demo:**
- Extend the Day 19 pipeline to add a Deploy stage
- Deploy to Azure App Service (staging slot)
- Add a manual approval gate before production
- Swap staging slot to production

---

#### Day 21 — Azure Artifacts & Pipeline Best Practices

**Goal:** Manage package dependencies internally and wrap up Azure DevOps with best practices.

**Key Topics:**
- Azure Artifacts: private package registry (Compare: AWS CodeArtifact / Nexus / JFrog)
- Supported package types: npm, NuGet, Maven, Python (PyPI), Universal Packages
- Creating a feed and publishing packages
- Upstream sources: proxy public registries through Azure Artifacts
- Pipeline caching: speed up pipelines by caching dependencies (Compare: GitHub Actions cache)
- Pipeline templates: reusable YAML templates across projects
- Service connections: securely connecting pipelines to Azure, Docker Hub, GitHub, etc.
- Azure DevOps security: project permissions, pipeline permissions

**Hands-On Demo:**
- Create an Azure Artifacts feed
- Publish an npm package to the feed from a pipeline
- Reference the internal feed in a downstream project
- Create a reusable pipeline template and reference it

---

### Phase 6 — Infrastructure as Code (Day 22–23)

---

#### Day 22 — ARM Templates & Bicep

**Goal:** Define Azure infrastructure as code using native Azure tools.

**Key Topics:**
- What is Infrastructure as Code (IaC)? Quick refresher (audience knows Terraform/CloudFormation)
- ARM Templates: JSON-based IaC for Azure (Compare: AWS CloudFormation)
- ARM Template structure: schema, parameters, variables, resources, outputs
- Limitations of ARM JSON: verbose, hard to read
- Azure Bicep: domain-specific language that compiles to ARM (Compare: AWS CDK — a higher-level abstraction over CloudFormation)
- Bicep vs ARM: why Bicep wins for new projects
- Bicep syntax: resources, parameters, variables, modules, outputs
- Deploying Bicep files via Azure CLI / Azure Portal
- What-if deployments (dry run — Compare: terraform plan)
- Azure Deployment Stacks

**Hands-On Demo:**
- Write a Bicep file to deploy a Storage Account + Resource Group
- Run what-if to preview changes
- Deploy via Azure CLI (one of the few CLI moments in the course)
- Modify and redeploy — show idempotency

---

#### Day 23 — Terraform on Azure

**Goal:** Use Terraform (which the audience knows from AWS) to provision Azure infrastructure.

**Key Topics:**
- Terraform recap: why it's the industry standard for multi-cloud IaC (audience knows this)
- Azure Provider for Terraform (azurerm)
- Authenticating Terraform with Azure: Service Principal, Managed Identity, Azure CLI
- Terraform state on Azure: using Azure Blob Storage as remote backend (Compare: AWS S3 + DynamoDB backend)
- Core resources: azurerm_resource_group, azurerm_virtual_network, azurerm_linux_virtual_machine
- Terraform workspaces for environment separation (dev/staging/prod)
- Integrating Terraform with Azure DevOps Pipelines (plan on PR, apply on merge)

**Hands-On Demo:**
- Write Terraform to deploy: Resource Group + VNet + Subnet + VM
- Configure Azure Blob Storage as remote backend
- Run init → plan → apply
- Add Terraform plan step to an Azure DevOps pipeline

---

### Phase 7 — Containers + AKS (Day 24–26)

---

#### Day 24 — Azure Container Registry (ACR) & Docker on Azure

**Goal:** Store and manage Docker images in Azure's private registry.

**Key Topics:**
- Docker recap: images, containers, Dockerfile (audience knows Docker)
- Azure Container Registry (ACR) (Compare: AWS ECR)
- ACR SKUs: Basic, Standard, Premium
- Pushing images to ACR from local machine
- ACR Tasks: build images in the cloud without local Docker (Compare: AWS CodeBuild for container builds)
- Geo-replication in ACR Premium (Compare: ECR replication)
- Image scanning with Microsoft Defender for Containers (Compare: AWS ECR image scanning)
- Integrating ACR with Azure DevOps pipeline (build and push in CI)
- Azure Container Instances (ACI): run containers without orchestration (Compare: AWS Fargate standalone tasks / AWS ECS without cluster management)

**Hands-On Demo:**
- Create an ACR
- Build a simple Docker image locally and push to ACR
- Create an Azure Container Instance from the ACR image
- Build an ACR Task that auto-builds on GitHub push

---

#### Day 25 — Azure Kubernetes Service (AKS) — Setup & Core Concepts

**Goal:** Deploy a managed Kubernetes cluster and understand AKS architecture.

**Key Topics:**
- Kubernetes recap: pods, deployments, services, namespaces (audience knows K8s)
- AKS vs self-managed Kubernetes (Compare: AWS EKS)
- AKS architecture: control plane (managed by Microsoft, free), node pools
- Node pool types: System node pool, User node pool
- VM sizes for nodes: choosing the right SKU
- AKS networking modes: Kubenet vs Azure CNI (Compare: EKS VPC CNI)
- AKS authentication: integrating with Entra ID for RBAC
- kubectl setup for AKS: az aks get-credentials
- AKS Add-ons: monitoring (Azure Monitor for containers), ingress (AGIC)
- AKS pricing: only pay for worker nodes, not control plane

**Hands-On Demo:**
- Create an AKS cluster via Azure Portal
- Get credentials and connect with kubectl
- Deploy a sample app (2-tier: frontend + backend) using YAML manifests
- Expose via LoadBalancer service — get public IP

---

#### Day 26 — AKS Advanced — Scaling, Ingress & CI/CD Integration

**Goal:** Scale AKS workloads and integrate with Azure DevOps for full GitOps-style deployment.

**Key Topics:**
- Horizontal Pod Autoscaler (HPA) (same as standard K8s HPA)
- Cluster Autoscaler — automatically scale node count (Compare: AWS EKS Cluster Autoscaler / Karpenter)
- Azure Kubernetes KEDA — event-driven autoscaling (Compare: KEDA on EKS)
- Ingress controllers: NGINX Ingress vs Application Gateway Ingress Controller (AGIC) (Compare: AWS ALB Ingress Controller)
- AKS + Azure DevOps pipeline: build image → push to ACR → deploy to AKS
- Helm charts basics: packaging Kubernetes apps (audience may know Helm)
- Persistent storage in AKS: Azure Disk (RWO) and Azure Files (RWX) storage classes
- AKS security: network policies, pod identity with Workload Identity

**Hands-On Demo:**
- Enable Cluster Autoscaler on AKS node pool
- Configure HPA for the sample app (scale on CPU)
- Set up NGINX Ingress controller
- Build full CI/CD: Azure DevOps pipeline → ACR → AKS deployment

---

### Phase 8 — Capstone Project (Day 27–28)

---

#### Day 27 — Capstone Part 1: Architecture Design & Infrastructure Setup

**Goal:** Design and build the infrastructure for a real-world 3-tier application on Azure.

**Capstone App:** A simple 3-tier web application (Frontend → API Backend → Database)

**Key Topics:**
- Architecture design session: walk through the complete architecture diagram
  - Azure App Service (frontend)
  - AKS (API backend containers)
  - Azure SQL Database (data layer)
  - Azure Key Vault (secrets)
  - ACR (container images)
  - Azure Monitor + Application Insights (observability)
  - VNet with subnets (networking)
  - Entra ID + RBAC (identity)
- Infrastructure provisioning with Terraform (or Bicep)
- Networking: VNet, subnets, NSGs, private endpoints for DB
- Identity: Managed Identities, Key Vault integration
- ACR setup and initial image push

**Hands-On Demo:**
- Provision full infrastructure with Terraform
- Deploy Azure SQL Database with private endpoint
- Set up Key Vault with connection string secret
- Configure ACR and push backend Docker image

---

#### Day 28 — Capstone Part 2: CI/CD Pipeline, Monitoring & Wrap-Up

**Goal:** Complete the capstone with a full CI/CD pipeline, monitoring setup, and course wrap-up.

**Key Topics:**
- Azure DevOps pipeline for the full application:
  - Stage 1: CI — build + test + push Docker image to ACR
  - Stage 2: Deploy to AKS (dev environment)
  - Stage 3: Manual approval gate
  - Stage 4: Deploy to AKS (production)
- Application Insights integration in the backend app
- Log Analytics: KQL queries for app logs
- Azure Monitor alerts: API error rate, latency alerts
- Course wrap-up: what you've learned across all 9 phases
- What's next: Azure certifications (AZ-900, AZ-104, AZ-204, AZ-400)
- LearnWithMithran: upcoming content preview

**Hands-On Demo:**
- Build and run the full CI/CD pipeline end-to-end
- Show the live application running on AKS
- Trigger an alert by simulating high CPU
- View Application Insights live metrics

---

## Quick Reference: AWS-to-Azure Cheat Sheet

| AWS Service | Azure Equivalent |
|-------------|-----------------|
| EC2 | Azure Virtual Machines |
| Auto Scaling Groups | VM Scale Sets |
| Elastic Beanstalk | Azure App Service |
| AWS Lambda | Azure Functions |
| VPC | Virtual Network (VNet) |
| Security Groups | Network Security Groups (NSG) |
| NACLs | NSG (combined with Security Groups) |
| Route 53 | Azure DNS + Traffic Manager |
| ALB | Application Gateway |
| NLB | Azure Load Balancer |
| CloudFront | Azure Front Door / Azure CDN |
| S3 | Azure Blob Storage |
| EBS | Azure Managed Disks |
| EFS | Azure Files |
| SQS | Azure Queue Storage |
| RDS | Azure SQL Database / Azure DB for PostgreSQL / MySQL |
| DynamoDB | Azure Cosmos DB |
| ElastiCache | Azure Cache for Redis |
| IAM (users/groups) | Microsoft Entra ID |
| IAM Roles (for services) | Managed Identities |
| IAM Policies | Azure RBAC Roles |
| AWS Organizations / SCPs | Azure Management Groups + Azure Policy |
| Secrets Manager + KMS | Azure Key Vault |
| CloudWatch | Azure Monitor + Log Analytics + Application Insights |
| CloudFormation | ARM Templates / Bicep |
| CDK | Bicep (conceptually) |
| Terraform (AWS provider) | Terraform (azurerm provider) |
| CodePipeline + CodeBuild | Azure DevOps Pipelines |
| CodeArtifact | Azure Artifacts |
| ECR | Azure Container Registry (ACR) |
| EKS | Azure Kubernetes Service (AKS) |
| Fargate | Azure Container Instances (ACI) |
| X-Ray | Application Insights |
| Security Hub + GuardDuty | Microsoft Defender for Cloud |
| Direct Connect | ExpressRoute |
| Site-to-Site VPN | Azure VPN Gateway |
| Client VPN | Azure Point-to-Site VPN |
| Session Manager | Azure Bastion |

---

## Instructions for Claude Code Sessions

When starting a new Claude Code session to write course content:

1. **Always read this file first** to get full course context.
2. **Specify the exact Day** you are working on (e.g., "Write content for Day 5 — Azure Functions").
3. **Output format for each day's content:**
   - File name: `dayXX_topic_name.md` (e.g., `day05_azure_functions.md`)
   - Include: Hook, AWS Comparison, Core Concepts, Portal Demo Steps, Summary, Key Takeaways
4. **Scope per session:** Write ONE day at a time. Do not attempt multiple days in one session.
5. **Length target:** Each day's script should be 1,500–3,000 words (maps to ~30–60 min video content; demos make up the remaining time).
6. **Tone:** Conversational, enthusiastic, instructor-speaking-to-camera. Not dry documentation.

---

*Last updated: May 2026 | LearnWithMithran | Azure Cloud + Azure DevOps Course*
