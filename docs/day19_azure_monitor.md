# Day 19 — Azure Monitor, Log Analytics & Alerts

**Phase 5 — Identity, Security + Monitoring**

> For eighteen days you have been building things. Virtual machines, web apps, databases, networks, a vault. And every single one of them, right now, is running somewhere with absolutely nobody watching it. If your App Service started returning 500s an hour ago, you would not know. If someone deleted a resource group last Tuesday, you could not prove who. If the secret you stored yesterday expires at 2am on a Sunday, the first person to find out will be a customer. Today we fix that — and I want to be honest about why this day matters more than it looks. Building things in Azure is the part that gets you hired. **Knowing what your things are doing is the part that keeps you employed.** Every serious incident you will ever be part of starts with someone asking "what actually happened?", and today is where you learn to answer.

---

## What You'll Learn

**Part One — Collect and query**

- The four kinds of telemetry Azure produces, and which ones are **free and already running right now**
- **Metrics** — near-real-time numbers, free, 93 days of history you already have
- **The Activity Log** — the free audit trail that answers "who deleted that?"
- **Log Analytics workspaces** — what they are, how many you should have, and **exactly how they bill**
- **Diagnostic settings** — the one mechanism that routes everything, for every service in Azure
- **KQL** — built up properly from one line to something you'd actually use at work
- Table plans, retention, and the cost traps that produce shocking invoices

**Part Two — Detect and respond**

- Getting data off a VM: **Azure Monitor Agent and data collection rules**, and the agents that no longer exist
- **The three alert types** — and which ones are free, which cost money, and how much
- **Action groups** — what actually happens when something fires
- Building a metric alert, an activity log alert and a log alert, and watching them fire
- **Application Insights** — application performance monitoring, and what changed
- Dashboards vs workbooks vs insights — which visualisation to reach for
- Cost control, the gotchas, and an interview keyword table

---

## Before We Begin

**This is the first day of the course where a careless lab can genuinely cost you money**, so let's be precise instead of hand-wavy. Azure Monitor is consumption-priced, and the bill is driven almost entirely by **how much data you ingest**.

| | Cost | Today |
|---|---|---|
| **Platform metrics** — collection and analysis | **Free. Always. No exceptions.** 93 days of retention included. | ✅ |
| **Activity log** — collection and 90-day retention | **Free.** | ✅ |
| **Activity log, service health and resource health alerts** | **Free.** No charge for the rule at all. | ✅ |
| **Metric alert rules** | $0.10 per monitored time series per month — **with the first 10 free** | ✅ — we use one |
| **Log Analytics ingestion (Analytics plan)** | **First 5 GB per billing account per month free**, then ~$2.30/GB | ✅ — our lab is a few MB |
| **Querying Analytics-plan data** | **Free**, however much you query | ✅ |
| **Log search alert rules** | ~**$0.50/month** for the first time series, $0.05 each additional | ⚠️ **Tiny but not free — we build one and delete it** |
| **Application Insights** | No separate charge — it bills as **Log Analytics ingestion**, so the 5 GB grant covers it | ✅ |
| **`Standard_B1s` Linux VM** — today's test machine | **Free account: 750 B1s hours/month.** Otherwise ~**$0.0104/hr** — about **3 cents** for this lab | ✅ |
| **Its public IP and 30 GB Standard SSD OS disk** | ~$0.005/hr + ~$0.08/day. Pennies — but note the **disk bills even when the VM is stopped** | ✅ |
| **Azure Monitor Agent** | **The agent itself is free.** You pay only for what its data collection rule ingests — a few MB today | ✅ |
| **SMS / voice notifications** | Charged per message | 💳 — we use email only |
| **Basic logs / Auxiliary logs** | Cheaper to ingest, **charged per GB scanned to query** | 💳 — explained, not used |

!!! danger "Correcting something you'll read everywhere — including in older versions of my own notes"
    You will find dozens of blog posts, and plenty of course material, saying **"the first 5 GB per day is free."**

    **That is wrong, and it is wrong by a factor of about thirty.** The free grant is **5 GB per billing account per month.** Not per day. Not per workspace.

    That distinction is not academic. Someone who believes "5 GB/day is free" will happily switch on verbose diagnostics across twenty resources, ingest 4 GB a day, assume it's covered, and receive an invoice for roughly **$270 for that month**. This is one of the most common surprise bills in all of Azure.

    Say it back to yourself: **five gigabytes, per month, per billing account.**

**Set this up first:**

- A resource group called `rg-day19-demo`.
- Nothing else. **Everything we monitor today, we build today** — because the honest way to learn monitoring is to watch real telemetry from something real, not to stare at an empty workspace.

    At the end of Part 1 we build the two things we'll watch all day: **a web app** and **a small Linux VM**. The VM matters. Most of what you'll monitor in a real job is a machine, and half of monitoring a machine is getting data out of the operating system — which Azure does **not** do for you by default. Part 8 is where we fix that, and it needs a real VM to fix it on.

!!! tip "Day 18 left three loose ends, and they're all today's job"
    Yesterday I answered three monitoring questions in one sentence each and moved past them: **the Log Analytics workspace** that received Key Vault audit logs, **the alert on denied vault access**, and **the Event Grid notification for an expiring secret.** All three were "that's Day 19."

    This is Day 19. By the end you'll be able to build all three properly.

---

## PART ONE — COLLECT AND QUERY

### Part 1 — What Azure Monitor Actually Is

#### One Platform, Four Kinds of Data

"Azure Monitor" is not a single blade — it's the umbrella name for the whole observability platform. Everything in it is built on **four kinds of telemetry**, and knowing which is which tells you where to look, what it costs, and what you can do with it.

| | **Metrics** | **Logs** | **Traces** | **Changes** |
|---|---|---|---|---|
| **What it is** | Numbers over time — CPU %, request count, response time | Timestamped records with structure — an HTTP request, a sign-in, a role assignment | The path of a single request through your application | What changed on your resources, and when |
| **Shape** | Time series | Rows in tables | Distributed traces | Diffs |
| **Where it lives** | Azure Monitor Metrics (a time-series database) | **A Log Analytics workspace** | Application Insights (which is logs underneath) | Activity log + Change Analysis |
| **Query with** | Metric Explorer / charts | **KQL** | Application Insights | The portal |
| **Retention** | **93 days**, automatic | You choose — 30 days to 12 years | Same as its workspace | 90 days |
| **Cost** | **Free** | **Per GB ingested** — this is the bill | Bills as logs | **Free** |

Two things to take from that table before we go further.

**The free column is bigger than beginners expect.** Metrics and the activity log are collected automatically, right now, on everything you have ever built in this course, at no charge — and nobody switched them on. Go and look at any resource's **Metrics** blade after this video and there will be weeks of history sitting there.

**The bill is one column.** Logs. Data ingestion into a Log Analytics workspace is, for almost every customer, the entirety of their Azure Monitor bill. Which means cost control in Azure Monitor is really one question: *what am I ingesting, and do I need it?* We come back to this properly in Part 7.

#### The Shape of the Whole Thing

```text
      SOURCES                    ROUTING              STORES              CONSUMERS
                                                                     ┌── Metric Explorer
  Azure resources ──┐                             ┌── Metrics ───────┤
  (automatic)       │                             │   93 days, free  └── Metric alerts
                    ├──► Diagnostic settings ─────┤
  Guest OS ─────────┤    (you configure these)    │
  (needs an agent)  │                             │                  ┌── Logs / KQL
                    │                             ├── Log Analytics ─┼── Log alerts
  Applications ─────┤                             │   workspace      ├── Workbooks
  (App Insights)    │                             │   you pay per GB └── Sentinel, Defender
                    │                             │
  Control plane ────┘                             └── Activity log ──── Activity log alerts
  (automatic)                                         90 days, free
```

Everything today is somewhere on that diagram. **Diagnostic settings in the middle are the hinge** — they're the single mechanism, identical across every Azure service, that says "take this resource's telemetry and send it over there." Learn that box once and you've learned it for all of Azure.

Look at the left column for a second, though, because it's the one that catches people out. **Azure resources and the control plane report themselves automatically. The guest OS does not.** Everything happening *inside* a virtual machine — disk space, syslog, the Windows Event Log, your application's log file — needs an agent, and that agent needs configuring. That row on the diagram is Part 8, and it's the reason we're building a VM in the next two minutes.

#### Hands-On: Build the Two Things We'll Watch All Day ✅

Monitoring an empty subscription teaches nothing. We need real telemetry, so first we need something real producing it — one PaaS service and one virtual machine, because they're monitored in genuinely different ways and the difference is half of today's lesson.

**First, the web app:**

1. **App Services → + Create → Web App.** **Resource group:** `rg-day19-demo`. **Name:** `app-lwm-day19-<yourname>`. **Publish:** Code. **Runtime:** any. **OS:** Linux. **Pricing plan: Free F1.** **Review + create → Create.** **✅**
2. When it deploys, open it and click the **Default domain** URL. You should get the Azure placeholder page. **Refresh it fifteen or twenty times** — you're generating request telemetry, and we'll be using it all day. **✅**

**Now the virtual machine.** Every setting here has a reason, so don't click through on autopilot:

3. **Virtual machines → + Create → Azure virtual machine.** **✅**

   | Setting | Value | Why this value |
   |---|---|---|
   | **Resource group** | `rg-day19-demo` | Everything in one group, deleted in one action |
   | **Virtual machine name** | `vm-lwm-day19` | |
   | **Image** | **Ubuntu Server 24.04 LTS — x64 Gen2** | |
   | **Size** | **Standard_B1s** | Free account includes **750 B1s hours a month**, and B-series burstable VMs expose CPU-credit metrics nothing else has — we'll watch those in Part 2 |
   | **Authentication type** | SSH public key → **Generate new key pair** | |
   | **Public inbound ports** | **None** | We are never going to SSH into this machine |
   | **Public IP** | Leave the default (a new Standard IP) | It's for **outbound** traffic only — see the note below |
   | **OS disk type** | **Standard SSD**, 30 GB default | Cheapest option, and small enough that Part 8's disk-fill demo visibly moves the needle |
   | **Boot diagnostics** | On, with a managed storage account | Free, and the only way to see the console if a VM won't boot |

4. **Review + create → Create.** When it prompts you to download the private key, **download it and then ignore it** — we won't use it once today. **✅**

    !!! note "Why a public IP on a machine with no open ports?"
        Those two settings sound contradictory and they aren't. **Public inbound ports: None** means the NSG blocks every inbound connection — nobody can reach this VM. The public IP exists purely so the VM can make **outbound** calls, and it needs those, because in Part 8 the Azure Monitor Agent has to reach Azure Monitor's endpoints to send anything.

        This used to be free and automatic. It isn't any more: **default outbound access for new VMs has been retired**, so a VM with no public IP and no NAT gateway has **no internet access at all** — and an agent on it would install and then silently fail to send data. That is a genuinely common "why is my agent not reporting?" and you now know the answer.

        If you'd rather not have a public IP at all, the production-grade alternative is a **NAT gateway** (Day 11) — outbound only, nothing reachable inbound, about $1.10/day. For a three-hour lab the public IP is the cheaper, simpler choice.

5. When it deploys, open `vm-lwm-day19` and stay on the **Overview** page. Scroll to the bottom. **✅**

   **There are charts there already.** CPU percentage, network in and out, disk bytes — the VM has been running for ninety seconds and Azure is already recording it. Nobody enabled anything, nobody installed anything, and it costs nothing.

   That's the whole point of Part 2, which is next.

---

### Part 2 — Metrics: The Data You Already Have

Metrics are numerical values sampled at regular intervals — CPU percentage, requests per second, bytes in, queue depth. They're **collected automatically for every Azure resource**, stored in a purpose-built time-series database, available within **a minute or two of happening**, kept for **93 days**, and they cost **nothing**.

That combination — fast, free, already on — makes metrics the right first place to look at almost any problem.

#### What Makes a Metric

Three properties worth knowing by name, because the alert wizard in Part 11 asks about all three:

- **Aggregation.** Metrics are pre-aggregated into time buckets. When you chart CPU you're not seeing every sample — you're seeing the **average**, **minimum**, **maximum**, **sum** or **count** for each bucket. Choosing the wrong one hides problems: a VM averaging 40% CPU can be pegged at 100% for ten minutes inside that window, and the **average** line will never show it. **Max is the aggregation that finds spikes.**
- **Granularity.** The bucket size — one minute, five minutes, an hour. Finer granularity is available for recent data and gets rolled up as it ages.
- **Dimensions.** Name–value pairs that split a metric into series. Storage account *Transactions* has an `ApiName` dimension; App Service *Requests* can split by status code. **Splitting by a dimension turns one line into many** — and, in an alert rule, turns one billable time series into many, which is worth remembering when the invoice arrives.

#### Hands-On: Metric Explorer on the VM ✅

We'll start on the VM, because a machine is what most people end up monitoring, and because it has the richest free metric set in Azure.

1. **`vm-lwm-day19` → Monitoring → Metrics.** **✅**
2. **Metric:** `Percentage CPU`. **Aggregation:** `Average`. A mostly flat line near zero — the VM is idle. **✅**
3. **+ Add metric** → **`Available Memory Bytes`**. **✅**

    !!! warning "Yes, memory. This one has changed, and a lot of course material hasn't caught up"
        For years the standard line was *"Azure can't see memory without an agent."* **That is no longer true.** `Available Memory Bytes` and `Available Memory Percentage` are **platform metrics** — collected by the host, free, no agent, no configuration.

        If you read a blog post or watch a video telling you to install an agent just to see memory, check its date. There are still real gaps that need an agent — we'll find them at the end of this part — but memory stopped being one of them.

4. **+ Add metric** → **`CPU Credits Remaining`**. **✅**

   This one only exists on **B-series burstable** VMs, which is exactly why we chose a B1s. Remember Day 3: a B1s doesn't get a full CPU, it gets a **baseline** plus a bank of credits it spends when it bursts above that baseline. This metric is the bank balance. Watch what happens to it in a moment.

#### Hands-On: Make the Chart Move ✅

An idle VM makes a boring chart. Let's give it something to do — **without opening a single port or touching an SSH key.**

5. **`vm-lwm-day19` → Operations → Run command → `RunShellScript`.** **✅**
6. Paste this and click **Run**: **✅**

   ```bash
   nohup timeout 300 bash -c 'while :; do :; done' >/dev/null 2>&1 &
   nohup timeout 300 bash -c 'while :; do :; done' >/dev/null 2>&1 &
   echo "CPU load started - will stop by itself in 5 minutes"
   ```

   Two busy loops on a one-core VM, pegged for five minutes, then they kill themselves.

    !!! tip "Run command is the tool most people never discover"
        It executes a script **as root** on the VM, through the Azure VM agent, using the **control plane** — so it needs **no open port, no SSH key, no public IP reachability and no Bastion.** It's an RBAC-gated operation: anyone with the right role can run commands on the machine.

        That's worth two thoughts. Operationally it's the fastest way to fix a VM you've locked yourself out of. From a security angle, **`Microsoft.Compute/virtualMachines/runCommand/action` is effectively root on the box**, which is a very good reason not to hand out Contributor on production VMs — Day 17's lesson showing up in a place people don't expect.

        It's also why our VM has no inbound rules at all today. We never needed them.

7. Wait **two minutes** (metrics lag by a minute or two — say that out loud rather than clicking refresh in silence), then watch the chart. **✅**

   **`Percentage CPU` climbs to ~100%.** And on the same chart, **`CPU Credits Remaining` starts falling** — you are watching the VM spend its burst budget in real time. Leave it long enough on a B1s and CPU gets throttled back to the baseline. That is the single clearest demonstration of burstable sizing anywhere in this course, and it cost nothing.

#### Hands-On: The Controls, and Dimensions on the App Service ✅

8. Still in Metric Explorer, work the toolbar — this blade is powerful and most people use a tenth of it: **✅**
   - **Time range** (top right) → last 30 minutes, then last 24 hours. Watch the granularity change automatically.
   - Change the chart type to **Bar** or **Area**.
   - **Add filter** and **Apply splitting** — both greyed out or empty for most VM metrics, because **VM metrics have almost no dimensions.**

9. So switch to the web app for that lesson: **App Service → Monitoring → Metrics**. **Metric:** `Requests`, **Aggregation:** `Sum`. **✅**
10. **Apply splitting** → split by **Http Status**. One line becomes several. **✅**

    That's dimensions, live — and it's why the App Service is the better teaching example here. The VM has the richer metrics; the App Service has the richer *shape*.

11. Click **Save to dashboard → Pin to dashboard**. Part 13 comes back to this. **✅**
12. Now the trick nobody shows you: click **New alert rule** at the top of the Metrics blade. **✅**

    It opens the alert wizard **with your metric, aggregation and splitting already filled in**. Building an alert by first getting the chart right, then clicking that button, is far easier than building the rule from scratch. Close it without saving — Part 11 does this properly.

!!! tip "The 93 days is already there, on everything"
    Go and open the Metrics blade on any resource you built earlier in this course and change the time range to the last 30 days. **The history is there**, because platform metrics were collected the whole time without anyone enabling anything.

    That's genuinely useful during an incident: *"has this been happening for weeks or did it start today?"* is often the most valuable question, and metrics can answer it retroactively even in an environment where nobody set up monitoring.

#### What Metrics Cannot Tell You — Find the Gap Yourself

Before we leave metrics, one exercise, and it sets up the second half of today.

13. Back on the **VM's Metrics** blade, open the metric dropdown and type **`disk`** in the search box. **✅**

    Count what comes back: *Disk Read Bytes*, *Disk Write Operations/Sec*, *OS Disk Latency*, *OS Disk Queue Depth*, *OS Disk IOPS Consumed Percentage*, data disk versions of all of them, burst credit percentages. **Dozens of disk metrics.**

    Now find the one that tells you **how full the disk is**.

    **It isn't there.** Azure will tell you your disk is doing 40 IOPS at 3 milliseconds of latency, and it cannot tell you that it is 98% full and about to take your application down tonight.

That's not an oversight — it's the boundary. **Platform metrics are measured by the host, from outside the machine.** The host can see the virtual disk's I/O because it's serving it. It cannot see the *filesystem* on that disk, because filesystems exist inside the guest OS, and Azure deliberately does not look inside your VM.

Three things sit on the far side of that boundary:

| Not available from platform metrics | Why |
|---|---|
| **Disk space used and free** | A filesystem concept, invisible from the host |
| **Per-process data** — what's actually eating the CPU | Inside the guest |
| **Every log in the OS** — syslog, Windows Event Log, your application's log file | Inside the guest |

Getting at those three needs an **agent inside the machine**, and that is **Part 8**. Hold that thought — we come back to this exact VM and get the disk number the portal just refused to give us.

---

### Part 3 — The Activity Log: Who Did That?

The **Activity log** is a subscription-level record of every **control-plane** operation — every create, update and delete performed through Azure Resource Manager, with the identity that did it, when, and whether it succeeded.

It's on by default, free, and kept for **90 days**.

Note what it is and isn't, because this is exactly the control-plane/data-plane split from Days 17 and 18 wearing a third hat:

- **It records:** someone deleted a resource group, someone assigned a role, someone restarted a VM, someone changed a network security group rule.
- **It does not record:** someone read a blob, someone fetched a secret, someone queried a database. Those are **data-plane** operations, and they only exist if you enabled a diagnostic setting on that service — which is exactly what we did to Key Vault yesterday.

#### The Categories

| Category | What it contains |
|---|---|
| **Administrative** | Every ARM create/update/delete. The big one. **Includes role assignments.** |
| **Service Health** | Azure-side incidents affecting services you use |
| **Resource Health** | The health of your specific resources |
| **Alert** | Records of alerts firing |
| **Autoscale** | Scale-out and scale-in events |
| **Policy** | Azure Policy effects — including deny events |
| **Recommendation** | Azure Advisor recommendations |

#### Hands-On: Read the Audit Trail ✅

1. Search **Monitor → Activity log**. Or open it on any resource for a pre-filtered view. **✅**
2. Set **Timespan** to the last 24 hours and look at what's there — everything you created today, and probably yesterday's Key Vault work too. **✅**
3. Click any entry and work through the tabs: **Summary**, **JSON**, **Change history**. **✅**

   **The JSON tab has a `caller` field.** That's the identity — a UPN for a person, a GUID for a service principal or managed identity. **This is the field that answers "who did that?"**, and it's the reason you can prove things after the fact.

4. Add a filter: **Operation** → search `Delete`. Now you're looking at every deletion in the subscription. **✅**
5. Click **Download as CSV**, and note the **Insights** and **Quick Insights** links. **✅**

!!! warning "90 days, and it isn't searchable the way you want"
    Two limitations that push every serious environment to do one extra thing:

    1. **Retention is 90 days and cannot be extended** in the activity log itself.
    2. The portal filters are fine for browsing and useless for real analysis — you cannot join it to anything, aggregate it properly, or alert on a pattern.

    Both are solved the same way: **a diagnostic setting on the activity log, exporting it to a Log Analytics workspace.** Then it's the `AzureActivity` table, queryable with KQL, retained as long as you want, and joinable to everything else. We do that in Part 5, and it's one of the first things I'd set up in any new subscription.

---

### Part 4 — The Log Analytics Workspace

A **Log Analytics workspace** is the store and query engine for all log data in Azure Monitor. It is a real Azure resource with a region, an access model, a retention setting and a bill.

It's also the thing that Microsoft Sentinel, Microsoft Defender for Cloud, Application Insights, VM Insights and Container Insights are all built on top of. **Learn the workspace and you've learned the foundation of half of Azure's security and operations tooling** — which is why yesterday's Key Vault audit logs went into one.

#### How Many Should You Have?

This is a real design question and it comes up in interviews.

**Start with one workspace per region per environment, and add more only for a reason.** A single workspace is simpler, lets you query across everything with one KQL statement, and consolidates your data volume toward better commitment-tier pricing.

Reasons that genuinely justify splitting:

- **Data sovereignty.** Data must stay in a particular geography — the workspace's region determines where data is stored.
- **Access isolation.** Different teams must not see each other's logs. There's a middle path here worth knowing: **resource-context RBAC** means a user with read access to a *resource* can query that resource's logs without access to the whole workspace, which solves a lot of cases without splitting anything.
- **Different retention or cost ownership.** Different chargeback boundaries, or one dataset that needs seven years and another that needs thirty days. (Though **per-table retention**, in Part 7, often solves this inside one workspace.)

The anti-pattern is **a workspace per application**, which people reach for by instinct. It fragments your data so cross-service queries become impossible, and it's the reason nobody can answer "did the database slowdown and the app errors happen at the same moment?"

#### Hands-On: Create a Workspace ✅

1. Search **Log Analytics workspaces → + Create**. **✅**
2. **Resource group:** `rg-day19-demo`. **Name:** `law-lwm-day19`. **Region:** the same region as your App Service and VM. **Review + create → Create.** **✅**

    Keep everything in one region today. Part 8's data collection rule **must** live in the same region as this workspace, and one region for the whole lab removes a class of confusing failures.
3. Open it and read the left menu — this is the map of the rest of the day: **✅**
   - **Logs** — the KQL query editor (Part 6)
   - **Tables** — every table, with its **table plan** and retention (Part 7)
   - **Usage and estimated costs** — what you're ingesting and what it will cost (Part 14)
   - **Data collection rules** — how agents get data here (Part 8)
   - **Agents** — connected machines
4. On **Overview**, note the **Workspace ID**, and under **Settings → Agents**, note that keys exist. **✅**
5. **Settings → Usage and estimated costs.** It's empty because we've ingested nothing. **Come back at the end of the video and it won't be.** **✅**

!!! note "The region matters for two reasons"
    **Cost** — sending data across regions can incur bandwidth charges (though notably, **data sent via diagnostic settings does not**). And **latency and sovereignty** — the data physically lives in the workspace's region.

    Put the workspace where the things it monitors are.

---

### Part 5 — Diagnostic Settings: The Hinge

Here is the single most transferable thing in today's video.

**Every Azure service produces logs and metrics that it does not send anywhere by default. A diagnostic setting is what routes them.** Storage accounts, Key Vault, App Service, SQL, NSGs, Front Door, AKS — every one of them, same blade, same shape, same four destinations.

Yesterday you configured one on Key Vault and I said "this is how every Azure service does it." This is where I prove that.

#### The Four Destinations

| Destination | Use it for | Cost shape |
|---|---|---|
| **Log Analytics workspace** | Querying with KQL, alerting, dashboards, Sentinel | Per GB ingested |
| **Storage account** | Cheap long-term archive, compliance retention | Storage rates — very cheap, and not queryable |
| **Event Hub** | Streaming out to a third-party SIEM (Splunk, Datadog, Elastic) | Event Hub throughput units |
| **Partner solution** | Direct integration with a partner service | Varies |

You can send to **several at once**, and the classic production pattern uses two: **Log Analytics for 30–90 days of interactive querying, plus a storage account for years of cheap archive.**

!!! warning "The Off-By-Default Rule"
    Get this wrong and you'll find out in the worst possible way.

    **Resource logs are not collected until you create a diagnostic setting.** They are not buffered. They are not retrievable retroactively. **A service that wasn't configured on Tuesday has no Tuesday logs, and there is no support ticket that recovers them.**

    Which is why "turn on diagnostic settings" belongs in your resource-creation checklist, not in your incident response. Yesterday's Key Vault lesson, restated: **you cannot answer "who read that secret last month?" if you weren't logging last month.**

    The scalable answer is **Azure Policy with a `DeployIfNotExists` effect** — a policy that automatically creates the diagnostic setting on every resource of a given type, including ones created next year by someone who's never heard of you. That's Day 17's policy lesson meeting today's, and it's how mature organisations do this.

#### Hands-On: Route the Activity Log ✅

The first diagnostic setting to create in any new subscription.

1. **Monitor → Activity log → Export Activity Logs** (top toolbar). **✅**
2. **+ Add diagnostic setting.** **✅**
3. **Name:** `activity-to-law`. **Categories:** tick **Administrative**, **Policy**, **Security**, **ServiceHealth**, **ResourceHealth**. **✅**
4. **Destination:** *Send to Log Analytics workspace* → `law-lwm-day19`. **Save.** **✅**

   The subscription's control-plane audit trail is now flowing into a queryable table called `AzureActivity`, with whatever retention you choose, instead of expiring at 90 days in a blade you can't query properly.

#### Hands-On: Route the App Service Logs ✅

5. **App Service → Monitoring → Diagnostic settings → + Add diagnostic setting.** **✅**
6. **Name:** `app-to-law`. **✅**
7. **Logs:** tick **HTTP logs** (`AppServiceHTTPLogs`) and **App Service Console Logs**. Under **Metrics**, tick **AllMetrics**. **✅**

    !!! tip "Why send metrics to logs when metrics are already free?"
        Good instinct, and there's a real answer. Platform metrics are free and kept 93 days — but they live in a **separate store** you can't join to log data, and they expire at 93 days.

        Ticking **AllMetrics** copies them into the workspace as the `AzureMetrics` table, where you **can** join them to logs and keep them for years. You pay ingestion for that copy.

        So: **don't tick it by default.** Tick it when you specifically need metrics alongside logs in one query, or need them for longer than 93 days. Ticking it on everything out of habit is a good way to inflate a bill for data you already had for free.

8. **Destination:** `law-lwm-day19`. **Save.** **✅**
9. Go back to your App Service URL and **refresh it another twenty times.** You're now generating log data that lands in the workspace. **✅**

    !!! note "Give it five to fifteen minutes"
        First-time log flow is not instant. The table has to be created and the pipeline has to warm up. If Part 6's queries come back empty, that's almost certainly why — go and make a coffee rather than assuming you did it wrong.

---

### Part 6 — KQL: The Query Language

**Kusto Query Language** is how you ask questions of log data. It is the skill from today that you will use most, it's the same language used by Sentinel, Defender, Azure Data Explorer and Resource Graph, and it is genuinely pleasant once the shape clicks.

#### The Shape

A KQL query is **a table, then a pipeline of operators**, each taking rows in and passing rows out:

```kusto
TableName
| operator
| operator
| operator
```

Read top to bottom, left to right. That's it. If you can read a sentence you can read KQL.

#### The Operators That Cover 90% of Real Work

Let's build one query up, adding a line at a time, so each operator earns its place.

**Start with a table.** Always dangerous on its own — this is *every* row:

```kusto
AzureActivity
```

**`take` — grab a few rows to see the shape.** Your first move with an unfamiliar table, every time:

```kusto
AzureActivity
| take 10
```

**`where` — filter.** Put it as early as possible; it makes everything after it faster and, on Basic/Auxiliary tables, cheaper:

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue contains "DELETE"
```

`ago()` is the function you'll type most: `ago(30m)`, `ago(7d)`.

**`project` — choose columns.** Log tables are wide and mostly noise:

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue contains "DELETE"
| project TimeGenerated, Caller, OperationNameValue, ActivityStatusValue, _ResourceId
```

**`summarize` — aggregate.** The operator that turns rows into answers:

```kusto
AzureActivity
| where TimeGenerated > ago(7d)
| summarize OperationCount = count() by Caller
| order by OperationCount desc
```

*Who has been busiest in this subscription this week?* — answered.

**`bin()` — group time into buckets**, which is what makes a chart possible:

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| summarize count() by bin(TimeGenerated, 1h)
| render timechart
```

**`render`** turns the result into a chart — `timechart`, `barchart`, `piechart`, `columnchart`.

**`extend` — add a calculated column:**

```kusto
AppServiceHTTPLogs
| where TimeGenerated > ago(1h)
| extend IsError = ScStatus >= 400
| summarize Total = count(), Errors = countif(IsError) by bin(TimeGenerated, 5m)
| extend ErrorRate = round(100.0 * Errors / Total, 2)
| render timechart
```

That one is worth reading twice — it's a real error-rate query, and it's six lines.

**`let` — name things**, so a query stays readable:

```kusto
let lookback = 24h;
let errorThreshold = 400;
AppServiceHTTPLogs
| where TimeGenerated > ago(lookback)
| where ScStatus >= errorThreshold
| summarize Errors = count() by CsUriStem
| top 10 by Errors desc
```

**`union` and `join`** combine tables — `union` stacks rows, `join` matches them on a key. You need to know they exist; you'll reach for them the first time you ask a question spanning two services.

#### Hands-On: Query Your Own Data ✅

1. **Log Analytics workspace → Logs.** Dismiss the queries dialog if it appears. **✅**
2. Look at the left panel — every table you have data in, grouped by solution. **This is how you discover what you've actually got.** **✅**
3. Run each of these, one at a time: **✅**

   **What am I even collecting?**
   ```kusto
   search *
   | summarize Records = count() by ["$table"]
   | order by Records desc
   ```

    !!! tip "What `$table` is"
        `search *` scans every table in the workspace and adds a built-in column called **`$table`** holding the name of the table each row came from. Summarising by it gives you one row per table with a record count — a complete inventory of what you're actually ingesting, in three lines.

        The square brackets and quotes around `["$table"]` are required, because `$` isn't a legal character in a bare KQL column name. Leave them off and the query fails to parse.

        Run this on any unfamiliar workspace as your very first query. It tells you what's being collected before you waste time guessing at table names — and it's often the fastest way to spot something expensive that nobody meant to switch on.

   **Every control-plane operation in the last day:**
   ```kusto
   AzureActivity
   | where TimeGenerated > ago(1d)
   | project TimeGenerated, Caller, OperationNameValue, ActivityStatusValue
   | order by TimeGenerated desc
   ```

   **Your own web traffic:**
   ```kusto
   AppServiceHTTPLogs
   | where TimeGenerated > ago(1h)
   | project TimeGenerated, CsMethod, CsUriStem, ScStatus, TimeTaken
   | order by TimeGenerated desc
   ```

   **Requests over time, as a chart:**
   ```kusto
   AppServiceHTTPLogs
   | where TimeGenerated > ago(1h)
   | summarize Requests = count() by bin(TimeGenerated, 5m)
   | render timechart
   ```

   **Slowest requests:**
   ```kusto
   AppServiceHTTPLogs
   | where TimeGenerated > ago(1h)
   | summarize AvgMs = avg(TimeTaken), MaxMs = max(TimeTaken), Hits = count() by CsUriStem
   | order by AvgMs desc
   ```

4. Set the time picker to **Last 24 hours**, and try **Save → Save as query** on one you liked. **✅**
5. Click **Queries** in the toolbar and browse Microsoft's library of prebuilt queries for the tables you have. **This is the fastest way to learn KQL** — find one close to what you want and modify it. **✅**

!!! tip "Two habits that separate good KQL from bad"
    **Filter early.** `where` before `summarize`, and always bound the time range. A query that scans a month to summarise an hour is slow, and on Basic or Auxiliary tables it's *expensive*, because those bill per GB scanned.

    **`take 10` first.** Before writing anything clever against an unfamiliar table, look at ten rows. You'll discover the column names are never quite what you assumed — `OperationNameValue` and not `OperationName`, `ScStatus` and not `StatusCode` — and you'll save yourself fifteen minutes of guessing.

---

### Part 7 — Table Plans, Retention, and Not Getting a Shocking Bill

This part has no demo and it is the most financially valuable ten minutes in the video.

#### Three Table Plans

Not all log data deserves the same treatment. A security audit log you query weekly and a firewall log you keep purely for compliance should not cost the same, and since 2024 they don't have to.

| Plan | Ingest cost | Query cost | Queryable for | Use it for |
|---|---|---|---|---|
| **Analytics** | ~$2.30/GB | **Free** | Full retention | Anything you query, alert on, or build dashboards from. **The default.** |
| **Basic** | ~$0.50/GB | **Charged per GB scanned** | 30 days | High-volume logs you occasionally search during an investigation |
| **Auxiliary** | ~$0.05/GB | **Charged per GB scanned** | Full retention | Verbose, low-value logs kept mainly for compliance |

The trade is straightforward: **cheaper to store, more expensive to look at, and fewer features.** Basic and Auxiliary tables have restricted KQL and can't be used for log alerts.

The mistake to avoid: **moving a table to Basic to save money, then querying it constantly.** You can end up paying more, not less. Basic and Auxiliary are for data you *store* and rarely *read*.

#### Retention: Two Numbers, Not One

The retention model has two settings, and mixing them up is common:

- **Interactive retention** — how long data stays immediately queryable with normal KQL. **The first 31 days are included free.** Beyond that, roughly $0.10/GB/month, up to two years.
- **Total retention** (long-term) — how long the data exists at all, up to **12 years**, at roughly $0.02/GB/month. Data past the interactive period isn't gone; it needs a **search job** to bring it back, and that's charged.

The default retention for Analytics tables in a new workspace is **30 days** — comfortably inside the free interactive window, which is not an accident.

And the feature that saves real money: **retention is configurable per table.** You don't have to choose one number for everything. Keep `AzureActivity` for two years because it's small and it's your audit trail; keep verbose HTTP logs for 30 days because there's a lot of it and it ages badly.

#### The Four Ways People Overspend

1. **Ticking every category on every diagnostic setting.** By far the biggest one. Someone enables "all logs" on a chatty service — Application Gateway, AKS, Front Door — and the volume is enormous. **Enable the categories you'll actually query.**
2. **Ticking `AllMetrics` everywhere out of habit.** Paying to ingest data that was already free in the metrics store, for 93 days, with a better query experience for charting.
3. **Long retention on everything.** Two years of debug logs nobody has read since the week they were written.
4. **Verbose application logging left at Debug in production.** A single chatty app can outweigh your entire infrastructure's telemetry.

#### The Controls

- **Daily cap.** Set a hard ceiling on GB per day for the workspace. Ingestion **stops** when it's hit — which protects the bill and blinds you, so **always create an alert on the cap being reached**. Excellent as a runaway guard, dangerous as a routine mechanism.
- **Commitment tiers.** Commit to 100 GB/day or more for a substantial discount over pay-as-you-go. Only relevant at real scale.
- **Table plans and per-table retention**, as above.
- **The `Usage` table**, which tells you exactly where the money is going:

```kusto
Usage
| where TimeGenerated > ago(30d)
| where IsBillable == true
| summarize BillableGB = sum(Quantity) / 1000 by DataType
| order by BillableGB desc
```

!!! tip "Run that query on any workspace you inherit"
    It takes ten seconds and it will tell you, in one sorted list, which table is responsible for the bill. In almost every real environment the top one or two rows account for most of the spend, and at least one of them will be something nobody has queried in a year.

    Being the person who runs that query and then halves the monitoring bill is a very good way to be noticed.

---

## PART TWO — DETECT AND RESPOND

> **Halfway point.** You can now collect telemetry and ask it questions. But everything so far has required *you* to go and look — and nobody is looking at 3am on a Sunday. The rest of the day is about the system looking for you, and telling you when something is wrong.

---

### Part 8 — Getting Data Off a Machine: Agents and Data Collection Rules

Azure collects platform metrics and resource logs about your VM for free — but that's data about the **virtual machine as a resource**. It tells you nothing about what's happening **inside** the operating system: disk space, memory pressure, the Windows Event Log, syslog, application log files.

For that you need an agent, and this is an area where the landscape changed and a lot of published material is now simply wrong.

#### The Agents That No Longer Exist

| Agent | Status |
|---|---|
| **Log Analytics agent (MMA/OMS)** | **Retired 31 August 2024.** Gone. |
| **Azure Diagnostics extension (WAD/LAD)** | **Deprecated 31 March 2026.** |
| **Azure Monitor Agent (AMA)** | **The only current answer.** |
| **Dependency Agent** (VM Insights Map) | Retires **30 June 2028** |

If a tutorial tells you to install the Log Analytics agent and paste a workspace ID and key, **it is describing a product that no longer exists.** Given how much Azure content on the internet predates 2024, this is worth being alert to.

#### Data Collection Rules

The Azure Monitor Agent works differently from what it replaced, and the difference is the important part.

The old agent's configuration lived **on the workspace**: connect a machine, and it collected whatever that workspace was configured to collect. One setting, everyone gets it.

AMA's configuration lives in a **data collection rule (DCR)** — a separate Azure resource that says *collect this data, from these machines, and send it to these destinations*. Rules and machines are associated many-to-many:

- **One machine can have several DCRs.** Baseline metrics from one, security event collection from another, an application's log file from a third.
- **One DCR can target hundreds of machines.**
- **One DCR can send different data to different destinations** — security events to the Sentinel workspace, performance counters to the ops workspace.
- **It's a resource**, so it's deployable with Bicep or Terraform, governable with Azure Policy, and controlled with RBAC.

That's a genuine improvement, and it's why "AMA plus DCR" is the phrase to remember rather than just "the new agent."

!!! warning "A retirement that lands in days: 14 September 2026"
    **The Data Collector API retires on 14 September 2026.**

    That's the old HTTP endpoint for pushing custom log data into a Log Analytics workspace — the one every "send custom logs to Azure" script and integration on the internet uses. Its replacement is the **Logs Ingestion API**, which is DCR-based, uses Entra ID authentication instead of a shared key, and supports transformations on the way in.

    If you inherit an environment with a script POSTing to a workspace with a shared key, **that is the thing to look at first.** By the time most people watch this, that date is here or just passed.

#### Transformations

One capability worth knowing by name because it's the modern answer to cost control at the source. A DCR can apply a **KQL transformation to data in flight** — before it's stored, and therefore before you're billed.

You can drop rows you don't care about, drop columns nobody reads, or mask a field containing personal data. **Filtering out 60% of a verbose log at ingestion time removes 60% of its cost**, which is a far better lever than deleting data later.

#### Hands-On: Build a Real Data Collection Rule ✅

This is the part Part 2 promised. `vm-lwm-day19` has been running since Part 1 and we still cannot see how full its disk is. Let's fix that properly.

!!! tip "Build first, talk second — a recording-order note"
    The agent takes **five to fifteen minutes** to install and start sending. So create the rule **now**, then go back and read the concept sections above while it works. Don't create it and then sit watching an empty table — that's the most boring five minutes you can put in a video, and it's entirely avoidable.

1. **Monitor → Settings → Data Collection Rules → + Create.** **✅**

    You may see a banner offering the **classic creation experience**. Either works; the steps below follow the current default one, and the classic version differs mainly in having an explicit Windows/Linux **Platform Type** selector on the Basics tab.

2. **Basics:** **✅**

   | Field | Value |
   |---|---|
   | **Rule Name** | `dcr-lwm-day19` |
   | **Resource group** | `rg-day19-demo` |
   | **Region** | **the same region as `law-lwm-day19`** |
   | **Data Collection Endpoint** | **Leave empty** |

    !!! warning "The region has to match the workspace"
        A DCR must be in the **same region as any Log Analytics workspace it delivers to.** Get it wrong and the workspace simply won't appear in the destination dropdown later, with no explanation of why. If you have machines spread across regions, you need one DCR per region pointing at the same workspace.

    !!! note "Why no data collection endpoint?"
        A **DCE** is only needed for specific scenarios — **Azure Monitor Private Link**, or the **Logs Ingestion API** for custom data. Performance counters and syslog from AMA don't need one. Know the acronym for the exam; don't create one today.

3. **Resources → + Add resources →** tick **`vm-lwm-day19`** → **Apply**. **✅**

   Stop here for a second, because three things just happened that you didn't ask for:

   - **The Azure Monitor Agent will be installed automatically** on that VM. There is no separate "install the extension" step any more — creating a DCR and adding resources **is** the recommended way to deploy AMA from the portal.
   - **An association** between the rule and the machine is created. That's a real object you can list later.
   - **A system-assigned managed identity is enabled on the VM**, because that's how the agent authenticates to Azure Monitor. Day 17's managed identity, switched on for you, holding a credential nobody has to store.

4. **Collect and deliver → + Add new dataflow** (or **+ Add data source** in the classic experience). **Data source type: Performance Counters.** **✅**
5. Choose **Custom**, set **sample rate to 60 seconds**, and select these four: **✅**

   ```text
   Processor(*)\% Processor Time
   Memory(*)\% Used Memory
   Logical Disk(*)\% Used Space        ← the one Part 2 could not give us
   Logical Disk(*)\Free Megabytes
   ```

6. **Destination:** *Azure Monitor Logs* → `law-lwm-day19`. **Add data source.** **✅**
7. **+ Add new dataflow** again. **Data source type: Linux Syslog.** **✅**
8. Select only the facilities you actually want — **`user`**, **`auth`** and **`daemon`** — with a minimum log level of **`LOG_INFO`**. Destination: the same workspace. **✅**

    !!! danger "This screen is where people burn the 5 GB grant"
        The syslog picker lets you tick **every facility at `LOG_DEBUG`** in about two seconds. Do that across a fleet and you will ingest gigabytes a day of noise nobody will ever read, at ~$2.30 a gigabyte.

        **Collect the facilities you'd actually investigate, at the level you'd actually act on.** Everything else is a bill with no reader. This is the single highest-leverage cost decision in Azure Monitor, and it's made on a checkbox screen that looks harmless.

9. **Review + create → Create.** **✅**

#### Hands-On: Verify the Agent Before You Blame the Query ✅

Now go and teach the rest of Part 8 while that installs. When you come back:

10. **Log Analytics workspace → Logs**, and run the check that should always come first: **✅**

    ```kusto
    Heartbeat
    | where TimeGenerated > ago(30m)
    | summarize arg_max(TimeGenerated, *) by Computer
    | project Computer, TimeGenerated, Category, Version, OSType
    ```

    **A healthy agent writes a `Heartbeat` record every single minute.** If `vm-lwm-day19` is there with a timestamp under a minute old, the agent is installed, authenticated and talking to Azure Monitor.

    **Make this your reflex.** When guest data is missing, check `Heartbeat` before you touch the query. No heartbeat is an agent, network or identity problem; a heartbeat with no data is a DCR configuration problem. Those are two completely different afternoons.

    !!! note "If there's no heartbeat after fifteen minutes"
        | Check | Why |
        |---|---|
        | **VM → Extensions + applications** | Is `AzureMonitorLinuxAgent` listed, and does it say **Provisioning succeeded**? |
        | **VM → Identity** | Is the system-assigned identity **On**? The agent authenticates with it |
        | **Outbound internet** | No public IP and no NAT gateway means the agent can install and never send. The single most common cause |
        | **DCR region** | Must match the workspace region |

#### Hands-On: Get the Number the Portal Refused to Give You ✅

11. Give the agent a couple of minutes of perf samples, then run: **✅**

    ```kusto
    Perf
    | where TimeGenerated > ago(1h)
    | where ObjectName == "Logical Disk" and CounterName == "% Used Space"
    | summarize avg(CounterValue) by bin(TimeGenerated, 5m), InstanceName
    | render timechart
    ```

    **There it is.** The percentage of disk used, per filesystem — the exact number that does not exist anywhere in platform metrics, now sitting in a table you can query, chart, alert on and keep for a year.

12. Make it move. **VM → Run command → `RunShellScript`:** **✅**

    ```bash
    logger -p user.err "LWM Day 19 - simulated application error"
    logger -p user.info "LWM Day 19 - routine informational message"
    fallocate -l 4G /var/tmp/bigfile
    df -h /
    ```

    `df` reports the disk several gigabytes fuller than it was. Wait a few minutes and **re-run the query** — the line **steps up**.

13. And the logs from inside the machine: **✅**

    ```kusto
    Syslog
    | where TimeGenerated > ago(1h)
    | project TimeGenerated, Computer, Facility, SeverityLevel, SyslogMessage
    | order by TimeGenerated desc
    ```

    Your two `logger` messages are in there, alongside everything else the OS has been saying — sudo invocations, systemd units starting, the Run command extension itself doing its work. **None of this was visible to Azure twenty minutes ago.**

14. Put the disk back: **Run command** → `rm /var/tmp/bigfile`, and watch the chart come down. **✅**

    That round trip — a real change inside a VM, visible as a queryable number in Azure — is the whole reason agents exist.

#### One Click That Does All Of This: VM Insights

15. **Monitor → Virtual Machines → Insights**, or **VM → Monitoring → Insights**. **Read it, don't enable it.** **✅**

    **VM Insights** is the packaged version of everything you just built by hand: enable it on a VM and Azure creates its own DCR (you'll see them named `MSVMI-*`), installs AMA, collects a standard performance set, and gives you pre-built charts plus — with the Dependency Agent — a **Map** view showing which processes are talking to what.

    It's genuinely good, and it's what most teams use. Two reasons we built the DCR manually first: **you can't tune what you don't understand**, and VM Insights collects **considerably more data** than our four counters. On one lab VM that's fine. On two hundred, it's a conversation with whoever owns the bill.

16. **Log Analytics workspace → Settings → Agents.** Read it and note what's *missing*: no "download the agent, paste this workspace ID and key" flow. That world is gone. **✅**

!!! tip "The exam-shaped answer"
    *"How do you collect the Windows Security Event Log from 200 VMs into Azure Monitor?"* → **Azure Monitor Agent, configured with a data collection rule, associated with those machines — deployed at scale with Azure Policy.**

    If your answer mentions the Log Analytics agent, a workspace key, or the diagnostics extension, it's out of date by two years.

---

### Part 9 — Alerts: The Three Types, and What They Cost

An alert rule is three things: **a signal to watch**, **a condition that makes it fire**, and **an action group that does something about it.** Same three parts every time.

#### The Three Types That Matter

| | **Metric alert** | **Log search alert** | **Activity log alert** |
|---|---|---|---|
| **Watches** | A platform or custom metric | A KQL query result | Control-plane events |
| **Speed** | **Near real-time**, ~1 min | Depends on frequency (5 min+) | Near real-time |
| **Answers** | "CPU is above 80%" | "More than 10 errors in 5 minutes" | "Someone deleted a resource group" |
| **Stateful?** | Optional — *Automatically resolve alerts* | Optional | No |
| **Cost** | $0.10/time series/month, **10 free** | ~**$0.50/month** + $0.05 per extra series | **Free** |

The practical guidance follows from that table: **use a metric alert if a metric can express it.** They're faster, simpler, and effectively free at small scale. Reach for a log alert only when the question genuinely needs the query language — correlating fields, counting distinct things, matching text.

There's also **resource health** and **service health** alerts, both free, and both worth having: *is my resource unhealthy*, and *is Azure itself having a problem in my region*. That second one saves you from spending an hour debugging Microsoft's outage.

#### Severity, and Using It Properly

Alerts carry a severity from **Sev 0 (Critical)** to **Sev 4 (Verbose)**. It's not decoration — it's what lets an action group route a Sev 0 to someone's phone and a Sev 3 to a mailbox.

The discipline: **Sev 0 and Sev 1 must mean "wake someone up."** The fastest way to make monitoring worthless is to mark everything critical. When every alert is urgent, people stop reading them — and the one that mattered arrives in a mailbox nobody opens. **Alert fatigue is the most common failure mode of monitoring, and it's self-inflicted.**

#### Stateful vs Stateless

A subtlety that catches people out. By default, **metric alerts are stateless**: the condition is met, it fires; still met on the next evaluation, it fires again. Notifications keep coming.

Tick **Automatically resolve alerts** and the rule becomes **stateful**: it fires once, stays in a *Fired* state while the condition holds, and moves to *Resolved* when it clears — sending a resolution notification.

**You almost always want this on.** It's the difference between "your disk filled up, and then recovered" and four hundred identical emails.

#### Alert Processing Rules

One more piece of vocabulary, because it solves a problem you'll hit within a month of running real alerts: **alert processing rules** sit between an alert firing and the notification going out. They can **suppress** notifications during a planned maintenance window, or **add an action group** to every alert in a scope without editing each rule.

That maintenance-window suppression is the answer to *"we're doing a deployment tonight, please stop paging the on-call engineer"* — and knowing it exists is much better than the usual alternative, which is disabling rules and forgetting to turn them back on.

---

### Part 10 — Action Groups: What Actually Happens

An **action group** is a reusable list of who gets notified and what gets triggered. Define it once, attach it to fifty alert rules, change the on-call email in one place.

#### Notifications vs Actions

| **Notifications** — tell a human | **Actions** — do something |
|---|---|
| Email | **Webhook** — post to any HTTPS endpoint |
| SMS 💳 | **Azure Function** — run arbitrary code |
| Voice call 💳 | **Logic App** — an orchestrated workflow |
| Azure mobile app push | **Automation Runbook** — e.g. restart a service |
| **Email Azure Resource Manager Role** — everyone with a role, no addresses to maintain | **ITSM** — raise a ticket in ServiceNow etc. |
| | **Secure webhook** — Entra-authenticated |

Email and webhook have generous free allowances; **SMS and voice are charged per message**, so we use email today.

That last notification type is quietly excellent: **Email Azure Resource Manager Role** notifies everyone holding a role — say, all Owners of the subscription — rather than a hard-coded list that rots the moment someone changes team. Day 17's RBAC, doing useful work on Day 19.

And the **actions** column is where this stops being notification and becomes automation. An alert that fires a Logic App can restart the app, scale out, open a ticket and post to Teams, with no human in the loop. That's the ceiling of what's possible here, and it's how mature teams handle the routine failures.

#### Hands-On: Create an Action Group ✅

1. **Monitor → Alerts → Action groups → + Create.** **✅**
2. **Basics:** **Resource group** `rg-day19-demo`, **Action group name** `ag-lwm-oncall`, **Display name** `LWM OnCall` (12 characters max — it prefixes SMS and emails). **✅**
3. **Notifications tab:** **Notification type** *Email/SMS message/Push/Voice* → **Name:** `Primary email` → tick **Email**, enter your address → **OK**. **✅**
4. Look at the other notification type in the dropdown — **Email Azure Resource Manager Role** — and note you could pick *Owner* or *Monitoring Reader*. **✅**
5. **Actions tab:** open the **Action type** dropdown and read the list — Webhook, Azure Function, Logic App, Automation Runbook, ITSM. **Add nothing.** **✅**
6. **Review + create → Create.** **✅**
7. **Check your inbox.** Azure sends a confirmation email immediately, which is also a free test that the action group works. **✅**

!!! tip "Build the action group first, always"
    Create action groups *before* alert rules, and name them by **who they reach**, not by what triggers them. `ag-platform-oncall` and `ag-app-team-email` are good names. `ag-cpu-alert` is a bad one, because in six weeks it'll be attached to nine rules that have nothing to do with CPU.

---

### Part 11 — Building Alerts That Actually Fire

Three alerts, in increasing order of cleverness. Two are free; I'll tell you clearly when we cross into the one that isn't.

#### Hands-On: A Metric Alert ✅ (free — within the 10-series allowance)

*"Tell me when my web app starts returning server errors."*

1. **App Service → Monitoring → Alerts → + Create → Alert rule.** The **Scope** is pre-filled with your app. **✅**
2. **Condition tab → Signal name:** search and select **`Http Server Errors`** (or `Http 5xx`). **✅**
3. **Alert logic:** **✅**

   | Field | Value |
   |---|---|
   | **Threshold** | Static |
   | **Aggregation type** | Total |
   | **Operator** | Greater than |
   | **Threshold value** | `0` |
   | **Check every** | 1 minute |
   | **Lookback period** | 5 minutes |

   Note the **Dynamic** threshold option next to Static — machine learning that models the metric's normal pattern and alerts on deviation. Genuinely useful for things with a daily rhythm where no fixed number is right. Read it, leave it on Static.

4. Note the **Preview** chart showing the metric with your threshold drawn on it — that's how you sanity-check a threshold before saving it. **✅**
5. **Actions tab → Select action group → `ag-lwm-oncall`.** **✅**
6. **Details tab:** **Severity** `2 - Warning`. **Name** `alert-app-5xx`. Under **Advanced options**, confirm **Enable upon creation** and tick **Automatically resolve alerts**. **✅**
7. **Review + create → Create.** **✅**
8. **Make it fire.** Visit a URL on your app that doesn't exist and will error — or, more reliably, **stop the App Service** (**Overview → Stop**), hit the URL a few times, then start it again. **✅**
9. Wait a few minutes, then **Monitor → Alerts.** Your alert should appear, and **an email should arrive.** Click into it: **Summary**, **History**, and the alert's own state. **✅**

    !!! note "Alerts are not instantaneous, and that's normal"
        Metric alerts typically fire within 1–5 minutes. There's metric ingestion latency, then evaluation latency. If you're waiting three minutes, nothing is broken.

        Also, **it takes 10–15 minutes after creating a resource before its metrics are available at all**, which is a real cause of "my brand new alert isn't working."

#### Hands-On: An Activity Log Alert ✅ (completely free)

*"Tell me when someone deletes something."* This is the alert I'd put in every subscription I owned.

1. **Monitor → Alerts → + Create → Alert rule.** **✅**
2. **Scope:** select your **subscription**. **✅**
3. **Condition → Signal type: Activity Log.** Search the signal list for **`Delete Resource Group`**. **✅**
4. **Actions:** `ag-lwm-oncall`. **✅**
5. **Details:** **Severity** `1 - Error`. **Name** `alert-rg-deleted`. **Create.** **✅**

   **This rule costs nothing to run, ever.** Activity log alerts are free, which makes them the best value in Azure Monitor — and "someone deleted a resource group" is exactly the event you want to hear about within the minute.

#### Hands-On: A Log Search Alert ⚠️ (~$0.50/month — build it, then delete it)

*"Tell me when there are more than five failed requests in five minutes."* A metric can't easily express this, so it's a genuine log alert.

!!! warning "This one is not free"
    A log search alert rule costs roughly **$0.50 per month** for its first time series. That is small, and it is not zero, and it bills whether or not it ever fires. **We build it to learn the mechanics and delete it in cleanup.** Don't leave a shelf of forgotten log alert rules behind you.

1. **Log Analytics workspace → Logs**, and get the query right *first* — always build the query before the rule: **✅**

   ```kusto
   AppServiceHTTPLogs
   | where TimeGenerated > ago(5m)
   | where ScStatus >= 400
   | summarize FailedRequests = count()
   ```

2. Run it, confirm it returns a number. **✅**
3. Click **+ New alert rule** in the Logs toolbar — the query carries across. **✅**
4. **Condition:** **Measurement** → *Table rows*, **Aggregation type** *Count*. **Alert logic:** Greater than `5`. **Frequency of evaluation:** 5 minutes. **✅**
5. **Actions:** `ag-lwm-oncall`. **Details:** Severity `2`, name `alert-app-errors`. **Create.** **✅**
6. Note in the wizard where you'd configure **split by dimensions** — and remember each resulting series is billed separately. This is how a $0.50 rule quietly becomes a $20 one. **✅**

!!! tip "The rule of thumb worth remembering"
    **If a metric can answer the question, use a metric alert.** Faster, effectively free, simpler to reason about.

    **Use a log alert when the question needs the query language** — correlating across fields, counting distinct users, matching on text, joining two tables. Powerful, slower, and billed per rule.

---

### Part 12 — Application Insights

Everything so far has monitored **infrastructure**. Application Insights monitors **the application** — and it answers a completely different class of question. Not *"is the server up?"* but *"why is checkout slow for some users, and which line of code is throwing?"*

#### What It Gives You

- **Request rates, response times and failure rates**, per endpoint
- **Dependency tracking** — every outbound call your app makes to SQL, Storage, Key Vault or an external API, timed individually. **This is usually where the answer is.** "The app is slow" is nearly always "one dependency is slow."
- **Exceptions** with stack traces
- **The Application Map** — an auto-generated diagram of your components and their dependencies, annotated with failure rates and latency. It draws your architecture from real traffic, which is often more accurate than the diagram in your wiki.
- **End-to-end transaction details** — one user's request followed across every service it touched
- **Live Metrics** — a real-time stream with roughly a one-second lag, invaluable during a deployment
- **Availability tests** — synthetic requests from Azure regions worldwide, alerting when your site is unreachable from somewhere

#### Two Things That Changed

Both worth knowing, because they invalidate a lot of older material:

- **Classic Application Insights was retired on 29 February 2024.** Every Application Insights resource is now **workspace-based** — its data lives in a Log Analytics workspace. Which is excellent news, because it means you can **query application telemetry and infrastructure logs in the same KQL query**, and it's why App Insights bills as workspace ingestion rather than separately.
- **Instrumentation keys are out; connection strings are in.** Support for instrumentation-key ingestion ended **31 March 2025**. If you see `InstrumentationKey=...` alone in a config file, that's the old way. **Use the connection string.**

#### Hands-On: Turn It On ✅

Auto-instrumentation means no code changes at all.

1. **App Service → Settings → Application Insights → Turn on Application Insights.** **✅**
2. **Create new resource** → name `appi-lwm-day19`. **Crucially, set the Log Analytics workspace to `law-lwm-day19`** — put it in the workspace you already have rather than letting it create a new one. **✅**
3. **Apply → Yes.** The app restarts. **✅**
4. Go and hammer your app's URL again — thirty refreshes, and try a URL that 404s. **✅**
5. Open **`appi-lwm-day19`** and work through the left menu: **✅**
   - **Application Dashboard** — the auto-built overview
   - **Live Metrics** — refresh your site with this open and watch requests appear in real time
   - **Failures** — grouped by response code and exception type
   - **Performance** — response times by operation, with a distribution rather than just an average
   - **Application map** — thin today with one component, but this is the blade that makes large systems comprehensible
6. Now the payoff of workspace-based App Insights. Go to the **Log Analytics workspace → Logs** and run: **✅**

   ```kusto
   requests
   | where timestamp > ago(1h)
   | summarize Count = count(), AvgDuration = avg(duration) by name, resultCode
   | order by Count desc
   ```

   **Application telemetry, in the same workspace, in the same query language as your infrastructure logs.** Note the App Insights tables are lowercase — `requests`, `exceptions`, `dependencies`, `traces`, `customEvents`, `pageViews`.

7. And the thing that's genuinely hard to do any other way — correlate the two: **✅**

   ```kusto
   requests
   | where timestamp > ago(1h)
   | summarize AppRequests = count() by bin(timestamp, 5m)
   | render timechart
   ```

   Run that beside your `AppServiceHTTPLogs` query from Part 6. **Same time buckets, two independent views of the same traffic** — one from the platform, one from inside the application. When those two disagree, you've learned something important.

!!! tip "Availability tests are the cheapest real monitoring you can buy"
    Under **Application Insights → Availability**, you can create a **standard test** that hits your URL from multiple Azure regions on a schedule and alerts when it fails.

    It catches the failure mode that internal monitoring is structurally blind to: **the app is fine, and users can't reach it.** DNS broke, the certificate expired, a firewall rule changed, the CDN is misconfigured. Every metric inside Azure looks perfect and the site is down.

    There's a small charge per test, so it's not part of today's lab — but it's one of the first things I'd add to anything with real users.

---

### Part 13 — Visualising It: Dashboards, Workbooks and Insights

Four ways to put this on a screen, and knowing which to reach for is the whole point.

| | Best for | Interactive? |
|---|---|---|
| **Azure Dashboard** | A fixed at-a-glance view — pin charts from anywhere in the portal | Barely |
| **Workbook** | **Interactive, parameterised reports** — dropdowns, tabs, drill-downs, mixed text and data | **Very** |
| **Insights** | Prebuilt, curated experiences per service — VM Insights, Container Insights, Storage Insights | Prebuilt |
| **Managed Grafana** 💳 | Teams who already live in Grafana, or multi-cloud dashboards | Very |

**Dashboards** are the simple option: click *Pin to dashboard* from any metric chart or query result, drag the tiles around, share it. Perfect for a wall screen.

**Workbooks** are the powerful one and the one worth investing an hour in. A workbook combines text, parameters, metric charts, log queries and visualisations into a document that responds to input — pick a resource from a dropdown, pick a time range, and every chart updates. They're shareable, exportable as ARM templates, and version-controllable.

The reason they matter operationally: a workbook can encode an **investigation**, not just a picture. *"Here is our checkout latency runbook: pick the region, pick the window, and these six queries run in the order we'd want to run them."* That turns one engineer's expertise into something the whole team can use at 3am.

And **Azure ships dozens of them free** — every workspace and most services come with a gallery of prebuilt workbooks.

#### Hands-On: Both ✅

1. **Log Analytics workspace → Workbooks.** Browse the gallery. Open **Workspace Usage** and explore it. **✅**
2. **+ New → + Add → Add text**, type `## Day 19 Monitoring`, then **Done Editing**. **✅**
3. **+ Add → Add query.** Paste: **✅**

   ```kusto
   AppServiceHTTPLogs
   | where TimeGenerated > ago(24h)
   | summarize Requests = count() by bin(TimeGenerated, 15m)
   ```

   Set **Visualization** to *Area chart*, then **Run Query → Done Editing**.

4. **+ Add → Add parameter** → name `TimeRange`, type **Time range picker**. Then edit your query to use `{TimeRange}` instead of `ago(24h)`. **Now the chart responds to the dropdown.** That's the difference between a workbook and a dashboard. **✅**
5. **Save** as `Day 19 — App Monitoring` in `rg-day19-demo`. **✅**
6. For contrast: go back to **App Service → Metrics**, build a chart, and **Pin to dashboard**. Then open **Dashboard** from the portal's left menu and see your tile. **✅**

!!! tip "Which one, in one line"
    **Dashboard** = a fixed picture on a wall. **Workbook** = an interactive report you investigate with. **Insights** = the prebuilt one, always check whether it already exists before building your own.

---

### Part 14 — Cost Control, Gotchas and Interview Prep

#### Check What Today Cost You

1. **Log Analytics workspace → Settings → Usage and estimated costs.** Empty three hours ago, populated now. **✅**
2. Run the billing query and see exactly where your (tiny) volume went: **✅**

   ```kusto
   Usage
   | where TimeGenerated > ago(7d)
   | where IsBillable == true
   | summarize BillableGB = sum(Quantity) / 1000 by DataType
   | order by BillableGB desc
   ```

3. **Set a daily cap while you're here** — **Usage and estimated costs → Daily cap → On**, set something small like 1 GB. **✅**

   Then remember the rule from Part 7: **a daily cap that is hit stops ingestion, which means it blinds you.** Always pair it with an alert on the cap being reached. It's a runaway guard, not a budgeting tool.

#### The Gotchas That Waste Real Days

- **Logs are off by default and not retroactive.** The single biggest one. No diagnostic setting on Tuesday means no Tuesday data, ever. Use **Azure Policy `DeployIfNotExists`** to enforce diagnostic settings at scale.
- **The 5 GB free grant is per month, per billing account.** Not per day, not per workspace. This is *the* Azure Monitor billing misconception.
- **New resources take 10–15 minutes before metrics appear.** Your alert isn't broken.
- **First-time log ingestion lags 5–15 minutes.** Neither is your diagnostic setting.
- **Average hides spikes.** Alert on **Max** when you care about peaks.
- **Stateless alerts spam you.** Tick **Automatically resolve alerts**.
- **Splitting an alert by dimensions multiplies its cost.** Each time series is billed.
- **Basic and Auxiliary tables charge per GB scanned to query.** Cheap to store, expensive to read — the wrong plan for data you query often.
- **The Log Analytics agent is retired**, the diagnostics extension is deprecated, **and the Data Collector API retires 14 September 2026.** If you're following a guide that uses any of them, the guide is out of date.
- **Alert fatigue is a real outage cause.** If everything is Sev 0, nothing is.

#### Interview and Exam Quick Reference

| If you're asked… | The answer is… |
|---|---|
| "Metrics vs logs?" | **Metrics: numeric time series, free, 93 days, near real-time. Logs: structured records in a workspace, billed per GB ingested, retention you choose** |
| "What does platform metric collection cost?" | **Nothing.** Same for the activity log |
| "Free Log Analytics allowance" | **5 GB per billing account per month** — *not* per day |
| "How do I get a resource's logs into a workspace?" | **A diagnostic setting.** Same mechanism for every Azure service |
| "Why are last month's logs missing?" | **Diagnostic settings are off by default and not retroactive.** The data was never collected |
| "Enforce diagnostic settings across a subscription" | **Azure Policy with `DeployIfNotExists`** |
| "Diagnostic setting destinations" | **Log Analytics, Storage account, Event Hub, partner solution** |
| "Stream Azure logs to Splunk/Datadog" | **Event Hub** |
| "Cheap long-term log archive" | **Storage account** destination, or **Auxiliary** table plan + long-term retention |
| "Query language" | **KQL** — same in Sentinel, Defender, Resource Graph, Data Explorer |
| "Who deleted the resource group?" | **Activity log** — the `Caller` field; or `AzureActivity` in KQL |
| "Collect the Windows Event Log from 200 VMs" | **Azure Monitor Agent + data collection rule**, deployed by Azure Policy |
| "Log Analytics agent / MMA" | **Retired 31 August 2024.** AMA only |
| "Custom log push via shared key" | **Data Collector API — retires 14 September 2026.** Use the DCR-based **Logs Ingestion API** |
| "Reduce ingestion volume at source" | **DCR transformations** — filter rows/columns before storage, so before billing |
| "CPU above 80% for 5 minutes" | **Metric alert** |
| "More than 10 errors in 5 minutes" | **Log search alert** |
| "Someone deleted a resource" | **Activity log alert** — and it's **free** |
| "Is Azure itself having an incident?" | **Service health alert** — free |
| "Cheapest alert type" | **Activity log / service health / resource health — no charge** |
| "Metric alert free allowance" | **10 time series**, then $0.10 each per month |
| "Alert fired 400 times for one problem" | **Stateless.** Enable *Automatically resolve alerts* |
| "Threshold that adapts to normal behaviour" | **Dynamic thresholds** |
| "Suppress alerts during maintenance" | **Alert processing rule** |
| "Notify everyone with a subscription role" | Action group → **Email Azure Resource Manager Role** |
| "Auto-remediate when an alert fires" | Action group → **Logic App / Azure Function / Automation Runbook** |
| "Why is my app slow?" | **Application Insights** — dependency tracking, then the Application Map |
| "Application Insights resource type" | **Workspace-based.** Classic retired 29 February 2024 |
| "Instrumentation key or connection string?" | **Connection string.** Key ingestion support ended 31 March 2025 |
| "Alert when the site is unreachable from outside" | **Application Insights availability test** |
| "Interactive parameterised report" | **Workbook.** A dashboard is a fixed view |
| "Where is my monitoring money going?" | The **`Usage`** table, or *Usage and estimated costs* |
| "Hard stop on ingestion spend" | **Daily cap** — and alert on hitting it, because it blinds you |

---

## Cleanup ✅

1. **Delete the log search alert rule** — the only recurring charge you created: **Monitor → Alerts → Alert rules → `alert-app-errors` → Delete**. **✅**
2. **Delete the other alert rules:** `alert-app-5xx` and `alert-rg-deleted`. **✅**
3. **Delete the subscription-level diagnostic setting** — this one lives on the *subscription*, so deleting the resource group will **not** remove it: **Monitor → Activity log → Export Activity Logs → `activity-to-law` → Delete**. **✅**

    !!! warning "Don't skip step 3"
        It's the same trap as yesterday's soft-deleted vault: the resource group delete looks like cleanup and leaves something behind, in a blade you weren't looking at. A diagnostic setting pointing at a deleted workspace is harmless but messy — and if you'd pointed it at a workspace you kept, it would quietly keep ingesting and billing.

4. **Delete the resource group `rg-day19-demo`.** This removes the App Service, the plan, the VM, its disk, NIC and public IP, the data collection rule, the workspace, Application Insights, the action group and the workbook. **✅**

    !!! warning "Stopping the VM is not the same as deleting it"
        If you want to keep the lab for another session, **deallocating the VM stops the compute charge but not the rest of it** — the OS disk and the public IP keep billing whether the machine runs or not. That's roughly $0.10 a day for something switched off.

        The data collection rule costs nothing on its own, but the agent keeps ingesting for as long as the VM is running. Delete the group and all of it goes at once.
5. **Keep `Priya Sharma` and `grp-finance-team`** — Day 30's capstone uses them. **✅**
6. **Keep `db-lwm-demo`** — Day 30 needs it. **✅**

---

## Summary

Today you stopped building and started **watching**, and the first thing you found was that Azure had been watching all along for free.

**Metrics and the activity log cost nothing, are already on, and are already holding history** — 93 days of numbers and 90 days of "who did what" on every resource you have ever created in this course. Most beginners never look. Metrics are the fastest answer to "is it broken and when did it start," and the activity log's `Caller` field is the answer to "who did that," which is a question that arrives at exactly the moment nobody wants to be guessing.

**Logs are the part that costs money, and it's worth being precise about it.** Everything is routed by a **diagnostic setting** — one mechanism, identical for every Azure service — into a **Log Analytics workspace**, billed per GB ingested, with **5 GB free per billing account per month**. Not per day. That one misreading is behind an enormous number of surprise Azure bills, and you now won't make it.

And because logs are off by default and **not retroactive**, "turn on diagnostic settings" belongs in your build checklist, not your incident response. The scalable version of that sentence is **Azure Policy with `DeployIfNotExists`** — Day 17's governance lesson doing real work here.

**KQL is the skill you'll actually use.** Table, then a pipeline: `where`, `project`, `summarize`, `bin`, `render`. Five operators cover most real work, and the same language runs Sentinel, Defender and Resource Graph. Filter early. Run `take 10` before you write anything clever.

Then the platform started working for you. **Metric alerts are fast and effectively free** — ten time series before anyone charges you. **Activity log alerts are completely free**, which makes *"tell me when someone deletes a resource group"* the best-value alert in Azure. **Log alerts cost about fifty cents a month each**, so use them when the question genuinely needs a query and not before. Attach them all to an **action group** named for who it reaches, tick **Automatically resolve alerts**, and mean it when you set Sev 0 — because **alert fatigue is a self-inflicted outage**.

**Application Insights** answered the different question: not *is the server up* but *why is it slow*, with dependency tracking that usually finds the answer, and — because it's workspace-based now — application telemetry sitting in the same KQL query as your infrastructure logs.

That's Phase 5 complete. Over three days you learned **who someone is** (Entra ID), **what they may do** (RBAC), **where the secrets live** (Key Vault), and **how to know what's actually happening** (Azure Monitor). That's the security and operations foundation, and it's the layer that separates "I can deploy this" from "I can run this."

### What's Next

**Day 20 — Azure DevOps Introduction**, and the whole shape of the course changes.

For nineteen days you've built things by clicking in a portal. That's the right way to learn — you can't automate what you don't understand. But it doesn't scale, it isn't repeatable, and it leaves no record of *why* anything is the way it is.

Phase 6 is where we stop clicking. Boards, Repos, Pipelines, Artifacts — the tooling that takes an idea from a work item, through source control, through an automated build and test, to a deployment nobody had to perform by hand. And when we get to Pipelines on Day 22, watch for two things you already own coming straight back: a **service principal with a federated credential** to authenticate the pipeline without a stored secret, and a **variable group linked to Key Vault** so the deployment reads its secrets the same way your App Service did today.

Everything from Phase 5 gets used. See you tomorrow.

---

## Key Takeaways

**Collect**

- **Azure Monitor is four data types:** metrics, logs, traces and changes. **Metrics and the activity log are free and already running**; logs are where the bill is.
- **Platform metrics: free, 93-day retention, near real-time.** Available on everything you've ever built, retroactively.
- **Activity log: free, 90 days, control plane only.** The `Caller` field answers "who did that?" Export it to a workspace to keep and query it properly.
- **Diagnostic settings are the universal routing mechanism** — same blade for every service, four destinations: Log Analytics, Storage, Event Hub, partner.
- **Resource logs are off by default and are never retroactive.** Enforce them with **Azure Policy `DeployIfNotExists`**.
- **The free Log Analytics grant is 5 GB per billing account per month.** Analytics-plan queries are free; **Basic and Auxiliary charge per GB scanned**.
- **Start with one workspace** per region per environment. Split for sovereignty, access isolation or retention — not per application.
- **Retention is two numbers:** interactive (first 31 days free, up to 2 years) and total/long-term (up to 12 years). **Configurable per table.**
- **KQL:** table then pipeline. `where` → `project` → `summarize` → `bin` → `render`. Filter early; `take 10` first.
- **AMA + data collection rules** is the only current agent story. **MMA retired 31 Aug 2024**, the diagnostics extension is deprecated, and **the Data Collector API retires 14 Sept 2026**. DCR **transformations** cut cost at ingestion.

**Detect and respond**

- **Metric alerts** — fast, **10 free time series**, then $0.10 each per month. Use these whenever a metric can express the question.
- **Log search alerts** — ~$0.50/month per rule. Use only when the question needs KQL. Splitting by dimensions multiplies the cost.
- **Activity log, service health and resource health alerts are free.** *"Someone deleted a resource group"* is the best-value alert in Azure.
- **Tick *Automatically resolve alerts*** or metric alerts are stateless and will spam you.
- **Dynamic thresholds** adapt to normal behaviour; **alert processing rules** suppress notifications during maintenance.
- **Action groups** separate *what fired* from *who hears about it*. Name them by audience. Email and webhook are effectively free; **SMS and voice are charged**. Actions can auto-remediate via Logic App, Function or Runbook.
- **Severity is a routing decision.** If everything is Sev 0, nobody reads any of it.
- **Application Insights is workspace-based** (classic retired 29 Feb 2024) and uses **connection strings**, not instrumentation keys (key ingestion ended 31 Mar 2025). Dependency tracking is usually where the answer is.
- **Workbook = interactive report. Dashboard = fixed view. Insights = the prebuilt one — check it exists before building your own.**
- **Watch the bill with the `Usage` table.** A **daily cap** stops runaway spend and blinds you, so alert on it being reached.
