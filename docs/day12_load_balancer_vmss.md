# Day 12 — Load Balancer, VM Scale Sets & Application Gateway: Distributing Traffic at Scale

**Phase 3 — Networking**

> Every VM you've deployed so far in this course has been a single box. It has one public IP, it does one job, and if it goes down — or simply gets overwhelmed by traffic — your application goes down with it. That's fine for a lab. It's not fine for anything real. Today we fix that problem from three different angles: **Azure Load Balancer**, which spreads incoming traffic across multiple VMs at the network layer so no single machine is a bottleneck or a single point of failure; **VM Scale Sets (VMSS)**, which let Azure automatically add and remove VMs based on actual demand instead of you guessing capacity up front; and **Application Gateway**, which does the same job as Load Balancer but one layer up — understanding HTTP itself, so it can route by URL path, terminate SSL, and host multiple sites behind one IP. Together, these three are the standard pattern behind almost every scalable compute workload in Azure — and a heavily tested area on both the **AZ-104 (Administrator)** and **AZ-305 (Solutions Architect)** exams, so we'll call out exam-relevant details as we go. As with Day 11, today's lab builds its own VNet and resources from scratch so it's fully self-contained.

---

## What You'll Learn

- **Azure Load Balancer** — Layer 4 (TCP/UDP) traffic distribution: frontend IPs, backend pools, health probes, load balancing rules, and inbound NAT rules
- Hands-on: build a resource group and VNet, deploy two plain VMs by hand, and wire up a Standard Load Balancer in front of them so you understand every moving part before automation takes over
- Public vs Internal Load Balancers, and SKU differences (Basic vs Standard — and why Basic no longer exists as an option)
- **VM Scale Sets (VMSS)** — managing a fleet of identical VMs as a single resource, Uniform vs Flexible orchestration modes
- Hands-on: build a VMSS with an integrated Standard Load Balancer, install a web server fleet-wide with a custom script extension, and prove traffic is actually being distributed
- **Autoscale** — scale-out and scale-in rules driven by real metrics, and watching a scale event happen live
- **Application Gateway** — Layer 7 (HTTP/HTTPS) routing: listeners, routing rules, URL path-based routing, multi-site hosting, and SSL termination
- Hands-on: build an Application Gateway v2 with two backend pools and a path-based routing rule, and prove two different sites are served from one public IP
- How exam questions frame Load Balancer vs Application Gateway, and the decision points the AZ-104/AZ-305 exams like to test

---

## Before We Begin

A fully paid lab today — Load Balancer, VM Scale Set instances, and Application Gateway all carry a cost. We'll delete everything at the end.

- **Standard Load Balancer**: ~$0.025/hour for the load balancer resource itself, plus ~$0.005/hour per load balancing rule and a small per-GB data processing charge. **💳 Paid**, but only a few cents for a short lab session.
- **VM Scale Set instances** on `Standard_B1s`: same billing as any VM. The Free Tier's 750 B-series hours/month can cover a couple of instances for a short demo, but running several VMSS instances simultaneously alongside other VMs you may have left running elsewhere can exceed that allowance fast — budget for **💳 small paid usage** today.
- **Application Gateway v2**: billed per gateway-hour (~$0.246/hour for Standard_v2) plus capacity units consumed. There's no way around this cost to even create one — **💳 instructor demo**, delete immediately after.
- **Public IP addresses**: a Standard SKU public IP costs a small hourly amount once it's no longer attached to a running resource — we'll clean these up at the end.
- **NAT Gateway**: ~$0.045/hour plus a small per-GB data processing charge — we deploy one briefly in Part 1 purely so our backend VMs can reach the internet during setup. **💳 Paid**, deleted with everything else at the end.
- Everything else (the VNet, subnets, NSGs, autoscale rules themselves) is **✅ free**.

> **A note on Basic Load Balancer:** if you've seen older tutorials mention a free "Basic" SKU, it no longer exists as a deployment option — Microsoft retired Basic Load Balancer on **September 30, 2025**. Every Load Balancer you create today, and from now on, is **Standard SKU**. This matters for the exam: Standard SKU is secure by default (closed unless you explicitly allow traffic via an NSG) and requires VMs in its backend pool to have Standard SKU public IPs or no public IP at all — mixing Basic and Standard resources together simply isn't possible anymore.

> **A note on private subnets by default:** since **March 31, 2026**, every new subnet defaults to **no outbound internet access** — removing a VM's public IP (as we do for both backend VMs today) removes its outbound path too, not just its inbound exposure, unless you explicitly grant one (a public IP, a NAT Gateway, or a Load Balancer outbound rule — see Day 11). This matters today specifically because of *when* it bites: the cloud-init script we use to install nginx runs once, at first boot — and if the VM has no outbound path at that moment, the install fails silently and you're left with a health probe that never turns healthy, with nothing in the Load Balancer's own configuration to explain why. That's why Part 1 deploys a small NAT Gateway **before** the VMs, not after.

---

## Part 1 — Why You Need a Load Balancer

### The Problem: One VM Is a Single Point of Failure

Imagine you've built a web app on a single VM. Two things will eventually go wrong:

1. **Traffic outgrows the VM.** One `Standard_B1s` VM can only serve so many concurrent requests before response times degrade or the VM falls over entirely.
2. **The VM fails.** Hardware issues, OS crashes, a bad deployment — when there's only one VM, any failure is a full outage.

The fix for both is the same: **run more than one VM**, and put something in front of them that spreads traffic across all of them and stops sending traffic to any VM that's unhealthy. That "something" is **Azure Load Balancer**.

### Azure Load Balancer — Layer 4 Traffic Distribution

Azure Load Balancer operates at **Layer 4** of the OSI model — it works with TCP and UDP, routing based on IP addresses and ports. It has no awareness of HTTP, cookies, URL paths, or anything above the transport layer. (That's the job of **Application Gateway**, which we'll build later today in Part 3 — Layer 4 vs Layer 7 is exactly the line between the two services, and a favorite distinction on the exams.)

**How traffic gets distributed:** Azure Load Balancer uses a **5-tuple hash** to decide which backend VM gets a given connection:

```
(Source IP, Source Port, Destination IP, Destination Port, Protocol)
```

Because the hash is calculated per-connection, the same client will usually land on the same backend VM for the duration of one TCP connection, but a new connection (even from the same client) can land on a different VM. There's no central "traffic cop" making smart decisions — it's a deterministic hash, which is why Load Balancer can operate at such high throughput with very low latency.

### The Building Blocks

Every Azure Load Balancer is made of four pieces:

| Component | What it does |
|---|---|
| **Frontend IP configuration** | The IP address clients connect to — public (internet-facing) or private (internal to a VNet) |
| **Backend pool** | The group of VMs (or VMSS instances) that receive distributed traffic |
| **Health probe** | A periodic check (TCP or HTTP/HTTPS) against each backend VM; any VM that fails the probe is automatically pulled out of rotation until it passes again |
| **Load balancing rule** | Ties it together — "traffic arriving on frontend port X, protocol Y, gets sent to backend port Z, using health probe P" |

There's a fifth, optional piece:

| Component | What it does |
|---|---|
| **Inbound NAT rule** | Forwards a specific frontend port to a specific *individual* backend VM — used for direct management access (e.g., SSH to VM #2 specifically), bypassing the load-balanced pool entirely |

```mermaid
graph TD
    Client["Client"]
    FE["Frontend IP\n(Public, port 80)"]
    LBR["Load Balancing Rule"]
    Probe["Health Probe\n(HTTP :80/)"]
    Pool["Backend Pool"]
    VM1["VM 1"]
    VM2["VM 2"]

    Client --> FE --> LBR --> Pool
    Pool --> VM1
    Pool --> VM2
    Probe -.->|"checks health"| VM1
    Probe -.->|"checks health"| VM2
```

### Public vs Internal Load Balancer

- A **Public Load Balancer** has a public frontend IP — it's how internet traffic reaches your backend VMs. This is what we're building today.
- An **Internal Load Balancer (ILB)** has a private frontend IP from inside your VNet — used to distribute traffic *between tiers*, e.g. a web tier load-balancing requests to an internal app tier, with no public exposure at all.

> **Exam tip:** if a scenario describes load-balancing traffic that should never touch the public internet — e.g., between an app tier and a database tier inside the same VNet — that's an Internal Load Balancer, not a Public one. Same component, different frontend IP type.

### Session Persistence

By default, Load Balancer has **no session persistence (None)** — each new connection can land anywhere, based on the 5-tuple hash. You can also configure **Client IP** affinity (also called source IP affinity), which forces all connections from the same client IP to the same backend VM. This matters for applications that store session state locally on each VM instead of in a shared store like Redis — without persistence, a stateful app could send a user's second request to a different VM that knows nothing about their session.

---

### Hands-On: Build the Lab Network and Two Plain VMs

**✅ Free Tier — networking and the VMs themselves are within free-tier limits**

Before we let VM Scale Sets automate everything, let's build a Load Balancer by hand in front of two ordinary VMs — this is the only way to actually see every component (frontend IP, backend pool, health probe, rule) as a distinct, individually-configured piece, the way the exam expects you to understand them.

**Step 1 — Create the resource group and VNet:**

1. Search for **Resource groups** → **+ Create** → name it `rg-day12-demo`, region **East US** → **Create**.
2. Search for **Virtual networks** → **+ Create**.
   - **Resource group:** `rg-day12-demo`
   - **Name:** `vnet-day12`
   - **Region:** East US
3. **Next: IP Addresses** → set the address space to `10.0.0.0/16`, replace the default subnet with **subnet-app** → `10.0.1.0/24`.
4. **Review + create** → **Create**.

**Step 2 — Create an NSG allowing HTTP and SSH, attach it to `subnet-app`:**

1. Search for **Network security groups** → **+ Create** → **Resource group:** `rg-day12-demo`, **Name:** `nsg-day12`, region East US → **Create**.
2. Open `nsg-day12` → **Inbound security rules** → **+ Add** → **Service:** HTTP, **Priority:** 100, **Name:** `Allow-HTTP` → **Add**.
3. Add a second rule: **Service:** SSH, **Priority:** 110, **Name:** `Allow-SSH` → **Add**.
4. Go to **Subnets** → **Associate** → **Virtual network:** `vnet-day12`, **Subnet:** `subnet-app` → **OK**.

**Step 3 — Deploy a NAT Gateway so `subnet-app` has outbound internet access:**

Our backend VMs in Step 4 get **no public IP** — by design, they should only be reachable through the Load Balancer. But with private subnets now the default (see the note above), that also means *zero outbound* access unless we explicitly grant one. We need that outbound path to exist **before** the VMs boot, so cloud-init can actually install nginx — a NAT Gateway is the cleanest way to give a whole subnet that path without putting a public IP on any individual VM.

1. Search for **Public IP addresses** → **+ Create** → **Name:** `pip-natgw-day12`, **SKU:** Standard, **Resource group:** `rg-day12-demo`, region East US → **Create**.
2. Search for **NAT gateways** → **+ Create**.
   - **Resource group:** `rg-day12-demo`
   - **Name:** `natgw-day12`
   - **Region:** East US
3. On **Outbound IP** → select `pip-natgw-day12`.
4. On **Subnet** → **Virtual network:** `vnet-day12`, check **subnet-app**.
5. **Review + create** → **Create**.

> **Exam tip:** NAT Gateway and Standard Load Balancer outbound rules can't coexist on the same subnet — pick one mechanism. We're using NAT Gateway here because it has to exist *before* the VMs boot (for cloud-init to succeed), while a Load Balancer outbound rule can only be configured *after* the Load Balancer and its backend pool already exist — wrong order for what we need today.

**Step 4 — Deploy two plain VMs (no public IP — they'll only be reachable through the Load Balancer):**

1. Search for **Virtual machines** → **+ Create** → **Azure virtual machine**.
   - **Resource group:** `rg-day12-demo`
   - **VM name:** `vm-web-1`
   - **Region:** East US
   - **Image:** Ubuntu Server 24.04 LTS
   - **Size:** Standard_B1s
   - **Authentication:** SSH public key (or password)
2. On **Networking**: **Virtual network:** `vnet-day12`, **Subnet:** `subnet-app`, **Public IP:** **None** (these VMs will only be reachable through the Load Balancer we're about to build), **NIC network security group:** None.
3. On **Advanced**, scroll to **Custom data** and paste this cloud-init script so the VM is web-serving the moment it boots:
   ```yaml
   #cloud-config
   package_update: true
   packages:
     - nginx
   runcmd:
     - echo "Hello from $(hostname)" > /var/www/html/index.html
   ```
4. **Review + create** → **Create**.
5. Repeat for a second VM named `vm-web-2` — same settings, same custom data (its `$(hostname)` will print `vm-web-2` instead, so you can tell them apart later).

**Step 5 — Build the Standard Load Balancer:**

1. Search for **Load balancers** → **+ Create**.
   - **Resource group:** `rg-day12-demo`
   - **Name:** `lb-day12`
   - **Region:** East US
   - **SKU:** Standard
   - **Type:** Public
2. On **Frontend IP configuration** → **+ Add a frontend IP**:
   - **Name:** `fe-day12`
   - **Public IP address:** Create new → name `pip-lb-day12`, SKU Standard
3. On **Backend pools** → **+ Add a backend pool**:
   - **Name:** `bep-day12`
   - **Virtual network:** `vnet-day12`
   - Under **IP version: IPv4**, click **+ Add** and select both `vm-web-1` and `vm-web-2`
4. On **Inbound rules** → **+ Add a load balancing rule**:
   - **Name:** `lbr-http`
   - **Frontend IP address:** `fe-day12`
   - **Backend pool:** `bep-day12`
   - **Health probe:** click **Create new** → **Name:** `probe-http`, **Protocol:** HTTP, **Path:** `/`, **Port:** 80, **Interval:** 5 seconds
   - **Port:** 80 → **Backend port:** 80
   - **Session persistence:** None
5. **Review + create** → **Create**.

**Step 6 — Add an inbound NAT rule to reach `vm-web-2` directly:**

1. Go to `lb-day12` → **Inbound NAT rules** → **+ Add**.
2. **Name:** `nat-ssh-vm2`, **Target virtual machine:** `vm-web-2`, **Network IP configuration:** its only NIC, **Frontend port:** `50022`, **Backend port:** `22`.
3. **OK**.

**Step 7 — Test it:**

1. Go to `lb-day12` → **Overview**, copy the **Public IP address**.
2. From your local machine, run `curl http://<public-ip>` repeatedly (or refresh in a browser several times). You'll see the response alternate between `Hello from vm-web-1` and `Hello from vm-web-2` — that's the 5-tuple hash spreading separate connections across the backend pool.
3. SSH directly to `vm-web-2` specifically — bypassing the pool entirely — using the inbound NAT rule's frontend port: `ssh -p 50022 azureuser@<public-ip>`.

You've now manually assembled every component the exam will ask about: a frontend IP, a backend pool, a health probe, a load balancing rule, and an inbound NAT rule for direct access to one specific VM.

> **Exam tip:** if a health probe fails for a VM, Load Balancer stops sending it *new* traffic immediately, but it does **not** terminate that VM's existing connections. Health probes affect new connection placement only.

---

## Part 2 — VM Scale Sets (VMSS)

### The Problem With Managing VMs by Hand

What you just did — building two identical VMs and registering them with a Load Balancer — doesn't scale. If traffic doubles, you'd manually create a third VM, configure it identically, and manually add it to the backend pool. If traffic drops at 2 AM, those VMs sit there costing money for no reason. **VM Scale Sets** automate all of that.

### What a VM Scale Set Is

A **VM Scale Set (VMSS)** is a single Azure resource that represents a *group* of identical VM instances. Instead of managing N separate VM resources, you manage one VMSS resource and tell it how many instances you want — or let autoscale decide.

**Orchestration modes:**

| Mode | What it means | Best for |
|---|---|---|
| **Uniform** | Every instance is identical — same image, same size, deployed from one model | Stateless, horizontally-scaled workloads: web servers, batch processing nodes |
| **Flexible** | Mix of VM sizes and configurations within the same scale set, instances behave more like individually manageable VMs | Workloads needing instance-level customization, or mixing VM sizes/Spot and on-demand instances together |

> **Exam tip:** Microsoft now recommends **Flexible** orchestration for new deployments in most scenarios (it supports features Uniform doesn't, like mixing VM sizes), but **Uniform** is still very much alive and is what most "classic" exam scenarios and existing course material describe — know that both exist and know the differentiator (identical fleet vs flexible/mixed fleet) rather than assuming one has fully replaced the other.

### Autoscale

The entire point of VMSS is **autoscale**: Azure adds or removes instances automatically based on rules you define, instead of you manually resizing the fleet.

- **Scale out** — add instances when load increases (e.g., average CPU > 70% for 5 minutes)
- **Scale in** — remove instances when load drops (e.g., average CPU < 30% for 10 minutes)
- Rules can be based on **metrics** (CPU, memory, custom Application Insights metrics, queue length) or a **fixed schedule** (e.g., scale to 1 instance overnight, 5 instances during business hours)
- You define a **minimum**, **maximum**, and **default** instance count — autoscale will never go below the minimum or above the maximum, no matter what the metrics say

### Update Policy and Overprovisioning

- **Update policy** controls how changes to the scale set's model (a new OS image, a new custom script extension version) roll out to existing instances:
  - **Manual** — you trigger updates yourself, instance by instance
  - **Automatic** — Azure updates all instances immediately
  - **Rolling** — Azure updates instances in batches, keeping the majority of the fleet healthy and serving traffic throughout the rollout
- **Overprovisioning** (Uniform mode only): when scaling out, Azure briefly creates *more* VMs than you asked for, keeps the ones that come up healthy fastest, and deletes the rest. This protects a scale-out operation from being delayed or failed by one slow or unhealthy VM creation.

### Integration with Load Balancer

This is the piece that makes VMSS genuinely hands-off: when you create a VMSS with an attached Load Balancer (or attach one afterward), **every instance the scale set creates is automatically registered into the Load Balancer's backend pool** — and automatically removed when an instance is deleted during scale-in. You never touch the backend pool manually again.

```mermaid
graph TD
    LB["Standard Load Balancer\nbep-vmss backend pool"]
    VMSS["VM Scale Set\n(min 2, max 5)"]
    I1["Instance 1"]
    I2["Instance 2"]
    I3["Instance 3 (added by autoscale)"]
    Autoscale["Autoscale Rule\nCPU > 70% → scale out"]

    Autoscale -.->|"triggers"| VMSS
    VMSS --> I1
    VMSS --> I2
    VMSS -.->|"scale-out event"| I3
    I1 --> LB
    I2 --> LB
    I3 --> LB
```

---

### Hands-On: Build a VM Scale Set with an Integrated Load Balancer

**💳 Paid — VMSS instance hours and the Standard Load Balancer it creates**

We'll build this as its own self-contained VMSS, with Azure creating a fresh Standard Load Balancer for it during the wizard — keeping it cleanly separate from the `lb-day12` you wired up by hand in Part 1.

> Note: these instances land in `subnet-app`, the same subnet `natgw-day12` already covers from Part 1 — so their cloud-init script gets the same outbound path for free and nginx installs correctly with no extra setup.

**Step 1 — Create the Scale Set:**

1. Search for **Virtual machine scale sets** → **+ Create**.
   - **Resource group:** `rg-day12-demo`
   - **Scale set name:** `vmss-day12`
   - **Region:** East US
   - **Orchestration mode:** Uniform
   - **Image:** Ubuntu Server 24.04 LTS
   - **Size:** Standard_B1s
   - **Authentication:** SSH public key
2. On **Instance count**, set:
   - **Initial instance count:** 2
3. On **Disks**, leave defaults.
4. On **Networking**:
   - **Virtual network:** `vnet-day12`
   - **Subnet:** `subnet-app`
   - **Load balancing options:** Azure load balancer
   - **Select a load balancer:** Create new → **Name:** `lb-vmss-day12`, **Type:** Public, **SKU:** Standard
   - **Backend pool:** Create new → `bep-vmss-day12`
   - **Health probe:** Create new → **Protocol:** HTTP, **Port:** 80, **Path:** `/`
5. On **Scaling**:
   - **Scaling policy:** leave **Manual** for now — we'll add a custom autoscale rule in Step 3
   - **Minimum/maximum instances:** Minimum `2`, Maximum `5`
6. On **Advanced** → **Custom data**, paste the same cloud-init script from Part 1:
   ```yaml
   #cloud-config
   package_update: true
   packages:
     - nginx
   runcmd:
     - echo "Hello from $(hostname)" > /var/www/html/index.html
   ```
7. **Review + create** → **Create**.

**Step 2 — Verify traffic distribution:**

1. Go to `vmss-day12` → **Overview**. Note the **public IP address** of `lb-vmss-day12` (visible from the scale set's Overview page, or by going to **Load balancers** directly).
2. Run `curl http://<public-ip>` repeatedly — the hostname in the response alternates between your two scale set instances, exactly as it did with the manual setup in Part 1, except this time you never touched a backend pool yourself.

**Step 3 — Configure autoscale rules:**

1. Go to `vmss-day12` → **Scaling** (under **Settings**).
2. Switch from **Manual scale** to **Custom autoscale**.
3. Under the default scale condition, click **+ Add a rule**:
   - **Metric name:** Percentage CPU
   - **Operator:** Greater than, **Threshold:** 70, **Duration:** 5 minutes
   - **Action:** Increase count by 1
4. Add a second rule:
   - **Metric name:** Percentage CPU
   - **Operator:** Less than, **Threshold:** 30, **Duration:** 10 minutes
   - **Action:** Decrease count by 1
5. Set **Instance limits**: **Minimum:** 2, **Maximum:** 5, **Default:** 2.
6. **Save**.

**Step 4 — Manually trigger and watch a scale-out:**

Waiting for real CPU load is slow for a demo, so let's force it:

1. Go to `vmss-day12` → **Scaling** → temporarily set **Default** (and minimum) to `3`, **Save**.
2. Go to **Instances** (under **Settings**) and watch a third instance appear — status moves from **Creating** to **Running** over roughly a minute, and it shows up in `lb-vmss-day12`'s backend pool automatically with zero manual steps.
3. Refresh `curl http://<public-ip>` a few times — you'll now see responses from all three instances.
4. Set the minimum back down to `2` afterward and **Save** — autoscale will scale back in once the extra instance is idle past your scale-in rule's duration.

> **Exam tip:** autoscale rules act on **aggregated metrics across the whole scale set** (e.g., *average* CPU across all instances), not any single instance. A scenario describing "one instance is overloaded but the rest are idle" is not a textbook autoscale trigger — that's a sign of an uneven load distribution problem, not a capacity problem.

**Step 5 — Leave it running for now:**

Both the Load Balancer and VMSS instances continue to bill by the hour — but we have one more thing to build in the same resource group before we tear everything down, so hold off on deleting anything until the end of Part 3.

---

## Part 3 — Application Gateway: Routing by URL, Not Just by Port

### The Problem: Load Balancer Can't See Inside the Request

Everything you built in Parts 1 and 2 routes purely on IP and port — Load Balancer has no idea whether a request is for `/checkout`, `/images/logo.png`, or `shop.example.com` vs `blog.example.com`. Often that's exactly what you want. But sometimes you need *smarter* routing: send `/api/*` requests to one set of backend VMs and everything else to another, host two completely different websites behind a single public IP, or terminate SSL/TLS once at the edge instead of on every backend VM. That's what **Application Gateway** is for.

### Application Gateway — Layer 7 HTTP/HTTPS Routing

**Application Gateway** is a fully managed **Layer 7** load balancer — it understands HTTP and HTTPS, not just TCP/UDP, which means it can make routing decisions based on the actual content of a request: the URL path, the hostname, request headers, or cookies.

**Core building blocks:**

| Component | What it does |
|---|---|
| **Listener** | Defines how the gateway accepts traffic — a **Basic listener** handles a single site; a **Multi-site listener** distinguishes between multiple hostnames on the same IP/port using the host header |
| **Backend pool** | The group of targets receiving traffic — and unlike Load Balancer, this can be VMs, VM Scale Sets, **App Service**, raw IP addresses, or even FQDNs, mixed in the same pool |
| **HTTP settings** | Defines how the gateway talks to the backend — protocol, port, cookie-based affinity, request timeout |
| **Routing rule** | Binds a listener to a destination — a **Basic rule** sends everything to one backend pool; a **Path-based rule** sends different URL paths to different backend pools via a path map |
| **Health probe** | A custom HTTP probe checking backend health — same concept as Load Balancer's probe, but it can inspect HTTP status codes and response body content, not just TCP/port reachability |

```mermaid
graph TD
    Client["Client"]
    Listener["Listener\n(Public IP, port 80)"]
    Rule["Path-Based Routing Rule"]
    PoolA["Backend Pool: app1\n(/app1/*)"]
    PoolB["Backend Pool: app2\n(/app2/*)"]

    Client --> Listener --> Rule
    Rule -->|"/app1/*"| PoolA
    Rule -->|"/app2/*"| PoolB
```

### Key Features

- **URL path-based routing** — `/api/*` to one backend pool, `/images/*` to another, `/` to a third, all through a single public IP and a single gateway
- **Multi-site hosting** — host multiple independent websites on one Application Gateway using host-header-based listeners (`app1.example.com` → Backend Pool A, `app2.example.com` → Backend Pool B)
- **SSL/TLS termination** — decrypt HTTPS once, at the gateway; backend VMs receive plain HTTP, cutting their CPU load and centralizing certificate management to one place instead of every VM
- **End-to-end SSL** — optionally re-encrypt traffic between the gateway and the backend for environments that require encryption all the way through, not just at the edge
- **Cookie-based session affinity** — route the same user to the same backend instance across multiple requests, configured per HTTP setting
- **Autoscaling** — the v2 SKU scales its own gateway capacity automatically based on traffic; you don't size it manually
- **Redirection and header rewrite** — redirect HTTP → HTTPS automatically, or rewrite request/response headers (inject CORS headers, strip internal headers) before forwarding

### SKUs

**Standard_v2** is the current generation and what you should use for any new deployment — it adds autoscaling and zone redundancy over the legacy v1 SKU, which is being phased out. (You'll also see a **WAF_v2** SKU mentioned in documentation — that adds a Web Application Firewall directly on the gateway. We're covering WAF in its own day later in this phase, paired with Azure Front Door, since WAF policies attach to both services the same way.)

> **Exam tip:** the line between Load Balancer and Application Gateway is the OSI layer. Any scenario describing routing by **URL path**, **hostname**, **cookies**, or needing **SSL termination** is a Layer 7 problem — Application Gateway. Anything that's just "spread TCP/UDP traffic across VMs" with no awareness of HTTP content is Layer 4 — Load Balancer.

| | Load Balancer | Application Gateway |
|---|---|---|
| OSI Layer | Layer 4 (TCP/UDP) | Layer 7 (HTTP/HTTPS) |
| Routing decisions based on | IP + port (5-tuple hash) | URL path, host header, cookies |
| SSL termination | No | Yes |
| Backend pool can include | VMs, VMSS | VMs, VMSS, App Service, IPs, FQDNs |
| Typical use | Raw TCP/UDP workloads, or as the backend for VMSS | Web applications needing path-based routing, SSL offload, or multi-site hosting |

---

### Hands-On: Build an Application Gateway with Path-Based Routing

**💳 Paid — Instructor Demo (~$0.246/hour Standard_v2 + capacity units). Delete immediately after.**

We'll point this gateway at two backend pools, each serving different content, and prove path-based routing actually works.

**Step 1 — Create a dedicated subnet for Application Gateway, and extend the NAT Gateway to cover it:**

Application Gateway requires its own **dedicated subnet** — it can't share `subnet-app` with the VMs or VMSS. And the Standard_v2 SKU has a hard platform requirement for **outbound internet access from that subnet** just to provision and run at all — this isn't an NSG nuance, it's how the gateway talks to Azure's control plane. With private subnets now the default, a freshly created subnet has no outbound path unless we grant one, same as Part 1.

1. Search for **Virtual networks** → `vnet-day12` → **Subnets** → **+ Subnet**.
   - **Name:** `subnet-appgw`
   - **Subnet address range:** `10.0.3.0/24`
   - Leave **Enable private subnet (no default outbound access)** ticked (it's the default) — we'll fix outbound the same way as Part 1.
   - **Save**.
2. Go to `natgw-day12` → **Subnets** → **+ Associate** → select `subnet-appgw` → **Associate**. One NAT Gateway can serve multiple subnets in the same VNet, so there's no need to deploy a second one.

**Step 2 — Reuse `vm-web-1` and `vm-web-2` as two distinct "apps":**

To keep this self-contained without deploying more VMs, we'll repurpose the two plain VMs from Part 1 as stand-ins for two different applications.

1. SSH into `vm-web-1` (via the inbound NAT rule pattern from Part 1, or temporarily attach a public IP) and run:
   ```
   echo "Welcome to App 1" | sudo tee /var/www/html/index.html
   ```
2. SSH into `vm-web-2` and run:
   ```
   echo "Welcome to App 2" | sudo tee /var/www/html/index.html
   ```

**Step 3 — Create the Application Gateway:**

1. Search for **Application gateways** → **+ Create**.
   - **Resource group:** `rg-day12-demo`
   - **Name:** `appgw-day12`
   - **Region:** East US
   - **Tier:** Standard V2
   - **Enable autoscaling:** Yes, **Minimum/Maximum instance count:** 0 / 2
   - **Virtual network:** `vnet-day12`, **Subnet:** `subnet-appgw` (the dedicated subnet from Step 1 — Application Gateway will refuse any subnet that already has other resources in it)
2. On **Frontends**: **Frontend IP address type:** Public → Create new → `pip-appgw-day12`, Standard SKU.
3. On **Backends** → **+ Add a backend pool**:
   - **Name:** `bep-app1` → add target `vm-web-1`
4. Add a second backend pool: **Name:** `bep-app2` → add target `vm-web-2`.
5. On **Configuration**, add a routing rule:
   - **Listener name:** `listener-day12`, **Frontend IP:** Public, **Protocol:** HTTP, **Port:** 80
   - **Backend targets:** select `bep-app1` as the default backend pool with a default HTTP setting (**Name:** `http-setting-default`, **Backend port:** 80)
6. **Review + create** → **Create**. (This takes 10–15 minutes to provision — Application Gateway is a heavier resource than Load Balancer.)

**Step 4 — Add path-based routing:**

1. Once deployed, go to `appgw-day12` → **Rules** → open `rule1` (the routing rule created during setup).
2. Under **Path-based rules**, click **+ Add multi-target**:
   - **Path:** `/app2/*`
   - **Backend target:** `bep-app2`
   - **HTTP setting:** Create a new setting — **Name:** `http-setting-app2`, **Backend port:** 80
   - Still inside the new HTTP setting, expand **Additional settings** and set **Override backend path** to `/`
3. **Save** — the default `/*` path target stays pointed at `bep-app1`, and now `/app2/*` is overridden to `bep-app2`.

> **Why the Override backend path matters:** without it, Application Gateway forwards the full incoming path — `/app2/` — to the backend VM as-is. Nginx then looks for a file at `/var/www/html/app2/`, finds nothing there, and returns a **404 Not Found**. Setting **Override backend path** to `/` tells the gateway to rewrite the path to `/` before forwarding the request, so nginx receives a plain root request and serves `/var/www/html/index.html` correctly. Any time a backend serves content at a different path than what the gateway listener exposes — whether the mismatch is intentional (a path-based rule) or accidental — this is the knob to fix it.

**Step 5 — Test path-based routing:**

1. Go to `appgw-day12` → **Overview**, copy the **Frontend public IP address**.
2. Visit `http://<gateway-ip>/` in a browser — you get **"Welcome to App 1"** (the default path target, `bep-app1`).
3. Visit `http://<gateway-ip>/app2/` — you get **"Welcome to App 2"** instead, even though it's the exact same public IP and the exact same gateway. The path in the URL, not the destination IP or port, decided which backend pool answered.

That's Layer 7 routing in action — something Azure Load Balancer structurally cannot do, no matter how it's configured.

**Step 6 — Final cleanup for today's entire lab:**

Everything from Parts 1–3 lives in one resource group, so this is a single step:

1. Search for **Resource groups** → open `rg-day12-demo` → **Delete resource group**.
2. Confirm by typing the resource group name, then **Delete**.

This removes the VNet, both lab VMs, `natgw-day12` and its public IP, `lb-day12` and its public IP, the VMSS and its instances, `lb-vmss-day12` and its public IP, `appgw-day12` and its public IP — every paid resource from today, in one step.

---

## Summary

**Azure Load Balancer** operates at Layer 4, distributing TCP/UDP traffic across a backend pool using a 5-tuple hash. You built one by hand — a frontend IP, a backend pool, a health probe, and a load balancing rule — plus an inbound NAT rule to reach one specific VM directly. Since September 30, 2025, Standard SKU is the only option; there's no more Basic SKU to choose between.

**VM Scale Sets** turn a fleet of identical VMs into a single manageable resource. Uniform orchestration deploys identical instances from one model; Flexible orchestration allows mixed VM sizes and configurations. When a VMSS is attached to a Load Balancer, every instance registers and deregisters from the backend pool automatically as the fleet scales — no manual pool management, ever again.

**Autoscale** is what makes this genuinely automatic: rules based on CPU (or other metrics) trigger scale-out and scale-in within a minimum/maximum instance range you define, and you watched a scale-out happen live, with the new instance picked up by the Load Balancer with zero manual steps.

**Application Gateway** does the job Load Balancer structurally can't — Layer 7 routing based on the actual content of an HTTP request. You built one with two backend pools and a path-based routing rule, and proved that `/` and `/app2/*` on the exact same public IP serve completely different backends, purely because of the URL path. SSL termination, multi-site hosting, and cookie-based affinity all follow from that same Layer 7 awareness.

### What's Next

Coming up: **VPN Gateway & ExpressRoute** — connecting your Azure VNet privately to on-premises networks, the other major networking topic in this phase. After that, we round out the phase with **Azure DNS**, and **Traffic Manager, Front Door, CDN & WAF** — where the Web Application Firewall you'd expect alongside Application Gateway gets its full treatment, paired with Front Door since WAF policies attach to both the same way.

---

## Key Takeaways

- Azure Load Balancer is **Layer 4** — it routes based on a **5-tuple hash** (source/destination IP, source/destination port, protocol) and has no awareness of HTTP, paths, or cookies
- The four core components are **frontend IP**, **backend pool**, **health probe**, and **load balancing rule**; **inbound NAT rules** are an optional fifth piece for direct access to one specific backend VM
- **Standard SKU** is the only Load Balancer SKU available since Basic was retired on September 30, 2025 — Standard is secure-by-default (NSG required to allow traffic) and requires Standard SKU public IPs on backend VMs
- **Public Load Balancer** = internet-facing frontend IP; **Internal Load Balancer** = private VNet frontend IP for tier-to-tier traffic that should never touch the internet
- Health probe failures stop *new* connections to a VM — they don't terminate existing ones
- Since **March 31, 2026**, new subnets default to **no outbound internet access** — a VM with no public IP (like our backend VMs) also has no outbound path unless you explicitly grant one via a public IP, **NAT Gateway**, or Load Balancer outbound rule; without it, a cloud-init script that needs to download packages fails silently at first boot, which looks identical to a networking/probe problem unless you know to check it
- NAT Gateway and Standard Load Balancer outbound rules **can't coexist on the same subnet** — and NAT Gateway has to be in place *before* VMs boot if cloud-init depends on outbound access, since a Load Balancer outbound rule can only be added after the Load Balancer and backend pool already exist
- **Application Gateway always needs its own dedicated subnet**, and the Standard_v2 SKU has a hard requirement for outbound internet access from that subnet to provision and run — under the new private-subnet default, that means extending a NAT Gateway (or otherwise granting outbound) to the gateway's subnet too, not just to your backend VMs' subnet
- **VM Scale Sets** manage a fleet of VMs as one resource — **Uniform** mode for identical instances, **Flexible** mode for mixed sizes/configurations
- A VMSS attached to a Load Balancer **auto-registers and auto-deregisters** instances from the backend pool as it scales — no manual backend pool management
- **Autoscale** rules act on aggregated metrics across the whole scale set (e.g., average CPU), trigger scale-out/scale-in within a minimum/maximum instance range, and can also run on a fixed schedule
- **Overprovisioning** (Uniform mode) creates extra VMs during scale-out and keeps only the fastest-healthy ones, protecting against a single slow VM delaying the whole scale event
- **Application Gateway** is **Layer 7** — it understands HTTP/HTTPS, enabling URL path-based routing, multi-site hosting via host-header listeners, and SSL termination, none of which Load Balancer can do
- Application Gateway's backend pool can mix VMs, VM Scale Sets, App Service, raw IPs, and FQDNs — far more flexible than Load Balancer's VM/VMSS-only pool
- **Standard_v2** is the current Application Gateway SKU, with built-in autoscaling; WAF is a separate add-on covered in its own day later in this phase
- Load Balancer vs Application Gateway is fundamentally an OSI-layer decision: anything involving URL-path routing, hostname-based routing, cookies, or SSL termination means Application Gateway, not Load Balancer
