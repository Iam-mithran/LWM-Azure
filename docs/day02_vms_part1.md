# Day 2 — Azure Virtual Machines Part 1: Creating Your First VM

**Phase 1 — Compute**

> Today you're going to rent a computer in Microsoft's data center, configure it exactly how you want, connect to it from your laptop, and learn how to shut it down without wasting money. By the end of this session, you'll have a real Linux server running in the cloud — one you built from scratch.

---

## What You'll Learn

- What Azure Virtual Machines actually are and how they work under the hood
- VM series and sizes — and which one to pick for what job
- Pricing models: Pay-as-you-go, Reserved Instances, and Spot VMs
- OS options and the Azure Marketplace
- Managed Disks — OS disks vs Data disks, and the four disk tiers
- Public IP addresses and DNS labels
- Network Security Groups — the firewall that controls what can reach your VM
- Full walkthrough: creating an Ubuntu VM from zero in the Azure Portal
- Connecting to your VM over SSH
- The critical difference between stopping and deallocating a VM

---

## What Is a Virtual Machine?

A **Virtual Machine (VM)** is a software-defined computer running inside a physical server in one of Microsoft's data centers. You don't buy or touch that server — you rent a slice of it.

```mermaid
graph TD
    subgraph DC["Microsoft Data Center — East US"]
        subgraph PHYS["Physical Server — 96 vCPUs, 384 GB RAM"]
            H["Hypervisor\n(manages isolation between VMs)"]

            subgraph VM1["Your VM — Standard_B1s"]
                C1["1 vCPU"]
                M1["1 GB RAM"]
                D1["30 GB OS Disk"]
            end

            subgraph VM2["Another customer's VM"]
                C2["2 vCPUs"]
                M2["8 GB RAM"]
                D2["64 GB OS Disk"]
            end

            subgraph VM3["Another customer's VM"]
                C3["4 vCPUs"]
                M3["16 GB RAM"]
                D3["128 GB OS Disk"]
            end

            H --> VM1
            H --> VM2
            H --> VM3
        end
    end
```

**What you control on a VM:**
- The operating system (Ubuntu, Windows Server, RHEL — your choice)
- Everything installed on it (web servers, databases, custom software)
- Network rules that control who can reach it
- The size and number of disks attached

**What Azure controls:**
- The physical hardware underneath
- The hypervisor that keeps your VM isolated from others
- Power, cooling, and physical security in the data center
- Hardware failures (if the physical server dies, Azure migrates your VM)

This separation is what makes cloud VMs compelling: you get the experience of owning a server without any of the hardware management.

---

## VM Series and Sizes

Azure offers hundreds of VM sizes organized into **series**, each optimized for a different type of workload. The series letter tells you the workload type; the number tells you how many vCPUs it has.

```mermaid
graph LR
    SIZES["Azure VM Sizes"]

    SIZES --> B["B-Series\nBurstable\n─────────────\nLow baseline CPU\nBursts when needed\nCheapest option\n─────────────\nB1s: 1 vCPU, 1 GB RAM\nB2s: 2 vCPU, 4 GB RAM\nB4ms: 4 vCPU, 16 GB RAM\n─────────────\nUse for: dev/test, light web\nservers, small apps"]

    SIZES --> D["D-Series\nGeneral Purpose\n─────────────\nBalanced CPU + memory\nMost common workload choice\n─────────────\nD2s v3: 2 vCPU, 8 GB RAM\nD4s v3: 4 vCPU, 16 GB RAM\nD8s v3: 8 vCPU, 32 GB RAM\n─────────────\nUse for: web apps, databases\nsmall-to-medium production"]

    SIZES --> F["F-Series\nCompute Optimized\n─────────────\nHigh CPU to memory ratio\nFastest per-core performance\n─────────────\nF2s v2: 2 vCPU, 4 GB RAM\nF4s v2: 4 vCPU, 8 GB RAM\n─────────────\nUse for: batch jobs, gaming\nservers, analytics"]

    SIZES --> E["E-Series\nMemory Optimized\n─────────────\nHigh memory to CPU ratio\n─────────────\nE2s v3: 2 vCPU, 16 GB RAM\nE4s v3: 4 vCPU, 32 GB RAM\nE8s v3: 8 vCPU, 64 GB RAM\n─────────────\nUse for: in-memory databases\nSAP HANA, Redis cache"]
```

**Other series worth knowing:**

| Series | Optimized for | Example use case |
|--------|--------------|-----------------|
| **N-series** | GPU workloads | Machine learning training, video rendering |
| **H-series** | High Performance Compute | Scientific simulations, CFD, molecular dynamics |
| **M-series** | Massive memory | SAP HANA up to 4 TB RAM |
| **L-series** | Storage optimized | NoSQL databases, high disk I/O |

**Understanding the size name — Standard_D2s_v3:**

| Part | Meaning |
|------|---------|
| `Standard` | Pricing tier (Standard vs Basic) |
| `D` | Series (General Purpose) |
| `2` | vCPU count |
| `s` | Supports Premium SSD storage |
| `_v3` | Hardware generation (newer = better performance/price) |

!!! info "Free Tier in this course: Standard_B1s"
    The Azure Free Tier includes **750 hours per month of B1s compute for 12 months** — that's enough to run a B1s VM around the clock for an entire year at zero cost. Every demo step today uses the B1s and is fully free.

!!! tip "How to pick a size"
    Start small — B2s or D2s — and use Azure Advisor's **right-sizing recommendations** to tell you if your VM is consistently under- or over-utilized. It's easier to resize up than to guess up front.

---

## VM Pricing Models

The same VM size can be priced three different ways depending on your commitment level.

```mermaid
graph TD
    subgraph PAYG["Pay-As-You-Go\n─────────────────────────\nCharged per second\nNo commitment\nStart/stop anytime\nHighest per-hour rate\n─────────────────────────\nBest for: unpredictable workloads\nlearning, dev/test"]
    end

    subgraph RI["Reserved Instances\n─────────────────────────\n1-year: ~30–40% cheaper\n3-year: ~50–60% cheaper\nCommit to a specific size + region\nPay upfront or monthly\n─────────────────────────\nBest for: stable production\nworkloads you run 24/7"]
    end

    subgraph SPOT["Spot VMs\n─────────────────────────\nUp to 90% cheaper than PAYG\nUse spare Azure capacity\nAzure can evict with 30s notice\nVM gets a shutdown signal\nPrice fluctuates with demand\n─────────────────────────\nBest for: batch jobs, rendering\nfault-tolerant, interruptible work"]
    end
```

**Pricing comparison example — Standard_D2s_v3 in East US:**

| Pricing Model | Monthly cost (approx.) | Savings vs PAYG |
|--------------|----------------------|-----------------|
| Pay-As-You-Go | ~$70/month | baseline |
| 1-Year Reserved | ~$44/month | ~37% cheaper |
| 3-Year Reserved | ~$30/month | ~57% cheaper |
| Spot (when available) | ~$7–$15/month | up to 90% cheaper |

!!! warning "Spot VMs are not for everything"
    If Azure needs that capacity back, your Spot VM gets a 30-second eviction notice and then shuts down. This is fine for batch processing or rendering jobs that save their state. It is not suitable for web servers, databases, or anything that needs to be always available.

!!! tip "Azure Hybrid Benefit"
    If your organization already owns Windows Server or SQL Server licenses (with Software Assurance), you can bring those licenses to Azure and avoid paying for the OS layer — typically saving another 30–40% on top of Reserved pricing. This is called **Azure Hybrid Benefit** and is configured when creating the VM.

---

## OS Options and the Azure Marketplace

When you create a VM, you choose an **image** — a pre-built snapshot of an operating system ready to boot. Azure provides hundreds of images through the **Azure Marketplace**.

**First-party images (Microsoft-published):**

| Category | Options |
|----------|---------|
| **Windows Server** | Windows Server 2019, 2022, Windows 10/11 (Dev/Test) |
| **Linux** | Ubuntu 20.04 / 22.04 / 24.04 LTS, Debian 11/12, SUSE SLES |
| **Enterprise Linux** | Red Hat Enterprise Linux 8/9 (licensed), Oracle Linux |

**Third-party Marketplace images:**

| Publisher | Images available |
|-----------|----------------|
| Canonical | Ubuntu Pro (enterprise support) |
| Bitnami | WordPress, LAMP, Nginx, Node.js, MongoDB — pre-configured stacks |
| JetBrains | TeamCity CI/CD server |
| CIS | Hardened OS images meeting CIS benchmarks |
| Cisco, Palo Alto | Network virtual appliances |

**Generation 1 vs Generation 2:**
- **Gen 1**: Legacy BIOS, wider compatibility
- **Gen 2**: UEFI boot, supports larger disks (>2 TB OS disk), faster boot, required for some features like Trusted Launch
- For new VMs, always use Gen 2 when the image supports it

!!! tip "Image versioning matters"
    Ubuntu 22.04 LTS has multiple minor versions. The Marketplace image labeled "Ubuntu Server 22.04 LTS" always gives you the latest minor release. If you need a pinned version for reproducibility, select it explicitly in the image details.

---

## Managed Disks

Every VM needs storage. Azure storage for VMs comes in two categories:

```mermaid
graph TD
    subgraph VM["Your Virtual Machine"]
        OS["💿 OS Disk\n─────────────────────────\nWhere the operating system lives\nMounts as C: (Windows) or / (Linux)\nRequired — cannot be removed\nTypically 30–128 GB\nDo NOT store your app data here"]

        DATA["📀 Data Disk\n─────────────────────────\nAdditional storage you attach\nMounts as D: / E: or /dev/sdc\nOptional — attach as many as needed\nGood for: databases, logs, file storage\nSurvives VM recreation if detached"]
    end

    TEMP["⚡ Temporary Disk\n─────────────────────────\nPhysical local SSD on the host\nExtremely fast — lowest latency\nMounts as D: (Windows) /dev/sdb (Linux)\nLost when VM is deallocated or resized\nNever store important data here"]

    VM --- TEMP
```

**Disk performance tiers:**

| Tier | Use case | IOPS (example) | Latency | Price |
|------|---------|---------------|---------|-------|
| **Standard HDD** | Backup, archive, infrequent access | ~500 IOPS | High | Lowest |
| **Standard SSD** | Light production, dev/test, web servers | ~500–6,000 IOPS | Medium | Low |
| **Premium SSD** | Production databases, high-throughput apps | ~7,500–80,000 IOPS | Low | Medium |
| **Ultra Disk** | SAP HANA, top-tier databases, HPC | Up to 160,000 IOPS | Sub-ms | Highest |

!!! info "Managed vs Unmanaged Disks"
    **Managed disks** are the modern standard — Azure creates and manages the underlying storage account for you. You just pick the size and tier. **Unmanaged disks** are legacy — you manage the storage account yourself. Always use managed disks for new VMs.

**Disk snapshots:**
A **snapshot** is a point-in-time read-only copy of a managed disk. You can create a snapshot before making risky changes, and restore from it if something goes wrong. Snapshots are billed by the size of changed data, not the full disk size.

---

## Public IP Addresses and DNS Labels

For your VM to be reachable from the internet, it needs a **Public IP address**.

```mermaid
graph LR
    YOU["👤 You\nyour-laptop.isp.com\n203.x.x.x"] -->|"SSH port 22"| PIP["🌐 Public IP\ne.g., 20.84.x.x\nAttached to VM's NIC"]
    PIP --> NIC["Network Interface Card (NIC)\nVM's network adapter"]
    NIC --> VM["Your Ubuntu VM\nPrivate IP: 10.0.0.4"]
```

**Dynamic vs Static Public IP:**

| Type | Behavior | Use case |
|------|---------|---------|
| **Dynamic** | IP changes every time the VM is deallocated and started again | Dev/test, short-lived VMs |
| **Static** | IP stays the same forever | Production, DNS records, whitelisted IPs |

!!! tip "DNS labels instead of IPs"
    Azure lets you assign a **DNS label** to a Public IP: `myvm.eastus.cloudapp.azure.com`. This hostname always resolves to your VM's current public IP — even if you use a dynamic IP and it changes. Much easier to use and remember than a raw IP.

---

## Network Security Groups (NSG)

A **Network Security Group** is the firewall for your VM. It contains rules that allow or deny traffic based on protocol, port, and source/destination address.

```mermaid
graph TD
    INTERNET["🌐 Internet\nAll traffic"]

    subgraph NSG["Network Security Group: my-vm-nsg"]
        R1["Rule 100\n✅ ALLOW\nSSH port 22\nfrom Any\npriority: 100"]
        R2["Rule 200\n✅ ALLOW\nHTTP port 80\nfrom Any\npriority: 200"]
        R3["Rule 65000\n✅ ALLOW\nAll traffic from VNet\n(default — always present)"]
        R4["Rule 65001\n✅ ALLOW\nAzure Load Balancer probe\n(default — always present)"]
        R5["Rule 65500\n❌ DENY\nAll inbound\n(default — blocks everything else)"]
    end

    VM["Your VM\nUbuntu B1s"]

    INTERNET --> R1 --> VM
    INTERNET --> R2 --> VM
    INTERNET --> R5
```

**Key NSG concepts:**

| Concept | Details |
|---------|---------|
| **Priority** | Lower number = evaluated first. Range: 100–4096. Lower wins if rules conflict. |
| **Direction** | Inbound (traffic coming in to the VM) or Outbound (traffic leaving the VM) |
| **Action** | Allow or Deny |
| **Source** | Any IP, a specific IP/CIDR range, a VNet, or a Service Tag (e.g., `Internet`, `AzureCloud`) |
| **Destination** | Same options as Source |
| **Protocol** | TCP, UDP, ICMP, or Any |

**Default inbound rules (always present, cannot be deleted):**

| Priority | Name | Action | What it does |
|----------|------|--------|-------------|
| 65000 | AllowVnetInBound | Allow | All traffic between resources in the same VNet |
| 65001 | AllowAzureLoadBalancerInBound | Allow | Health probes from Azure Load Balancer |
| 65500 | DenyAllInBound | Deny | Blocks everything not explicitly allowed above |

!!! tip "NSG can be attached to a subnet or a NIC"
    When attached to a **subnet**, it applies to every VM in that subnet. When attached to a **NIC** (network interface), it applies to one specific VM. You can use both — traffic is evaluated at the subnet NSG first, then the NIC NSG.

---

## Demo: Create and Connect to Your First VM

All steps below are **✅ Free Tier** — you can follow every single one on an Azure Free or Pay-As-You-Go account.

---

### Part A — Create a Resource Group

!!! success "Step 1 — Open Resource Groups"
    In the Azure Portal search bar, type **"Resource groups"** and click the result.

!!! success "Step 2 — Create a dedicated resource group for this demo"
    Click **"+ Create"** and fill in:

    | Field | Value |
    |-------|-------|
    | Subscription | *(your subscription)* |
    | Resource group name | `vm-demo-rg` |
    | Region | *(closest to you — e.g., East US, Australia East, West Europe)* |

    Click **"Review + create"** → **"Create"**.

    > **Why a separate resource group?** Everything we create today — the VM, its disk, its public IP, its NSG, its virtual network — will live here. When we're done, deleting this one resource group cleans everything up instantly.

---

### Part B — Create an Ubuntu VM

!!! success "Step 3 — Start the VM creation wizard"
    In the Azure Portal search bar, type **"Virtual machines"** and click the result. Click **"+ Create"** → **"Azure virtual machine."**

    The creation wizard has 7 tabs. We'll walk through each one.

---

#### Basics Tab

!!! success "Step 4 — Project details"
    | Field | Value |
    |-------|-------|
    | Subscription | *(your subscription)* |
    | Resource group | `vm-demo-rg` |

!!! success "Step 5 — Instance details"
    | Field | Value |
    |-------|-------|
    | Virtual machine name | `ubuntu-demo-vm01` |
    | Region | *(same region you used for the resource group)* |
    | Availability options | **No infrastructure redundancy required** |
    | Security type | **Standard** |
    | Image | **Ubuntu Server 24.04 LTS — x64 Gen2** |
    | VM architecture | **x64** |
    | Run with Azure Spot discount | *(leave unchecked)* |
    | Size | Click **"See all sizes"** → filter by **B-series** → select **B1s (1 vCPU, 1 GiB RAM)** → click **Select** |

    > **Why B1s?** It's the size included in the Azure Free Tier — 750 hours per month, free for 12 months. Perfect for learning.

    > **Why Ubuntu 24.04 LTS?** LTS stands for Long-Term Support — this version receives security updates until 2034. It's the industry-standard choice for cloud Linux servers.

!!! success "Step 6 — Administrator account"
    | Field | Value |
    |-------|-------|
    | Authentication type | **SSH public key** |
    | Username | `azureuser` |
    | SSH public key source | **Generate new key pair** |
    | Key pair name | `ubuntu-demo-vm01_key` |

    > **SSH key vs password:** SSH public key authentication is more secure than a password. Azure generates a key pair — the private key goes to your laptop, the public key goes to the VM. Only someone with the private key file can log in.

!!! success "Step 7 — Inbound port rules"
    | Field | Value |
    |-------|-------|
    | Public inbound ports | **Allow selected ports** |
    | Select inbound ports | **SSH (22)** |

    This creates an NSG rule that allows SSH traffic from any IP. We'll inspect this NSG in detail after the VM is created.

    Click **"Next: Disks >"**

---

#### Disks Tab

!!! success "Step 8 — OS disk configuration"
    | Field | Value |
    |-------|-------|
    | OS disk size | **Default size (30 GiB)** |
    | OS disk type | **Standard SSD (locally redundant storage)** |
    | Delete with VM | ✅ Checked |

    > **OS disk type:** We're using Standard SSD instead of Premium SSD to stay within free tier limits. Premium SSD offers lower latency and higher IOPS — we'll use it for production databases later in the course.

    > **Delete with VM:** Checking this means the OS disk is automatically deleted when you delete the VM. If unchecked, the disk (and its cost) persists even after the VM is gone.

!!! success "Step 9 — Add a Data Disk"
    Click **"+ Create and attach a new disk."** Configure:

    | Field | Value |
    |-------|-------|
    | Name | `ubuntu-demo-vm01-data01` |
    | Source type | **None (empty disk)** |
    | Size | Click **"Change size"** → select **4 GiB, Standard HDD** → **OK** |
    | Delete disk with virtual machine | ✅ Checked |

    Click **"OK."**

    > We're adding a tiny data disk just to see how the concept works. In production, your application data, logs, and databases live on separate data disks — never on the OS disk.

    Click **"Next: Networking >"**

---

#### Networking Tab

!!! success "Step 10 — Virtual network and subnet"
    Azure will automatically create a new Virtual Network and subnet for you since this is your first VM in this resource group.

    | Field | Value |
    |-------|-------|
    | Virtual network | *(auto-generated: `ubuntu-demo-vm01-vnet`)* |
    | Subnet | *(auto-generated: `default 10.0.0.0/24`)* |
    | Public IP | *(auto-generated: `ubuntu-demo-vm01-ip`)* |

    > We'll explore Virtual Networks in detail on Day 6. For now, Azure's defaults get us connected.

!!! success "Step 11 — Network Security Group"
    | Field | Value |
    |-------|-------|
    | NIC network security group | **Basic** |
    | Public inbound ports | **Allow selected ports** |
    | Select inbound ports | **SSH (22)** |

    > This creates an NSG named `ubuntu-demo-vm01-nsg` with one inbound rule: allow TCP port 22 from any source.

!!! success "Step 12 — Load balancing (skip)"
    | Field | Value |
    |-------|-------|
    | Load balancing options | **None** |

    We'll set up load balancing on Day 7.

    Click **"Next: Management >"**

---

#### Management Tab

!!! success "Step 13 — Auto-shutdown"
    Scroll to **"Auto-shutdown."** Enable it:

    | Field | Value |
    |-------|-------|
    | Enable auto-shutdown | ✅ On |
    | Shutdown time | **11:00 PM** (or your local evening time) |
    | Time zone | *(your local time zone)* |
    | Send notification before shutdown | ✅ On |
    | Notification email | *(your email address)* |

    > **This is important for cost control.** Auto-shutdown automatically deallocates your VM each night so you're not paying for compute while you sleep. You can manually start it again the next day.

    Leave everything else on this tab at defaults.

    Click **"Next: Monitoring >"**

---

#### Monitoring Tab

!!! success "Step 14 — Diagnostics"
    | Field | Value |
    |-------|-------|
    | Boot diagnostics | **Enable with managed storage account (recommended)** |
    | OS guest diagnostics | Off *(leave as default)* |

    > **Boot diagnostics** captures a screenshot of your VM's console during startup and stores it in a storage account. If your VM ever fails to boot, this screenshot is how you diagnose what went wrong — you'd otherwise have no visibility into a headless cloud VM.

    Click **"Next: Advanced >"**

---

#### Advanced Tab

Leave all settings at defaults. This tab covers cloud-init scripts, user data, and proximity placement groups — topics for later in the course.

Click **"Next: Tags >"**

---

#### Tags Tab

!!! success "Step 15 — Apply tags"
    | Name | Value |
    |------|-------|
    | Environment | Learning |
    | Owner | *(your name)* |
    | Project | AzureCourse |
    | Day | Day02 |

    Click **"Next: Review + create >"**

---

#### Review + Create

!!! success "Step 16 — Review and deploy"
    Azure validates all your settings. Review the summary — confirm:
    - VM size: **Standard_B1s**
    - Image: **Ubuntu Server 24.04 LTS**
    - OS disk: **Standard SSD, 30 GiB**
    - Data disk: **Standard HDD, 4 GiB**
    - Region: *(your region)*

    Click **"Create."**

!!! success "Step 17 — Download the private key"
    A dialog appears: **"Generate new key pair."** Click **"Download private key and create resource."**

    Your browser downloads `ubuntu-demo-vm01_key.pem`. **Move this file to a safe location on your laptop** — you'll need it to SSH into your VM. On Linux/macOS, move it to `~/.ssh/`. On Windows, move it to `C:\Users\YourName\.ssh\`.

    Deployment takes 1–3 minutes. When complete, click **"Go to resource."**

---

### Part C — Explore Your VM in the Portal

!!! success "Step 18 — Overview page"
    The VM Overview page shows:
    - **Status:** Running
    - **Public IP address:** Note this down — you'll use it to SSH in
    - **Private IP address:** The VM's internal network address (10.0.0.x)
    - **Operating system:** Linux (Ubuntu 24.04)
    - **Size:** Standard_B1s
    - **Subscription**, **Resource group**, **Location**

!!! success "Step 19 — Explore the Disks section"
    In the left-hand menu, click **"Disks."** You'll see:
    - The **OS disk** (`ubuntu-demo-vm01_disk1_...`) — 30 GiB Standard SSD
    - The **Data disk** (`ubuntu-demo-vm01-data01`) — 4 GiB Standard HDD

    Click on the OS disk to open its own blade. Notice it has its own Overview, Snapshots, Access control, and more — disks are first-class Azure resources.

    Click the back arrow to return to your VM.

!!! success "Step 20 — Explore the Networking section"
    In the left-hand menu, click **"Networking."** You'll see:
    - The **Network Interface Card (NIC)** — your VM's network adapter
    - The **NSG rules** — the one SSH rule you configured is listed here
    - The **Inbound port rules** and **Outbound port rules** tabs

    Click **"ubuntu-demo-vm01-nsg"** to open the NSG blade directly. You can see the full list of inbound rules — your port 22 rule at priority 300, and the three Azure default rules at 65000, 65001, and 65500.

---

### Part D — Connect to the VM via SSH

!!! success "Step 21 — Open a terminal"
    On **Windows:** Open Windows Terminal or PowerShell. On **macOS/Linux:** Open Terminal.

    Alternatively, use **Azure Cloud Shell** (click the `>_` icon in the Azure Portal) — it already has SSH installed and you can upload your key file.

!!! success "Step 22 — Set permissions on the private key (macOS/Linux only)"
    SSH requires that your private key file is readable only by you:

    ```bash
    chmod 400 ~/.ssh/ubuntu-demo-vm01_key.pem
    ```

    On Windows, the .pem file permissions are managed differently — Windows Terminal and PowerShell handle this automatically.

!!! success "Step 23 — Connect via SSH"
    Replace `YOUR_PUBLIC_IP` with the IP address from your VM's Overview page:

    ```bash
    ssh -i ~/.ssh/ubuntu-demo-vm01_key.pem azureuser@YOUR_PUBLIC_IP
    ```

    On Windows with PowerShell:
    ```powershell
    ssh -i C:\Users\YourName\.ssh\ubuntu-demo-vm01_key.pem azureuser@YOUR_PUBLIC_IP
    ```

    When prompted "Are you sure you want to continue connecting (yes/no)?" type **yes** and press Enter.

    You're now inside your Ubuntu VM in Microsoft's data center.

!!! success "Step 24 — Explore the VM from inside"
    Run these commands to confirm what you're running:

    ```bash
    # Check the OS version
    lsb_release -a
    ```
    ```bash
    # Check CPU and memory
    nproc && free -h
    ```
    ```bash
    # Check disk layout — you'll see the OS disk and the data disk
    lsblk
    ```
    ```bash
    # Check your public internet address (should match the VM's public IP)
    curl ifconfig.me
    ```
    ```bash
    # Check uptime
    uptime
    ```

    You're looking at a real Ubuntu server, running live in Azure. One vCPU, 1 GB RAM — small, but fully functional.

!!! success "Step 25 — Format and mount the data disk"
    The data disk is attached but not yet formatted or mounted. Let's make it usable:

    ```bash
    # List block devices — find the data disk (usually /dev/sdc)
    lsblk
    ```
    ```bash
    # Format the data disk with ext4 filesystem
    sudo mkfs.ext4 /dev/sdc
    ```
    ```bash
    # Create a mount point
    sudo mkdir /data
    ```
    ```bash
    # Mount the disk
    sudo mount /dev/sdc /data
    ```
    ```bash
    # Verify it's mounted
    df -h /data
    ```

    You now have a second disk mounted at `/data`. In a real workload, this is where you'd store your application data, database files, or logs — keeping them separate from the OS disk.

!!! success "Step 26 — Exit the SSH session"
    ```bash
    exit
    ```

    You're back on your local machine. The VM continues running in Azure.

---

### Part E — The Critical Difference: Stop vs Deallocate

This is one of the most important cost concepts for VMs — and one of the most common mistakes beginners make.

!!! success "Step 27 — Understand the VM states"

    ```mermaid
    graph LR
        RUNNING["▶️ Running\n─────────────────────\nCompute: ✅ Billed\nOS disk: ✅ Billed\nPublic IP: ✅ Billed\nData in memory: ✅ Preserved"]

        STOPPED["⏹️ Stopped (OS-level)\n─────────────────────\nTriggered by: shutdown inside the OS\nor Stop button in portal\n─────────────────────\nCompute: ✅ STILL BILLED\nOS disk: ✅ Billed\nPublic IP: ✅ Billed\nData in memory: ❌ Lost"]

        DEALLOCATED["💤 Deallocated\n─────────────────────\nTriggered by: Stop (Deallocate) in portal\nor auto-shutdown\n─────────────────────\nCompute: ❌ NOT BILLED\nOS disk: ✅ Billed (storage only)\nPublic IP: ✅ Billed if static\nData in memory: ❌ Lost"]

        RUNNING -->|"Shut down from inside OS\nor Stop in portal"| STOPPED
        RUNNING -->|"Stop (Deallocate) in portal\nor auto-shutdown"| DEALLOCATED
        STOPPED -->|"Azure still holds compute\nresources reserved for you"| STOPPED
        DEALLOCATED -->|"Start button"| RUNNING
    ```

    **The key insight:** When you shut down the OS from inside the VM (`sudo shutdown now`), or press "Stop" in the portal, Azure keeps the compute capacity **reserved** for you. You're still paying for the vCPU and RAM even though the VM isn't running.

    **Deallocating** releases that reserved compute back to Azure's pool. You stop paying for compute. You pay only for disk storage (a few cents per GB per month) until you start the VM again.

!!! success "Step 28 — Deallocate your VM"
    In the Azure Portal on your VM's Overview page, click the **"Stop"** button in the top toolbar.

    A dialog appears with a checkbox: **"Do you want to reserve the public IP address?"** — leave it unchecked (we're using a dynamic IP, so it doesn't matter).

    Click **"OK."**

    The VM status changes from **Running** → **Stopping** → **Stopped (deallocated).**

    Notice the Public IP address disappears from the Overview. If you had a dynamic IP, it's been released. When you start the VM again, it will get a new IP.

!!! success "Step 29 — Confirm the state change"
    In the left-hand menu, click **"Overview."** The status reads **"Stopped (deallocated)."** The Start button is now available.

    The compute charge for this VM has stopped. You're now only paying for the two managed disks — a few cents per month for the Standard HDD and Standard SSD.

!!! success "Step 30 — Start the VM again (optional)"
    Click the **"Start"** button. The VM goes from Stopped → Starting → Running in about 30–60 seconds. Note that the Public IP address has changed (if dynamic) — this is the tradeoff of not using a static IP.

---

### Part F — Inspect the NSG Rules

!!! success "Step 31 — Navigate to the NSG"
    In the Azure Portal search bar, type **"Network security groups"** and click the result. Click **"ubuntu-demo-vm01-nsg."**

!!! success "Step 32 — Review inbound rules"
    Click **"Inbound security rules"** in the left menu. You'll see:

    | Priority | Name | Port | Protocol | Source | Destination | Action |
    |----------|------|------|---------|--------|-------------|--------|
    | 300 | SSH | 22 | TCP | Any | Any | Allow |
    | 65000 | AllowVnetInBound | Any | Any | VirtualNetwork | VirtualNetwork | Allow |
    | 65001 | AllowAzureLoadBalancerInBound | Any | Any | AzureLoadBalancer | Any | Allow |
    | 65500 | DenyAllInBound | Any | Any | Any | Any | Deny |

!!! success "Step 33 — Add a rule to allow HTTP (optional)"
    Click **"+ Add"** to add a new inbound rule:

    | Field | Value |
    |-------|-------|
    | Source | **Any** |
    | Source port ranges | `*` |
    | Destination | **Any** |
    | Service | **HTTP** |
    | Destination port ranges | `80` (auto-filled) |
    | Protocol | **TCP** (auto-filled) |
    | Action | **Allow** |
    | Priority | `310` |
    | Name | `HTTP` |

    Click **"Add."**

    > We're not running a web server on this VM yet — but this is exactly how you'd open port 80 when you install one. Add the NSG rule first, then install the software.

---

## Summary

```mermaid
graph TD
    VM["Azure Virtual Machine\nRenting compute in Microsoft's DC"]
    VM --> SERIES["VM Series\nB · D · F · E · N · H · M"]
    VM --> PRICING["Pricing Models\nPAYG · Reserved · Spot"]
    VM --> OS["OS + Marketplace\nUbuntu · Windows Server\nRHEL · Custom Images"]
    VM --> DISKS["Managed Disks\nOS Disk + Data Disks\nHDD → SSD → Premium → Ultra"]
    VM --> NETWORK["Networking\nPublic IP · DNS Label\nNSG Rules"]
    VM --> STATES["VM States\nRunning → Stopped → Deallocated\nOnly deallocated stops compute billing"]
```

You've gone from zero to a live Ubuntu server in Microsoft's data center — created, configured, connected to over SSH, explored from the inside, and shut down correctly. These are the fundamental mechanics you'll use for every VM in this course.

**What's next on Day 3:** We go deeper into VMs — Availability Sets, Availability Zones, VM Scale Sets, VM backups, Azure Bastion, and custom script extensions. Same service, much more power.

---

## Key Takeaways

| Concept | What to remember |
|---------|-----------------|
| Virtual Machines | Software-defined computers in Microsoft's data centers — you control the OS and software; Azure controls the hardware |
| B-series (B1s) | Burstable, cheapest series — perfect for dev/test; Free Tier includes 750 hrs/month |
| D-series | General purpose — balanced CPU/memory — most common choice for production |
| Pay-as-you-go | Default pricing — charged per second, no commitment, most expensive per hour |
| Reserved Instances | 1-year saves ~37%; 3-year saves ~57% — commit when workloads are stable and predictable |
| Spot VMs | Up to 90% cheaper — evicted with 30s notice — batch and fault-tolerant workloads only |
| OS Disk | Where the OS lives — never store app data here |
| Data Disks | Separate storage for app data, databases, logs — survives VM recreation if detached first |
| Standard SSD vs Premium SSD | Standard SSD for dev/test; Premium SSD for production databases (required for highest SLA) |
| Public IP | Dynamic (changes on deallocate) or Static (fixed, slightly higher cost) — use DNS labels to abstract IP changes |
| NSG | Layer 4 firewall — lower priority number = higher precedence — default deny-all protects your VM unless you explicitly allow traffic |
| Stopped ≠ Deallocated | Stopped VM still bills for compute — always use Stop (Deallocate) or auto-shutdown to truly stop the charge |
| Boot diagnostics | Enable it — your only way to see what a headless VM is doing if it fails to boot |
| Auto-shutdown | Enable it on every learning VM — automatically deallocates nightly so you're never accidentally billed overnight |
| SSH key auth | More secure than passwords — download and safeguard your .pem file; it cannot be recovered after creation |

---

[:material-arrow-left: Previous: Day 1 — Azure Fundamentals](day01_fundamentals.md) &nbsp;&nbsp; [:material-arrow-right: Next: Day 3 — Virtual Machines Part 2](day03_vms_part2.md)
