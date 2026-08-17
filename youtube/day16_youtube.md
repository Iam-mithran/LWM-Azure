# Day 16 — YouTube Metadata

---

## Video Title

Azure SQL Database Explained: Free Forever Tier, Serverless, Backups & Security (Hands-On) | Day 16 | LearnWithMithran

---

## Thumbnail

**Main text (large, bold):** `Free Forever Database`
**Sub text:** `Day 16 — Azure SQL Database`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- A database cylinder icon with a "$0" badge and a small pause (⏸) symbol on it — representing Serverless auto-pause
- A crossed-out server rack icon beside it, labelled "No OS. No patching. No licence."
- A small clock-rewind icon in a corner labelled "Point-in-Time Restore"
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** A real, production-grade SQL database on Azure that costs nothing — it pauses itself when you're not using it, backs itself up, and encrypts itself, all without you configuring a thing.

---

## Description

*Welcome back to Learn With Mithran! Every single thing you've built in this course so far has been missing something — it doesn't remember anything. The VM from Day 3 served a page. The App Service from Day 6 hosted a site. The load balancer from Day 12 spread traffic. Restart any of them and whatever they were holding is gone. Today we build the layer that actually persists. And here's the part that surprises people from a traditional IT background: you will not install SQL Server, patch it, back it up, configure high availability, or renew a licence. Azure does all of that. You get a connection string and a database.*

In this hands-on session, you'll deploy a real Azure SQL Database on the **free offer** — and this is not a 12-month trial, it's a genuinely free General Purpose Serverless database available on every subscription type, with roughly 100,000 vCore-seconds of compute and 32 GB of storage every month, forever. You'll load the AdventureWorksLT sample data, query it straight from your browser with the portal Query Editor, and create your own tables with real SQL. Then we go deep on the parts that make it production-ready: the **logical server** (the most misunderstood object in Azure SQL — it is *not* a machine, and it costs nothing), the three layers of firewall rules including the "Allow Azure services" toggle that's far wider than most people realise, and Private Endpoints for locking a database off the public internet entirely. You'll deliberately break your own connection to see the single most common Azure SQL error in the world, then deliberately DROP a table and **restore it from a backup you never configured** — because point-in-time restore has been running since the second the database existed. We'll prove Dynamic Data Masking live by creating a limited user and watching the exact same query return masked emails and phone numbers, turn on auditing, enable automatic tuning that writes your indexes for you, and finish with PostgreSQL and MySQL Flexible Server so you know exactly when to reach for a different engine. This is one of the cheapest days in the entire course — almost every step is genuinely free. 🚀

📂 *Get the course notes and diagrams from GitHub:*
- https://github.com/Iam-mithran/LWM-Azure

♾️ *Join the Discord:*
- https://discord.gg/N7GBNHBdqw

📢 *Follow Us on Social Media:*
- https://www.instagram.com/learnwithmithran/

☎️ *Contact Information:*
Phone Mithran: +91 91500 87745
Greens Technologys, Perumbakkam (https://maps.app.goo.gl/u34U3rXu8zPFfQh5A)

🧩 *Put the pieces together with this reference – watch here!*

☁️ Master core AWS services step-by-step – watch the full AWS playlist here (https://youtube.com/playlist?list=PLPLf8iqkntdMxtXT04-TG1WzDvBPUJ3qk&si=CFx_IMjpWcufkTme)
🛠️ Get hands-on with top DevOps tools and workflows – dive into the DevOps playlist here (https://youtube.com/playlist?list=PLPLf8iqkntdNaU9GbaZckoQalKPRJMvT6&si=eUAHHibEmDI4bQuP)
🧠 Level up your coding with practical Python lessons – start learning here (https://youtube.com/playlist?list=PLPLf8iqkntdNefseVlDOaRQ7zersK79AI&si=6UUBU90Q6Ov96g96)
🐧 Build your Linux fundamentals from scratch – explore the Linux series here (https://youtube.com/playlist?list=PLPLf8iqkntdMew0yP5Ad9pbaZki0Wf-2w&si=4uJ2EAYamtO6PZgz)

🎯 *Topics Covered*:

🔹 Why managed databases exist — the exact list of jobs Microsoft takes over, and the smaller list you keep
🔹 PaaS vs IaaS in database terms — Azure SQL Database vs SQL Server on a VM, and the rule for choosing
🔹 The four Azure SQL products sharing one name — Single Database, Elastic Pool, Managed Instance, SQL on a VM
🔹 Elastic Pools explained — why they exploit *variation* in usage, not volume, and the trap that makes them save you nothing
🔹 Managed Instance explained — SQL Server Agent, cross-database queries, CLR, and why it's for lift-and-shift only
🔹 DTU vs vCore purchasing models — and why the industry moved decisively to vCore
🔹 Service tiers by what actually differs underneath — General Purpose, Business Critical (free readable replica), Hyperscale (100 TB, restores in minutes)
🔹 Serverless compute explained — auto-pause, auto-resume, per-vCore-second billing, and the ~1 minute resume trade-off
🔹 The Azure SQL Database free offer — free forever, not a 12-month trial, on every subscription type
🔹 Demo — create a SQL Database on the free offer with AdventureWorksLT sample data loaded
🔹 The logical server explained — it is NOT a machine, it has no OS, and it costs absolutely nothing
🔹 Firewall rules — server-level vs database-level, and why "Allow Azure services" is wider than you think
🔹 Private Endpoints for Azure SQL — a private IP in your VNet, with the connection string unchanged
🔹 Demo — break your own connection on purpose and see the #1 Azure SQL error in the world
🔹 Demo — query from the browser with Query Editor: SELECT, JOIN, CREATE TABLE, INSERT, UPDATE, DELETE
🔹 Automatic backups you never configured — full weekly, differential every 12–24h, transaction log every 5–10 minutes
🔹 Point-in-time restore, long-term retention up to 10 years, and geo-restore compared by what each protects against
🔹 Demo — DROP a table on purpose and restore the database to the moment before you did it
🔹 Transparent Data Encryption, minimum TLS, and customer-managed keys in Key Vault
🔹 Microsoft Entra ID authentication and managed identities — connecting with no password anywhere
🔹 Demo — Dynamic Data Masking proved live with a limited user: same query, same table, different answer
🔹 Why masking is NOT access control — and what to use instead when you need real protection
🔹 SQL Auditing to Log Analytics, and Microsoft Defender for SQL — vulnerability assessment and threat protection
🔹 Scaling as a slider, Query Performance Insight, and automatic tuning that creates your indexes for you
🔹 Business continuity — built-in HA, zone redundancy, active geo-replication, and auto-failover groups
🔹 The exam distinction between geo-replication and failover groups, stated plainly
🔹 Azure Database for PostgreSQL and MySQL Flexible Server — when each one wins
🔹 Relational vs NoSQL — why you should start relational and what a real reason to move looks like
🔹 AZ-104 and AZ-305 exam framing for database decisions, with the keywords that give away the answer
🔹 Cleanup — and the one restored database that will quietly bill you if you forget it

📌 *Who Is This Video For:*

💻 Beginners who've built compute, storage, and networking and are ready for the data layer
🧑‍🎓 Students preparing for AZ-104 (Administrator) or AZ-305 (Solutions Architect) certification
☁️ Anyone who wants a real SQL database to practise on that genuinely costs nothing to keep running
🔥 Learners who want to *see* a dropped table come back from a backup, not just be told backups exist
🖥️ Developers and DBAs moving from self-managed SQL Server to a fully managed service
🚀 Anyone designing production architectures who needs to pick between Single Database, Elastic Pool, Managed Instance, PostgreSQL, and MySQL

🔍 *Chapters:*
0:00 Intro — Everything You've Built So Far Forgets Everything
3:00 Part 1 — Why Managed Databases Exist
8:00 What Microsoft Runs vs What You Run — PaaS vs IaaS
12:00 Portal Tour — The Whole Azure SQL Family in One Screen
15:00 Part 2 — Single Database vs Elastic Pool
20:00 Managed Instance and SQL Server on a VM
25:00 The Decision Table — How AZ-104 and AZ-305 Phrase It
28:00 Part 3 — DTU vs vCore Purchasing Models
33:00 Service Tiers — General Purpose, Business Critical, Hyperscale
38:00 Serverless — The Database That Pauses Itself
42:00 Demo — Create a SQL Database on the Free Forever Offer
52:00 Part 4 — The Logical Server Is Not a Machine
56:00 Firewall Rules and the "Allow Azure Services" Trap
61:00 Private Endpoints for Azure SQL
64:00 Demo — Networking Blade, and Breaking Your Own Connection on Purpose
69:00 Part 5 — Demo: Query Editor, Sample Data, and Real SQL
77:00 Part 6 — Backups You Never Configured: PITR, LTR, Geo-Restore
82:00 Demo — Drop a Table and Restore It From a Backup
88:00 Part 7 — TDE, Entra ID Authentication, and Managed Identities
92:00 Demo — Dynamic Data Masking Proved With a Limited User
98:00 Auditing and Microsoft Defender for SQL
101:00 Part 8 — Scaling, Query Performance Insight, Automatic Tuning
106:00 Part 9 — HA, Zone Redundancy, Geo-Replication, Failover Groups
111:00 Part 10 — PostgreSQL and MySQL Flexible Server
116:00 Summary, Exam Tips, and Cleanup

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #AzureSQL #AzureSQLDatabase #SQLServer #ServerlessSQL #AzureDatabase #ManagedInstance #ElasticPool #PostgreSQL #MySQL #AzureForBeginners #MicrosoftAzure #AZ104 #AZ305 #LearnAzure #AzurePortal #AzureTutorial #LearnWithMithran #CloudComputing #AzureBeginner #GreensTechnologies #AzureTraining #AzureCertification #PaaS #DynamicDataMasking

---

## Tags

azure sql database tutorial, azure sql serverless, azure sql free tier, azure sql database free offer, azure sql logical server, azure sql firewall rules, point in time restore azure sql, dynamic data masking azure, azure sql elastic pool, azure sql managed instance, dtu vs vcore, hyperscale azure sql, business critical azure sql, azure sql query editor, azure database for postgresql flexible server, azure database for mysql flexible server, azure sql auditing, azure sql automatic tuning, failover group azure sql, az-104, az-305, azure for beginners, azure portal, azure tutorial, learnwithmithran, greens technologies, azure training, azure certification, microsoft azure, azure sql demo
