# Day 16 — Azure SQL Database + Other Databases

**Phase 4 — Databases**

> Every application you've built or touched in this course so far has been missing something. The VM in Day 3 served a web page. The App Service in Day 6 hosted a site. The load balancer in Day 12 spread traffic across servers. But none of them **remembered** anything. Restart the VM and whatever the app was holding in memory is gone. Real applications — every single one — need somewhere to put data that survives a restart, a redeploy, a region failure, and the person who wrote the app leaving the company. Today we build that layer. And here's the part that surprises people coming from a traditional IT background: you are not going to install SQL Server. You're not going to patch it, back it up, configure high availability, or renew its licence. Azure does all of that. You get a connection string and a database, and you spend your time on schemas and queries instead of on the engine underneath them. By the end of today you'll have a live SQL database running on Azure's **free forever** offer, queried straight from your browser, secured, backed up, and restored from a backup — plus a clear map of when to reach for PostgreSQL or MySQL instead.

---

## What You'll Learn

- **Why managed databases exist** — the exact list of jobs Microsoft takes over, and the smaller list you keep
- **The Azure SQL family** — Single Database, Elastic Pool, Managed Instance, and SQL Server on a VM, and how to pick between them without guessing
- **Purchasing models** — DTU vs vCore, and why the industry has moved decisively toward vCore
- **Service tiers** — Basic/Standard/Premium and General Purpose/Business Critical/Hyperscale, explained by what actually differs underneath
- **The Serverless compute tier** — a database that pauses itself when nobody's using it, and why that's the single most beginner-friendly billing model Azure offers
- **The Azure SQL Database free offer** — a genuinely free database, not a 12-month trial, and exactly what its limits are
- **The logical server** — the most misunderstood object in Azure SQL, and why it is *not* a machine
- **Firewall rules, public access, and Private Endpoints** — the three layers that decide who can reach your database
- Hands-on: create a database, connect from the portal Query Editor, and run real SQL
- **Backups** — automatic backups, point-in-time restore, long-term retention, and geo-restore — including an actual restore
- **Security** — Microsoft Entra ID authentication, Transparent Data Encryption, auditing, and Dynamic Data Masking (with a live demo)
- **Performance** — scaling tiers, Query Performance Insight, and automatic tuning that writes your indexes for you
- **Business continuity** — active geo-replication, failover groups, and zone redundancy
- **The rest of the family** — Azure Database for PostgreSQL and MySQL Flexible Server, and when each one wins
- Exam framing: how AZ-104 and AZ-305 phrase database questions, and the keywords that give away the intended answer

---

## Before We Begin

Good news — **this is one of the cheapest days in the entire course.** Almost everything today is genuinely free.

- **Azure SQL Database — free offer:** Microsoft provides a free General Purpose Serverless database on **every** subscription type, not just free accounts and not just for 12 months. The allowance is roughly **100,000 vCore-seconds of compute per month**, **32 GB of data storage**, and **32 GB of backup storage**. When you exhaust the monthly compute allowance you choose in advance what happens: the database auto-pauses until the next month, or it keeps running and you get billed normally. Set it to **auto-pause** and your worst case is a paused database, not a bill. **✅ Free**
- **Azure free account SQL offer (separate thing):** if you're inside your first 12 months on a free account, you *also* get an S0 database (10 DTU, 250 GB) free for 12 months. You can use either. The free offer above is the better default because it doesn't expire. **✅ Free**
- **Query Editor, firewall rules, backups, point-in-time restore, Dynamic Data Masking, auditing, Query Performance Insight, automatic tuning:** all included, no separate charge. **✅ Free**
- **Microsoft Defender for SQL:** roughly **$15/server/month** after a free trial period. **💳 Instructor demo**
- **Active geo-replication / failover groups:** you pay full price for the secondary database, so this doubles your cost. **💳 Instructor demo**
- **Elastic Pools and Managed Instance:** pools start well above free-tier territory; Managed Instance starts around **$700+/month** and takes hours to provision. **💳 Conceptual + portal tour only** — nobody should deploy a Managed Instance to follow along, including me.
- **PostgreSQL / MySQL Flexible Server:** free account holders get 12 months of a B1ms burstable instance with 32 GB storage. Outside that, a burstable instance is a few dollars a month. **✅ / 💳**

!!! warning "The one thing that can actually cost you money today"
    Do not create a **Managed Instance** "just to look at it." It provisions for 4–6 hours, bills the entire time, and cannot be stopped — only deleted. We tour it in the portal instead. Everything else today, you can safely click.

**Set this up before you start:** a resource group called `rg-day16-demo`. Everything today goes in it, and we delete the whole group at the end.

---

## Part 1 — Why Managed Databases Exist

### The Problem

Imagine doing this the traditional way. You'd take the Ubuntu or Windows VM you built on Day 3, install a database engine on it, and then own the following forever:

- Installing and configuring the database engine
- Patching the operating system every month
- Patching the database engine when security fixes ship
- Configuring backups — and, far more importantly, **testing that they restore**
- Building high availability: a second server, replication between them, automatic failover
- Monitoring disk space before it fills at 3am
- Sizing the VM correctly, then resizing it when you sized it wrong
- Buying and tracking SQL Server licences

That is a full-time job. It's a real job that real people have. But it has nothing to do with your application — it's pure overhead, and every hour spent on it is an hour not spent on the thing your users actually pay for.

**Azure SQL Database is that entire list, handed to Microsoft.**

### What Microsoft Runs vs What You Run

This split is the single most important idea today, and it's worth memorising because exam questions are built directly on it:

| Microsoft handles | You handle |
|---|---|
| Operating system — install, patch, secure | Schema design — tables, indexes, relationships |
| Database engine — install, patch, upgrade | Queries and stored procedures |
| Automatic backups (every few minutes) | Choosing the right service tier and size |
| High availability and automatic failover | Firewall rules and network access |
| Hardware, storage, and replacing failed disks | Users, permissions, and authentication |
| Encryption at rest, on by default | Connection strings in your application |
| Underlying licensing | Your data itself |

Notice the pattern: Microsoft owns **infrastructure and availability**. You own **data and access**. That line never moves.

### PaaS vs IaaS, in Database Terms

You already met this distinction on Day 6 with App Service. It's identical here:

- **SQL Server on an Azure VM** = IaaS. You get a VM with SQL Server installed. Full control, full responsibility, full OS access. Choose this when you need something the managed service won't let you do — a specific SQL Server build, a third-party agent installed on the OS, cross-database transactions with a legacy app, or SQL Server Reporting Services running alongside.
- **Azure SQL Database** = PaaS. You get a database. No OS, no server to log into, no patching. Choose this for basically everything new.

**The rule of thumb:** start with Azure SQL Database. Move to a VM only when you hit a specific, nameable blocker. "I might need OS access one day" is not a blocker.

### Hands-On: Tour the Azure SQL Landing Page ✅

Before creating anything, let's see the whole family in one screen.

1. In the portal search bar, type **Azure SQL** and open it. **✅**
2. Click **+ Create**. You land on the **Select SQL deployment option** page — this single screen is the entire product family. **✅**
3. Look at the three cards:
   - **SQL databases** — with a dropdown for *Single database* or *Elastic pool*
   - **SQL managed instances** — with a dropdown for *Single instance* or *Instance pool*
   - **SQL virtual machines** — a gallery of pre-built SQL Server VM images
4. Click **Show details** on each card and read the descriptions. Don't create anything yet — just register that these are four genuinely different products sharing a name. **✅**

Stay on this page. We're coming back to it in Part 2.

---

## Part 2 — The Azure SQL Family: Four Products, One Name

### Single Database

One database, with its own dedicated allocation of compute and storage, completely isolated from everything else. This is the default and the one you'll use 90% of the time.

Every single database lives inside a **logical server** (much more on that in Part 4). You can scale it up or down independently, back it up independently, and restore it independently.

**Use it for:** almost any new application. One app, one database.

### Elastic Pool

Now imagine you're running a SaaS product with 200 customers, and each customer gets their own database for isolation. Buying 200 individually-sized databases is enormously wasteful, because at any given moment most of those customers are asleep and using nothing.

An **Elastic Pool** gives you one shared bucket of compute that all 200 databases draw from. Customer A spikes at 9am, customer B spikes at 3pm, and they share the same purchased capacity instead of each paying for their own peak.

Key mechanics:
- You buy the pool (in eDTUs or vCores), not the individual databases
- Databases move in and out of a pool freely, with no downtime
- You can set per-database min/max limits so one noisy tenant can't starve the others
- Pools become cost-effective at roughly **15+ databases** with uncorrelated usage patterns

**Use it for:** multi-tenant SaaS, or any situation with many databases whose busy periods don't line up.

**The trap to avoid:** if all your databases peak at the same time, a pool saves you nothing — you'll just size the pool for the combined peak. Pools exploit *variation*, not volume.

### Managed Instance

Managed Instance is for one specific scenario: **you have an existing SQL Server and you want to move it to Azure without rewriting it.**

Single Database is a managed *database*. Managed Instance is a managed *SQL Server instance* — it gives you back the instance-scoped features that Single Database doesn't have:

- **SQL Server Agent** — scheduled jobs
- **Cross-database queries and transactions** — joining across databases on the same instance
- **CLR (Common Language Runtime)** — .NET code inside the database
- **Service Broker**, **Database Mail**, **linked servers**
- Near **100% surface-area compatibility** with on-premises SQL Server

It always deploys **inside your VNet** with a private IP — there's no public endpoint by default. Everything you learned on Days 10 and 11 about VNets and subnets applies directly.

The cost: it starts around **$700+/month**, takes **4–6 hours** to provision, and needs a dedicated subnet with specific delegation. It's a serious enterprise product.

**Use it for:** lift-and-shift migration of existing SQL Server workloads. Not for new applications.

### SQL Server on Azure Virtual Machines

Pure IaaS — a VM from Day 3 with SQL Server pre-installed and licensing handled through the image. You get root/administrator access and total control, and you own everything from the OS up.

The one feature worth knowing: the **SQL Server IaaS Agent Extension** adds portal-managed automated backups and patching, which softens the management burden somewhat.

**Use it for:** when you need OS access, an unsupported SQL Server feature, or an exact version match with something on-premises.

### The Decision Table

Memorise this. AZ-104 and AZ-305 questions are built from it:

| If the question says… | The answer is… |
|---|---|
| "New application, minimal management overhead" | **Single Database** |
| "Many databases, variable and unpredictable usage" | **Elastic Pool** |
| "Migrate existing SQL Server with minimal changes" / "SQL Server Agent" / "cross-database queries" | **Managed Instance** |
| "Requires OS-level access" / "specific SQL Server version" / "third-party agent on the server" | **SQL Server on a VM** |
| "Unpredictable, intermittent workload" / "dev/test that sits idle" | **Single Database, Serverless tier** |
| "Up to 100 TB" / "very large database" / "fast restore regardless of size" | **Hyperscale tier** |

### Hands-On: Compare Deployment Options in the Portal ✅

1. Back on the **Select SQL deployment option** page, open the **Resource type** dropdown under *SQL databases*. **✅**
2. Switch between **Single database** and **Elastic pool** and note that the creation blade changes — pools ask you to size the pool, not the database. **✅**
3. Click into **SQL managed instances → Show details** and note the pricing estimate and the VNet requirement. Then back out **without creating it**. **✅**
4. Click into **SQL virtual machines** and browse the image gallery — notice you're choosing a *VM image* here, which tells you everything about which layer you're operating at. **✅**

---

## Part 3 — Purchasing Models, Service Tiers, and Serverless

This is where most beginners get lost, because Azure gives you two completely different ways to buy the same thing. Let's untangle it.

### DTU — The Bundled Model

A **DTU (Database Transaction Unit)** is a blended unit combining CPU, memory, and I/O into one number. You don't choose how much CPU or how much memory — you choose "how much database," and Microsoft decides the mix.

DTU tiers:

| Tier | DTU range | Max storage | Typical use |
|---|---|---|---|
| **Basic** | 5 | 2 GB | Tiny apps, learning |
| **Standard** | S0–S12 (10–3000) | 250 GB – 1 TB | Most production workloads |
| **Premium** | P1–P15 (125–4000) | Up to 4 TB | High I/O, low latency, local SSD |

**The advantage:** it's simple. One slider, one price.
**The disadvantage:** you can't tune it. If you're memory-starved but CPU-idle, your only option is to buy more of *everything*.

DTU is a legacy model. It still exists, it still works, and it appears on exams — but Microsoft's investment goes into vCore.

### vCore — The Transparent Model

**vCore** lets you choose your CPU count and memory independently, and pick your hardware generation. It costs a little more mental effort and gives you far more control.

Why vCore has won:

- **Azure Hybrid Benefit** — if you already own SQL Server licences with Software Assurance, you can apply them here and cut compute costs substantially. This is not available on DTU.
- **Reserved capacity** — commit to 1 or 3 years for a large discount.
- **Serverless and Hyperscale are vCore-only** — the two most interesting tiers don't exist in DTU at all.
- Independent scaling of compute and storage.

vCore service tiers — and here's what actually differs underneath, which is the part most explanations skip:

| Tier | Storage architecture | Replicas | Max size | Best for |
|---|---|---|---|---|
| **General Purpose** | Remote premium storage, separated from compute | 1 primary + remote storage redundancy | 4 TB | Most workloads — the sensible default |
| **Business Critical** | **Local SSD** attached to the compute node | 3 replicas + **1 free readable secondary** | 4 TB | Low-latency, high-I/O, read-heavy workloads |
| **Hyperscale** | Distributed storage with page servers and a log service | Up to 4 readable secondaries | **100 TB** | Very large databases; near-instant restore regardless of size |

The single most useful detail in that table: **Business Critical gives you a free read-only replica.** Point your reporting queries and dashboards at it and they stop competing with your transactional workload. That's a real architectural lever, and it's exactly the kind of thing AZ-305 asks about.

And the Hyperscale detail worth knowing: because of how its storage works, **restore time doesn't scale with database size**. A 50 TB Hyperscale database restores in minutes. A 50 TB General Purpose database would take many hours. That's a decisive difference for large systems.

### Serverless — The Tier That Makes This Day Free

Standard compute is **provisioned**: you pick a size and pay for it 24/7, whether or not anyone connects.

**Serverless** works differently:

- You set a **min and max vCore range**, and the database scales inside it automatically based on demand
- You're billed **per vCore-second of actual use**, not per hour of existence
- After a configurable idle period (minimum **1 hour**), the database **auto-pauses**
- While paused, **you pay nothing for compute** — storage only
- The next connection **auto-resumes** it, taking roughly a minute

That resume delay is the whole trade-off. Your first query after an idle period will time out or hang for about a minute while the database wakes up. Your application needs retry logic to handle it gracefully.

**Serverless is the right choice for:** development, testing, learning, internal tools, and any genuinely intermittent workload.
**Serverless is wrong for:** a customer-facing production app that must respond instantly at any hour — the auto-pause resume delay is unacceptable there.

For this course, Serverless with auto-pause is perfect, and it's exactly what the free offer uses.

### Hands-On: Create Your Azure SQL Database on the Free Offer ✅

This is the main build of the day. Take your time.

1. From **Azure SQL → + Create → SQL databases (Single database) → Create**. **✅**
2. On the **Basics** tab:
   - **Subscription:** yours
   - **Resource group:** `rg-day16-demo`
   - **Database name:** `db-lwm-demo`
3. **Server** — click **Create new**. This is the logical server, which we'll dissect in Part 4. Fill in: **✅**
   - **Server name:** `sql-lwm-<yourname>` — this must be **globally unique**, because it becomes `sql-lwm-<yourname>.database.windows.net`
   - **Location:** pick a region near you (I'm using Central India)
   - **Authentication method:** choose **Use both SQL and Microsoft Entra authentication**. This is the best of both — you get an Entra admin *and* a SQL login, and we'll use both later today.
   - **Set Entra admin:** click it and select your own account
   - **Server admin login:** `sqladmin`
   - **Password:** something strong — **write it down now**, you'll need it in the Query Editor and it cannot be recovered, only reset
   - Click **OK**
4. Back on Basics — **look for the free offer banner.** Azure surfaces a prompt like *"Want to try Azure SQL Database for free?"* with an **Apply offer** link. **Click it.** This is the whole reason today is free. **✅**
   - If you don't see the banner, continue to step 5 and set the tier manually — the free offer is normally applied on the **Compute + storage** blade.
5. **Want to use SQL elastic pool?** → **No**. **✅**
6. **Workload environment:** **Development**. This nudges the defaults toward cheaper settings. **✅**
7. **Compute + storage** → click **Configure database**: **✅**
   - **Service tier:** *General Purpose — Serverless*
   - **Hardware:** Standard-series (Gen5) is fine
   - **Max vCores:** 1 (or 2 — this is a ceiling, not a commitment)
   - **Min vCores:** 0.5
   - **Enable auto-pause:** **Yes**, delay **1 hour** — this is what makes it cost nothing when idle
   - **Data max size:** 32 GB
   - Apply
8. **Backup storage redundancy:** **Locally-redundant** for this demo — geo-redundant costs more and we don't need it. **✅**
9. **Next: Networking**: **✅**
   - **Connectivity method:** **Public endpoint**
   - **Allow Azure services and resources to access this server:** **Yes**
   - **Add current client IP address:** **Yes** ← without this, you cannot connect from your own machine
   - **Connection policy:** Default
   - **Minimum TLS version:** 1.2
10. **Next: Security** — leave the defaults for now. We'll come back and turn things on deliberately in Part 7. **✅**
11. **Next: Additional settings**: **✅**
    - **Use existing data:** select **Sample** ← **do this.** It loads the AdventureWorksLT sample database, so you have real tables to query in about two minutes instead of typing test data by hand.
    - **Collation:** leave default
12. **Review + create → Create**. Deployment takes **2–5 minutes**. **✅**

While that deploys, let's talk about what you just created — because the "server" is not what you think it is.

---

## Part 4 — The Logical Server: The Most Misunderstood Object in Azure SQL

### It Is Not a Machine

You just created something called a "server" at `sql-lwm-<yourname>.database.windows.net`. Here is the critical point:

**There is no server.** You cannot RDP into it. You cannot SSH into it. It has no operating system, no CPU allocation of its own, and no cost of its own. Creating a logical server is **completely free** — you're only billed for the databases inside it.

A logical server is an **administrative and networking container**. It provides:

- A **globally unique DNS endpoint** — `<name>.database.windows.net`
- A **shared firewall** — server-level rules apply to every database inside it
- **Administrative accounts** — the SQL admin login and the Microsoft Entra admin
- A **security boundary** — auditing, Defender for SQL, and Entra configuration are set here
- A **region** — every database inside it lives in that region

The databases inside a single logical server can be **completely different sizes and tiers**. You can have a Basic database and a Business Critical database on the same logical server. They don't share compute — only the endpoint and the security configuration.

**Why this matters practically:** when you troubleshoot "I can't connect," the answer is almost always at the *server* level (firewall or authentication), not the database level. And when someone says "our SQL server is expensive," they mean the databases — the server itself is free.

### Firewall Rules — The Three Layers

Azure SQL is protected by default: **a brand-new logical server rejects every connection from everywhere**, including from you. You have to explicitly open it.

There are three distinct controls, and confusing them is the classic beginner mistake:

**1. Server-level firewall rules** — IP ranges that can reach *any* database on this server. Set in the portal under the server's **Networking** blade. This is the common case.

**2. Database-level firewall rules** — IP ranges that can reach *one specific* database. These can only be created with T-SQL (`sp_set_database_firewall_rule`), not in the portal. Use them when you want tighter isolation than the server-wide rule allows. Worth knowing they exist; rarely needed.

**3. "Allow Azure services and resources to access this server"** — a toggle that sounds harmless and isn't. It creates a rule for the IP range `0.0.0.0`, which means **any Azure resource in any subscription** — including other people's — can attempt a connection. They still need valid credentials, so it's not an open door, but it's a much wider surface than most people realise when they flip it on.

!!! danger "Use the Azure-services toggle deliberately"
    Turn it on for a demo or for quick App Service connectivity. For production, turn it **off** and use a **Private Endpoint** instead. If an exam question mentions "no public internet exposure" or "traffic must stay on the Microsoft backbone," Private Endpoint is the answer every time.

### Private Endpoints — Bringing the Database Into Your VNet

You built these on Day 11. The concept is identical here: a **Private Endpoint** gives your SQL logical server a **private IP address inside your VNet**. Traffic from your VMs and App Services then reaches the database over Azure's private backbone, never touching the public internet.

When you enable it, Azure creates a Private DNS zone (`privatelink.database.windows.net`) so that `sql-lwm-<yourname>.database.windows.net` resolves to the *private* IP from inside your VNet — the connection string doesn't change at all. That's the elegance of it.

The full pattern for a locked-down production database:

1. Create a Private Endpoint into your application's VNet
2. Set **Public network access** to **Disabled**
3. Application connects over the private IP; the database is unreachable from the internet entirely

### Hands-On: Explore and Configure Server Networking ✅

Your database should be deployed by now.

1. Open your database `db-lwm-demo`. On the **Overview** page, find **Server name** at the top right and copy it. Note the **Pricing tier** shows *General Purpose: Serverless* and the status shows **Online**. **✅**
2. Click the **server name** link to jump up to the logical server. Look at the Overview — notice there's no CPU metric, no size, no OS. It's a container. **✅**
3. Go to **Security → Networking**. **✅**
4. On the **Public access** tab, review: **✅**
   - **Public network access** is set to *Selected networks*
   - Your client IP rule is listed — this was added during creation
   - The **Exceptions** checkbox for *Allow Azure services* is ticked
5. Click **+ Add your client IPv4 address** if your IP has changed since creation, then **Save**. **✅**
6. Now demonstrate the failure mode deliberately — this teaches more than success does: **✅**
   - Delete your client IP rule and **Save**
   - Go back to the database → **Query editor** → try to sign in
   - You'll get an error naming your IP address and telling you it's not allowed. **This exact error is the #1 Azure SQL support question in the world.** Now you'll recognise it instantly.
   - Re-add your IP and **Save**
7. Click the **Private access** tab and read the Private Endpoint option. **We're not creating one today** (it needs a VNet and adds cost), but note that this is where you'd wire up exactly what you built on Day 11. **✅**

---

## Part 5 — Connecting and Querying

### How Applications Connect

Your database exposes a standard **TDS** endpoint on **port 1433**. Anything that speaks SQL Server can connect to it:

- **Query Editor** — built into the Azure Portal, no installation, works from a browser. This is what we'll use.
- **SQL Server Management Studio (SSMS)** — the full Windows tool, most powerful option
- **Azure Data Studio / VS Code with the MSSQL extension** — cross-platform, lighter
- **Your application** — via a connection string

The portal hands you ready-made connection strings under **Settings → Connection strings**, with tabs for ADO.NET, JDBC, ODBC, PHP, and Go.

!!! tip "Never paste a connection string into your source code"
    The connection string contains your database password. It belongs in **Azure Key Vault** (Day 19) or in an App Service application setting — never in a file that gets committed to Git. Better still: use a **managed identity** so there's no password at all. We'll build that on Day 17.

### Hands-On: Query Your Database from the Browser ✅

1. Open `db-lwm-demo` → **Query editor (preview)** in the left menu. **✅**
2. Sign in. You have two options, and **try both** so you understand the difference: **✅**
   - **SQL authentication:** login `sqladmin` and the password you saved
   - **Microsoft Entra authentication:** click *Continue as <your account>* — this works because you set yourself as Entra admin during creation
3. Expand **Tables** in the left pane. Because you loaded the sample data, you'll see the `SalesLT` schema with `Customer`, `Product`, `SalesOrderHeader`, and others. **✅**
4. Run a simple read: **✅**
   ```sql
   SELECT TOP 10 * FROM SalesLT.Product;
   ```
5. Something more realistic — a join and an aggregate: **✅**
   ```sql
   SELECT c.CompanyName,
          COUNT(soh.SalesOrderID) AS OrderCount,
          SUM(soh.TotalDue)       AS TotalSpent
   FROM SalesLT.Customer c
   JOIN SalesLT.SalesOrderHeader soh ON c.CustomerID = soh.CustomerID
   GROUP BY c.CompanyName
   ORDER BY TotalSpent DESC;
   ```
6. Now create your own table and put data in it — this is the part that proves it's a real database, not a read-only demo: **✅**
   ```sql
   CREATE TABLE Students (
       StudentID   INT IDENTITY(1,1) PRIMARY KEY,
       FullName    NVARCHAR(100) NOT NULL,
       Email       NVARCHAR(100),
       PhoneNumber NVARCHAR(20),
       EnrolledOn  DATE DEFAULT GETDATE()
   );

   INSERT INTO Students (FullName, Email, PhoneNumber) VALUES
       ('Anita Sharma',  'anita@example.com',  '+91-9876543210'),
       ('Ravi Kumar',    'ravi@example.com',   '+91-9876543211'),
       ('Sofia Almeida', 'sofia@example.com',  '+91-9876543212');

   SELECT * FROM Students;
   ```
   Keep this table — we use it for the masking demo in Part 7 and the restore demo in Part 6.
7. Update and delete, to complete the picture: **✅**
   ```sql
   UPDATE Students SET Email = 'anita.sharma@example.com' WHERE FullName = 'Anita Sharma';
   DELETE FROM Students WHERE FullName = 'Ravi Kumar';
   SELECT * FROM Students;
   ```
8. Go to **Settings → Connection strings** and look at the **ADO.NET** tab. Note the `Password={your_password}` placeholder — that's the value that must live in Key Vault, never in code. **✅**

---

## Part 6 — Backups and Restore

### What Azure Does Automatically

You did not configure backups. They are already running. This is one of the strongest arguments for the managed service, and the schedule is worth knowing precisely:

- **Full backup** — weekly
- **Differential backup** — every 12–24 hours
- **Transaction log backup** — **every 5–10 minutes**

That transaction log cadence is what makes **point-in-time restore** possible: Azure can rebuild your database as it existed at any moment inside the retention window, down to roughly the minute.

### Point-in-Time Restore (PITR)

- **Default retention:** 7 days
- **Configurable:** 1–35 days
- **Restores to a new database** — it never overwrites your existing one

That last point is important and often surprises people. Restoring doesn't roll your database back; it creates a *second* database alongside it. If you want to replace the original, you restore to a new name, verify the data, then rename or repoint your application.

### Long-Term Retention (LTR)

35 days doesn't satisfy "keep 7 years of records" compliance requirements. **LTR** lets you set weekly, monthly, and yearly backup retention policies for **up to 10 years**, stored in Azure Blob Storage.

LTR backups are restore-only — you can't query them in place, and restoring one creates a new database just like PITR does.

### Geo-Restore

If you chose **geo-redundant backup storage** at creation, your backups are replicated to the Azure paired region. **Geo-restore** lets you rebuild the database in a *different region* entirely — your recovery path if an entire region is lost.

The RPO (Recovery Point Objective) for geo-restore is up to **1 hour**, because the replication is asynchronous. If you need better than that, you need active geo-replication (Part 9), not backups.

| Feature | Protects against | RPO | Extra cost |
|---|---|---|---|
| **Point-in-time restore** | Bad data, dropped table, bad deployment | ~5–10 min | Included |
| **Long-term retention** | Compliance / audit requirements | Weekly-to-yearly | Blob storage cost |
| **Geo-restore** | Full region failure | Up to 1 hour | Geo-redundant storage cost |
| **Active geo-replication** | Region failure, with low RPO | Seconds | Full price of the secondary |

### Hands-On: Break Something and Restore It ✅

The best way to trust a backup is to use one.

1. In **Query editor**, do something destructive on purpose: **✅**
   ```sql
   DROP TABLE Students;
   SELECT * FROM Students;   -- Invalid object name 'Students'
   ```
   Note the current UTC time — you'll need it.
2. Go to the database → **Overview** → **Restore** in the top toolbar. **✅**
3. Configure the restore: **✅**
   - **Source:** Point-in-time
   - **Restore point:** a time a few minutes **before** you dropped the table
   - **Database name:** `db-lwm-demo-restored`
   - Note that it wants a compute size — the restored database is a **real, separately-billed database**
4. **Review + create → Create**. This takes several minutes for a small database. **✅**
5. When it's done, open `db-lwm-demo-restored` → **Query editor** and run: **✅**
   ```sql
   SELECT * FROM Students;
   ```
   Your table is back, with the exact rows it had at that moment. Nothing was configured to make this possible — it was on from the second the database existed.
6. Look at retention settings: go to the **logical server → Data management → Backups**. **✅**
   - The **Retention policies** tab is where you'd change PITR from 7 days and configure LTR weekly/monthly/yearly rules
   - The **Available backups** tab lists what exists right now
7. **Delete `db-lwm-demo-restored` now** — it's outside your free allowance and bills separately. Right-click → **Delete**. **✅**

!!! note "Why the restored database costs money"
    Your free offer covers **one** database. The restore created a second one, on a normal paid tier. Deleting it immediately keeps today free. This is the single most likely way to accidentally incur a charge on this day.

---

## Part 7 — Security

Azure SQL has more security features than any other service we've covered. Here are the ones that matter, in the order you'd actually apply them.

### Encryption — Already Done

**Transparent Data Encryption (TDE)** encrypts your database files, backups, and transaction logs at rest. It is **enabled by default** on every new Azure SQL Database, using a Microsoft-managed key. You don't have to do anything.

If your compliance requirements demand you control the key, you can switch to **customer-managed keys** stored in **Azure Key Vault** (Day 19) — a pattern called Bring Your Own Key. The database then can't be read without your key, which also means losing that key means losing the database. Handle accordingly.

**In transit**, connections are TLS-encrypted and the server enforces a minimum TLS version (set 1.2 or higher — anything older is a finding in a security review).

### Authentication — Two Ways In

**SQL authentication** is the username/password you created. It works everywhere, and it has all the problems passwords always have: it's shared, it's stored in connection strings, it rarely gets rotated, and it doesn't support MFA.

**Microsoft Entra ID authentication** is the modern answer, and it's what Day 17 is about:
- Sign in with your organisational identity
- **MFA and Conditional Access apply** to database access
- Grant access to **Entra groups** — add someone to the group, they get database access; remove them from the group, access disappears with no database change at all
- **Managed identities** let an App Service or VM connect **with no password anywhere** — the strongest option available

Set the Entra admin at the **logical server** level, then create contained users inside the database:

```sql
-- Give an Entra user access to this database
CREATE USER [someone@yourdomain.com] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [someone@yourdomain.com];

-- Give an App Service managed identity access (no password involved)
CREATE USER [my-app-service-name] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [my-app-service-name];
ALTER ROLE db_datawriter ADD MEMBER [my-app-service-name];
```

### Dynamic Data Masking

This one is genuinely clever and costs nothing. **Dynamic Data Masking** hides sensitive column values from unprivileged users **at query time**. The data on disk is unchanged — the masking happens in the response.

A support agent runs `SELECT * FROM Students` and sees `xxxx@xxxx.com` and `XXXX-XXX3210`. An administrator runs the identical query and sees real values. Same query, same table, different result based on who asked.

Built-in masking functions: default (full mask), credit card, email, random number in a range, and custom (show first N and last M characters).

**Its limitation, which you must understand:** masking is not access control. A determined user with query permissions can infer masked values with clever `WHERE` clauses. It reduces casual exposure; it does not protect against an attacker. For real protection use **Always Encrypted** (encrypted client-side, invisible to the database engine) or proper **row-level security**.

### Auditing

**SQL Auditing** logs database events — who connected, what they ran, what failed — to one of three destinations:

- **Log Analytics workspace** — best choice; queryable with KQL, and integrates with everything you'll build on Day 20
- **Storage account** — cheapest for long retention
- **Event Hub** — for streaming into an external SIEM

Enable auditing at the **server** level and every database inherits it. This is usually a compliance requirement rather than an optional extra.

### Microsoft Defender for SQL 💳

A paid add-on (~$15/server/month) that provides:

- **Vulnerability assessment** — scans your configuration against best practices and gives you a prioritised remediation list
- **Advanced threat protection** — alerts on SQL injection attempts, anomalous access patterns, brute-force login attempts, and access from unusual locations
- **Data discovery & classification** — automatically finds and labels columns holding sensitive data (email, phone, national ID, financial)

For any database holding real customer data, this is worth the money.

### Hands-On: Mask Sensitive Data and Turn On Auditing ✅

Let's make the masking demo concrete, because seeing it is much more convincing than reading it.

1. First, recreate the `Students` table (you dropped it in Part 6) in `db-lwm-demo`: **✅**
   ```sql
   CREATE TABLE Students (
       StudentID   INT IDENTITY(1,1) PRIMARY KEY,
       FullName    NVARCHAR(100) NOT NULL,
       Email       NVARCHAR(100),
       PhoneNumber NVARCHAR(20)
   );
   INSERT INTO Students (FullName, Email, PhoneNumber) VALUES
       ('Anita Sharma',  'anita@example.com',  '+91-9876543210'),
       ('Sofia Almeida', 'sofia@example.com',  '+91-9876543212');
   ```
2. Go to the database → **Security → Dynamic Data Masking**. **✅**
3. Azure will likely already show **recommended masks** — it detected the `Email` column by name. Click **Add mask** on it. **✅**
4. Click **+ Add mask** to add one manually: **✅**
   - **Schema:** dbo, **Table:** Students, **Column:** PhoneNumber
   - **Masking field format:** *Custom string*, with **prefix 4**, **padding** `XXXXX`, **suffix 3**
   - **Add**
5. **Save** the masking policy. **✅**
6. Now prove it works. As admin you're exempt from masking, so create a limited user in **Query editor**: **✅**
   ```sql
   CREATE USER supportagent WITH PASSWORD = 'A-Strong-Password-Here-123!';
   ALTER ROLE db_datareader ADD MEMBER supportagent;
   ```
7. Sign out of Query Editor, sign back in as `supportagent`, and run: **✅**
   ```sql
   SELECT * FROM Students;
   ```
   Emails come back as `xxxx@xxxx.com` and phone numbers as `+91-XXXXX210`. Sign back in as `sqladmin` and run the same query — full values. **Same query, different answer, based on identity.** **✅**
8. Turn on auditing: go to the **logical server → Security → Auditing** → toggle **Enable Azure SQL Auditing** → destination **Log Analytics** (create a workspace if you don't have one) → **Save**. This connects directly to what we build on Day 20. **✅**
9. Look at **Security → Microsoft Defender for Cloud** on the database and read the recommendations panel — the findings are visible even before you enable the paid tier. **💳** for full enablement.

---

## Part 8 — Performance and Scaling

### Scaling Is a Slider

Changing your database's size is a portal setting, not a migration:

- **Scale up/down** — more or fewer vCores/DTUs. Takes seconds to minutes, with a brief connection drop at the end when the switchover happens.
- **Change tier** — General Purpose to Business Critical, or DTU to vCore. Same mechanism, takes longer.
- **Serverless autoscaling** — happens continuously, with no intervention and no connection drop.

Because scaling is this easy, **start small**. Don't over-provision "just in case." Deploy on the smallest tier that works, watch the metrics, and scale when the data tells you to.

### Query Performance Insight

Built into the portal, no setup required. It shows you:

- Top queries by CPU, duration, and execution count
- How each query's consumption has trended over time
- Which specific query is responsible for your CPU spike

This is the first place to look when someone says "the app is slow." Nine times out of ten it's one badly-written query, and this blade names it.

### Automatic Tuning

Azure will analyse your workload and **improve it for you**:

- **CREATE INDEX** — identifies missing indexes and creates them
- **DROP INDEX** — finds unused and duplicate indexes and removes them
- **FORCE LAST GOOD PLAN** — detects when a query's execution plan regresses and reverts it automatically

Each option can be set to **on** (Azure applies it automatically), **off**, or **inherit from server**. Azure validates every change by measuring performance before and after — and **reverts anything that made things worse**.

`FORCE LAST GOOD PLAN` is the one to enable first. Plan regression is a genuinely nasty failure mode — the query didn't change, the data didn't change much, but the optimiser picked a worse plan and performance fell off a cliff. Having Azure catch and revert that automatically is enormously valuable.

### Hands-On: Scale, Inspect, and Enable Automatic Tuning ✅

1. On `db-lwm-demo` → **Settings → Compute + storage**. **✅**
   - Adjust **Max vCores** and watch the estimated cost update live
   - Switch the tier selector to *Provisioned* and back to *Serverless* to compare the pricing models
   - **Discard** — don't save changes that would take you off the free offer
2. Generate some load so the tools have something to show, then look at metrics: **✅**
   ```sql
   SELECT COUNT(*) FROM SalesLT.SalesOrderDetail sod
   CROSS JOIN SalesLT.Product p;
   ```
3. Go to **Intelligent Performance → Query Performance Insight**. Look at the *Long running queries* tab. (Data can take a few minutes to appear — worth mentioning on camera so nobody thinks it's broken.) **✅**
4. Go to **Intelligent Performance → Automatic tuning**. **✅**
   - Set **FORCE LAST GOOD PLAN** to **ON**
   - Set **CREATE INDEX** and **DROP INDEX** to **ON**
   - **Apply**
5. Go to **Monitoring → Metrics** and chart **CPU percentage** and **Data space used**. For a Serverless database, also chart **App CPU billed** — this is literally what you're being charged for, and watching it drop to zero when the database pauses is a satisfying thing to see. **✅**

---

## Part 9 — Business Continuity 💳

Backups protect your data. They don't keep your application *running* — restoring takes minutes to hours. For high availability you need a live copy.

### Built-In High Availability (Free, Automatic)

Every Azure SQL Database already has HA inside its region:

- **General Purpose:** compute and storage are separated. If the compute node fails, Azure spins up a new one and reattaches the storage. Failover takes a minute or two.
- **Business Critical:** maintains **three synchronous replicas** on local SSD. Failover is nearly instant, because a replica is already running and current.

You configure nothing. It's included in the price of the tier.

### Zone Redundancy

Enable **zone redundancy** and your replicas are spread across separate Availability Zones — physically separate datacentres within the region, each with independent power, cooling, and networking. An entire datacentre can fail and your database stays online.

This is the Availability Zones concept from Day 5, applied to a PaaS service. Available on Premium, Business Critical, General Purpose, and Hyperscale.

### Active Geo-Replication

Creates up to **four readable secondary databases** in other regions, kept current by asynchronous replication.

- Secondaries are **readable** — point reporting workloads at the nearest one
- Failover is **manual** — you decide when to promote
- Each secondary is billed at full price
- RPO is typically **under 5 seconds**

### Auto-Failover Groups

The production-grade version. A failover group wraps geo-replication with:

- **A listener endpoint** — `<group-name>.database.windows.net`, which always points at whichever database is currently primary. Your application connection string never changes during a failover.
- **Automatic failover** — with a configurable grace period
- **Group-level failover** — multiple databases fail over together, so an app with several databases doesn't end up split across regions

**The exam distinction, stated plainly:** geo-replication is manual and per-database. Failover groups add automatic failover and a stable endpoint. If the question mentions "the application connection string must not change" or "automatic failover," the answer is **failover group**.

### Hands-On: Tour Business Continuity 💳

We're touring, not building — a geo-replica doubles your bill.

1. On `db-lwm-demo` → **Data management → Replicas**. Click **+ Create replica** and walk through the blade: choose a target region, note it wants a server in that region, note the pricing. **Cancel without creating.** **💳**
2. On the **logical server → Data management → Failover groups**. Click **+ Add group** and read the configuration: the group name becomes the listener endpoint, and note the **read/write failover policy** and **grace period** settings. **Cancel.** **💳**
3. Back on the database → **Compute + storage**, look for the **zone redundancy** checkbox — note which tiers offer it and which don't. **✅**

---

## Part 10 — The Rest of the Family: PostgreSQL, MySQL, and Friends

Azure SQL is built on SQL Server. If your application speaks a different database language, Azure has fully managed services for those too — with the same operational model: Microsoft runs the engine, you run the data.

### Azure Database for PostgreSQL — Flexible Server

The managed PostgreSQL service. **Flexible Server** is the current and only deployment model for new work (the older Single Server model is retired).

What you get:
- Choice of major PostgreSQL versions
- **Burstable, General Purpose, and Memory Optimized** compute tiers — burstable (B1ms) is the cheap entry point
- **Stop/start** — genuinely stop the server and pay only for storage, ideal for dev
- **High availability** with a standby in the same or a different Availability Zone
- **Read replicas** for scaling reads
- **Extensions** — PostGIS for geospatial, pgvector for AI embeddings, TimescaleDB for time-series
- VNet integration or public access with firewall rules
- Automatic backups with PITR, 7–35 days

**Choose PostgreSQL for:** open-source applications, geospatial workloads, anything using pgvector for AI/embedding search, and teams that already know Postgres.

### Azure Database for MySQL — Flexible Server

The managed MySQL service, structured almost identically to the PostgreSQL one.

- MySQL 8.0+
- Burstable/General Purpose/Memory Optimized, with stop/start
- **Zone-redundant HA** with automatic failover
- Read replicas
- Automatic backups with PITR

**Choose MySQL for:** WordPress, Drupal, Magento, Laravel, and traditional LAMP-stack applications — anything where the ecosystem assumes MySQL.

### Choosing Between Them

The honest answer: **choose based on what your application already speaks.** These are all mature, capable, fully managed relational databases. Migrating an application between engines is expensive and rarely worth it.

| If your situation is… | Choose |
|---|---|
| .NET application, or existing SQL Server skills on the team | **Azure SQL Database** |
| Existing on-premises SQL Server to migrate as-is | **SQL Managed Instance** |
| Open-source stack, geospatial data, or AI vector search | **PostgreSQL Flexible Server** |
| WordPress, Drupal, Magento, LAMP stack | **MySQL Flexible Server** |
| Globally distributed, low-latency, schema-flexible NoSQL | **Cosmos DB** (Bonus Day 33) |
| Sub-millisecond caching in front of any of the above | **Azure Cache for Redis** (not covered in this course) |

### A Word on Relational vs NoSQL

Everything today has been **relational**: fixed schema, tables with rows and columns, relationships enforced by foreign keys, queried with SQL, and **ACID** transactional guarantees. That's the right default for the overwhelming majority of applications — orders, users, inventory, payments, bookings.

**NoSQL** (Cosmos DB) trades some of those guarantees for flexible schemas and effortless global distribution. It's the right answer for specific problems — globally distributed apps needing single-digit-millisecond reads worldwide, or genuinely schema-less data.

**Start relational.** Move to NoSQL when you have a named problem that relational can't solve. "It might scale better" is not a named problem.

### Hands-On: Create a PostgreSQL Flexible Server ✅/💳

Free-account holders inside their first 12 months can do this free (B1ms burstable, 32 GB). Otherwise it's a few dollars a month — or just follow along and watch.

1. Search **Azure Database for PostgreSQL flexible servers** → **+ Create**. **✅/💳**
2. Configure: **✅/💳**
   - **Resource group:** `rg-day16-demo`
   - **Server name:** `pg-lwm-<yourname>`
   - **Region:** same as your SQL database
   - **PostgreSQL version:** latest available
   - **Workload type:** *Development* — this selects burstable defaults
   - **Compute + storage → Configure server:** *Burstable, B1ms, 32 GiB*. Note the **stop/start** capability mentioned here.
   - **Availability zone:** No preference
   - **High availability:** **unchecked** (it doubles the cost)
3. **Authentication:** PostgreSQL authentication only, set an admin username and password. **✅/💳**
4. **Networking:** *Public access*, tick **Allow public access from any Azure service** and **+ Add current client IP address**. **✅/💳**
5. **Review + create → Create.** Provisioning takes 5–10 minutes. **✅/💳**
6. When it's ready, note the parallels to everything you just learned: a **Networking** blade with firewall rules, a **Backup and restore** blade with PITR, **Server parameters** (the PostgreSQL equivalent of engine configuration), and **Replication** for read replicas. **The concepts transfer completely — only the engine changed.** **✅/💳**
7. Open **Databases → postgres** and try the built-in query interface if available in your region, or connect with `psql` from Cloud Shell:
   ```bash
   psql "host=pg-lwm-<yourname>.postgres.database.azure.com port=5432 dbname=postgres user=<admin> sslmode=require"
   ```
8. **Stop the server** when you're finished exploring — **Overview → Stop**. This is the PostgreSQL equivalent of auto-pause, and it's the difference between a free month and a surprise. **✅/💳**

---

## Part 11 — Cleanup ✅

Do this at the end of the session, on camera, every time.

1. Confirm `db-lwm-demo-restored` is already deleted (Part 6). If not, delete it now — it's the one thing here that definitely bills. **✅**
2. If you created the PostgreSQL server and don't want it, delete it — or at minimum **Stop** it. **✅**
3. Delete the whole resource group: search **Resource groups** → `rg-day16-demo` → **Delete resource group** → type the name → **Delete**. **✅**
4. **Decide about `db-lwm-demo` before you delete it.** Deleting the resource group removes it too. It's on the free offer and costs nothing while paused, so there's a good argument for keeping it — the Day 17 (Entra ID) and Day 19 (Key Vault) demos both benefit from having a live database to point at, and the capstone on Day 31 needs one. **My recommendation: keep `db-lwm-demo` and its logical server, and delete everything else.** **✅**
5. Whatever you keep, check **Cost Management → Cost analysis** tomorrow and confirm it reads what you expect. Make this a reflex.

---

## Summary

Today you stopped building infrastructure and started building the part of an application that actually persists.

**Azure SQL Database** is SQL Server with the operational burden removed. Microsoft patches it, backs it up, replicates it, and fails it over. You get a connection string, a schema, and your queries. That trade is why PaaS databases have become the default for new applications.

The **four deployment options** solve four different problems: Single Database for new apps, Elastic Pool for many databases with uncorrelated load, Managed Instance for lifting an existing SQL Server into Azure intact, and SQL Server on a VM for when you genuinely need the OS.

The **vCore purchasing model** with the **Serverless** compute tier is the most beginner-friendly billing arrangement in Azure — the database pauses when nobody's using it, and a paused database costs nothing but storage. Combined with the free offer, you can keep a real database running indefinitely and pay essentially zero.

The **logical server** is not a machine. It's a free container providing a DNS endpoint, a firewall, and administrative identity. Almost every connection problem you'll ever have with Azure SQL is solved at that level.

**Backups you never configured** have been running since the moment you created the database, and you restored from one today. **Encryption at rest** was already on. **Dynamic Data Masking** changed what a query returned based on who ran it. **Automatic tuning** will write your indexes. This is what "managed" actually means in practice — not fewer features, but fewer things you have to remember to switch on.

And the whole family works the same way: **PostgreSQL** and **MySQL Flexible Server** give you the identical operational model with a different engine. Learn the pattern once and it transfers.

### What's Next

You've now got compute, networking, storage, and data. The obvious question is the one we've been deferring all course: **who is allowed to touch any of it?**

Tomorrow, Day 17 begins Phase 5 with **Microsoft Entra ID** — Azure's identity platform. Users, groups, multi-factor authentication, Conditional Access, and managed identities. That last one is the direct sequel to today: we'll connect an App Service to the database you just built **with no password in the connection string at all**, using an identity Azure manages for you. Every credential you wrote down today becomes something you no longer need.

---

## Key Takeaways

- **Azure SQL Database is PaaS** — Microsoft owns the OS, engine, patching, backups, and HA; you own the schema, queries, access, and data. That line is the source of most exam questions.
- **Four deployment options:** Single Database (new apps), Elastic Pool (many databases, variable load), Managed Instance (lift-and-shift with instance-level features), SQL on a VM (needs OS access).
- **vCore beats DTU** for anything new — it's the only path to Serverless, Hyperscale, Hybrid Benefit, and reserved capacity.
- **Serverless auto-pauses** after a configurable idle period (1 hour minimum) and costs nothing for compute while paused — at the price of a ~1 minute resume delay on the next connection.
- **The free offer is free forever**, not a 12-month trial: ~100,000 vCore-seconds plus 32 GB storage per month, on every subscription type.
- **The logical server is not a machine.** It's free, it has no OS, and it exists to provide a DNS endpoint, a firewall, and admin identity.
- **A new server blocks everyone by default.** Add your client IP explicitly. "Allow Azure services" opens access to Azure resources in *any* subscription — use Private Endpoint for production instead.
- **Backups are automatic and always on:** full weekly, differential every 12–24h, transaction log every 5–10 minutes. PITR retention 1–35 days; LTR up to 10 years.
- **Restore always creates a new database** — it never overwrites the original. Delete restored copies promptly; they bill separately.
- **TDE is on by default.** Minimum TLS 1.2. Prefer **Entra ID authentication** over SQL logins, and **managed identities** over both.
- **Dynamic Data Masking is not access control** — it reduces casual exposure at query time. Use Always Encrypted or row-level security when you need real protection.
- **Business Critical includes a free readable replica** — the cheapest way to move reporting load off your primary.
- **Hyperscale restores in minutes regardless of size** — up to 100 TB. That property alone drives the choice for very large databases.
- **Failover group vs geo-replication:** the failover group adds automatic failover and a stable listener endpoint, so the application connection string never changes.
- **Choose the engine your application already speaks.** PostgreSQL for open-source/geospatial/vector, MySQL for LAMP and WordPress, Azure SQL for .NET and SQL Server heritage. Start relational; move to NoSQL only for a named problem.
