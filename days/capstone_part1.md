# Day 20 — Capstone: Design and Build a Complete Three-Tier Application

> Phase 9 — Capstone Project (moved up: this day now runs immediately after Day 19)
>
> **Scope note — reworked 21 September 2026.** This file previously described a Day 30 capstone built
> on **AKS, ACR, Docker images and Terraform**. Every one of those is taught *after* the point where
> this day now sits, so the old plan asked students to build with services they had never seen. The
> capstone is now a **three-tier application on App Service + VMs + VNet + Azure SQL**, using **only
> what Days 1–19 taught**, built entirely in the portal.
>
> **No containers, no Kubernetes, no Functions, no microservices, no IaC, no pipelines.**

## This Is One Video, and It Has No Length Cap

Every other day in this course is capped at two hours. **This one is not.** It is a project video:
the student builds one architecture from a blank resource group to a working, monitored, secured
application, and **stopping halfway through would leave them with an app that doesn't run.**

- **Expect 3 to 3.5 hours.** Roughly **22 parts, 18,000–20,000 words.**
- **Do not split it.** The value of a capstone is that it is continuous — the moment it becomes two
  videos, half the audience watches one of them.
- **Chapters carry the weight** that a shorter runtime normally would. Every part gets a timestamp,
  and the four phase boundaries get obvious "you can stop here for a break" markers.
- **The full cleanup happens at the end of this video.** Unlike the two-part plan this replaces,
  nothing has to survive until next time.
- **Built entirely from scratch.** Nothing from earlier days is reused — the SQL server and database
  are created fresh inside `rg-lwm-capstone`, so one resource-group delete removes the whole system.

## Numbering — Unresolved, Deliberately

This day is **Day 20**. The rest of the course is **not yet renumbered** and this file does not assume
an answer:

- `course_outline.md` and `recording_plan.md` still show Day 20 as *Azure DevOps Introduction*, and
  the capstone at Days 30–31. Both need updating once the numbering is settled.
- **`capstone_part2.md` has not been reworked.** It still describes CI/CD, Bicep/Terraform and AKS
  against the old numbering. Now that the whole three-tier build fits in this one video, the natural
  job for that file is **the end-of-course video that automates this same app** — a pipeline that
  deploys it and a Bicep/Terraform template that recreates it — once DevOps and IaC have been taught.
  That is a recommendation, not a decision.
- **Do not renumber anything downstream until that call is made.**

## Scope Rules for This Day

Same teaching standard as Days 17–19 — student-friendly depth, not a reference manual — with the
length cap lifted:

- **22 parts**, four phases, one continuous build.
- **Near-free lab.** The whole build runs for **well under a dollar** for a session. Exact numbers in
  the cost table below. Nothing here needs a paid subscription.
- **Portal only.** No Bicep, no Terraform, no pipelines — those days have not happened yet. Cloud
  Shell appears twice, for `cloud-init` and a `sqlcmd` connectivity test.
- **Everything traces back to a day.** Every service used has already been taught. The capstone's
  job is **assembly and judgement**, not new features — see the traceability table in Part 1.
- **Design before clicking.** Parts 1–4 are architecture and planning with no portal at all. That is
  deliberate: this is the first day where the answer is a decision, not a blade.
- **One genuinely new thing is allowed:** **App Service regional VNet integration** (Part 13). It is
  the glue between the PaaS tier and the VNet, Day 6 never reached it, and there is no way to build
  this architecture without it. It gets taught properly rather than clicked past.
- **Cut deliberately:** Application Gateway, Front Door, WAF, VPN/ExpressRoute, VMSS autoscale,
  Azure Backup and multi-region design. All are named as production upgrades **with prices** in
  Part 20, none are built — every one is either expensive or a distraction from the three-tier shape.

### Why App Service for the web tier and VMs for the app tier

Both were asked for, and the pairing is the strongest of the three layouts considered:

- It puts **PaaS and IaaS side by side in one architecture**, so the trade-off stops being abstract.
  The web tier scales with a slider and has no OS; the app tier has an OS, NSGs, a load balancer and
  a patching story. Students feel the difference rather than being told it.
- It is the **most common real-world shape** for teams migrating gradually — new front end on App
  Service, existing application servers still on VMs.
- It forces **VNet integration**, the single most useful App Service networking concept.

## Portal Currency — Verified September 2026

Researched against current Microsoft documentation before writing. Five findings shape the script,
and **two of them are corrections to our own earlier days**:

| Finding | Detail | Where it lands |
|---|---|---|
| **VNet integration needs Basic, not Premium** ⚠️ | Regional VNet integration requires a **Basic, Standard, Premium, Premium v2/v3/v4 or Elastic Premium** plan. **B1 works.** Free F1 does not. There is **no extra charge** for the feature beyond the plan. | Part 13. **Day 6's plan table lists VNet integration as a Premium feature and Day 18 Part 11 says "Standard or higher" — both are wrong and should be corrected in those scripts.** |
| **Bastion Developer SKU is free** ⚠️ | Free, needs **no `AzureBastionSubnet`**, dev/test only, **one VM connection at a time**, and **does not work across VNet peering**. | Part 10. **Day 11 teaches only Basic (~$0.19/hr) and Standard — it should gain a Developer row**, because it removes the one genuinely painful cost from that day. |
| **Integration subnet sizing** | The subnet must be **delegated to `Microsoft.Web/serverFarms`**, hard minimum **/28**, **/26 recommended** by Microsoft (and /27 is the portal minimum when the subnet is created during integration), one IP per plan instance, and **at most two integrations per App Service plan**. | Parts 4 and 13 |
| **Azure SQL free offer still current** | ~**100,000 vCore-seconds/month**, **32 GB** data, **32 GB** backup, on every subscription type, not time-limited. Auto-pause keeps the worst case at "paused", not "billed". | Part 7 — we create `db-lwm-expenses` fresh on the free offer |
| **Basic Load Balancer is gone** | Standard only. Internal and public Standard LBs bill the same way: ~**$0.025/hr** plus ~**$0.005/hr per rule**. | Part 11 |

## Goal

Take a written requirement, turn it into an architecture, and build the whole thing in one sitting —
a **three-tier application where each tier is isolated, the database is unreachable from the
internet, no credential is stored anywhere, every tier reports to one monitoring workspace, and every
decision can be justified out loud.**

The finishing line is concrete: **a browser hits a public URL, that page's data came from a database
that rejects every connection from the internet, and nothing in the configuration contains a
password.**

---

## The Architecture

```text
                          Internet
                              │
                              ▼
        ┌─────────────────────────────────────────┐
        │  WEB TIER                               │   App Service B1
        │  app-lwm-capstone-<name>                │   managed identity
        │  Key Vault reference in app settings    │   no credential stored
        └─────────────────┬───────────────────────┘
                          │  regional VNet integration
                          │  snet-web-integration  10.20.1.0/26
                          ▼
        ┌─────────────────────────────────────────┐
        │  APP TIER                               │   Internal Load Balancer
        │  lb-app-internal   10.20.2.100          │      ├── vm-app-1  (zone 1)
        │  snet-app  10.20.2.0/24                 │      └── vm-app-2  (zone 2)
        └─────────────────┬───────────────────────┘      no public IP on either
                          │  NSG: only snet-web-integration may reach :80
                          ▼
        ┌─────────────────────────────────────────┐
        │  DATA TIER                              │   Azure SQL — public access DISABLED
        │  snet-data  10.20.3.0/24                │   Private endpoint + privatelink zone
        │  db-lwm-expenses (free offer)           │   NSG: only snet-app may reach :1433
        └─────────────────────────────────────────┘

  Secrets:     kv-lwm-capstone-<name>  — connection string, read via managed identity
  Management:  Azure Bastion Developer SKU (free) — no public IPs on any VM
  Monitoring:  one Log Analytics workspace, every tier's diagnostics, four alert rules
  Reserved:    10.20.0.0/26 for AzureBastionSubnet, if Bastion is ever upgraded
```

## What Today Costs

| Component | Cost | Label |
|---|---|---|
| VNet, subnets, NSGs, ASGs, private DNS zone | **Free** (private DNS zone ~$0.50/mo) | ✅ |
| 2 × `Standard_B1s` VMs | **Free tier: 750 B1s hours/month.** Otherwise ~$0.012/hr each | ✅ |
| Azure SQL `db-lwm-expenses` | **Free offer** — created fresh in Part 7 | ✅ |
| Azure Bastion **Developer SKU** | **Free** | ✅ |
| **App Service plan B1** | ~**$0.018/hr** (~$13/mo). **Not free, and F1 cannot do VNet integration** | ✅ *cents for a session* |
| Internal Load Balancer (Standard) | ~$0.025/hr + ~$0.005/hr per rule ≈ **$0.72/day** | ✅ *cents for a session* |
| Private endpoint for SQL | ~$0.01/hr ≈ **$0.25/day** + tiny data processing | ✅ *cents for a session* |
| Key Vault (Standard) | No base fee; operations $0.03/10,000 | ✅ |
| Log Analytics + Application Insights | Inside the 5 GB per billing account per month free grant | ✅ |
| **A 4-hour build, everything running, deleted at the end** | **≈ $0.25** | |
| **Left running 24/7 for a week** | **≈ $12** | ⚠️ say this out loud, twice |
| Application Gateway + WAF instead of the ILB | ~$0.246/hr + capacity units ≈ **$180+/mo** | 💳 concept only, Part 20 |

**The honest framing for camera:** nothing here is expensive, but **three meters run continuously** —
the App Service plan, the load balancer and the private endpoint — and none of them auto-pause the
way the database does. That is exactly why Part 22 deletes everything, and why Part 5 sets a budget
alert before a single resource exists.

**If a student has to stop mid-build:** deallocate both VMs and stop the App Service; that parks most
of the cost at about $1/day. The lab is designed to be finished in one sitting, but say this anyway.

---

## Phase One — Design (Parts 1–4, no portal)

**1. The Requirement, and Where Every Service Came From**
Read a realistic one-paragraph brief aloud — an internal expenses app: a web UI for staff, an API
holding the business logic, a SQL database, "the database must not be reachable from the internet",
"two staff must administer it without sharing an account", "we need to know when it breaks". Then the
traceability table: **every service we are about to use, and the day that taught it** — VNet/NSG (10),
private endpoint + Bastion (11), internal LB (12), private DNS (13), VMs and zones (3–5), App
Service (6), SQL (16), managed identity + RBAC (17), Key Vault (18), Monitor (19).
The point to land: **a capstone is not new material, it is judgement applied to known material.**

**2. Why Three Tiers, and What Each One Is For**
Return to Day 4's one/two/three-tier diagram and make it concrete. Presentation, logic, data — and
the actual reason for the split: **each tier scales differently, fails differently, and needs a
different blast radius.** The security argument is the strong one: the only thing exposed to the
internet is the tier holding no data and no business rules.

**3. The Decisions, Stated as Trade-Offs**
The part that separates this from a click-along. Six decisions, each argued both ways in 60 seconds:
App Service vs VMs for the web tier; two VMs across **availability zones** vs an availability set vs
one VM; **internal** load balancer vs public; **private endpoint** vs service endpoint vs firewall
rules for SQL; managed identity vs a connection string in config; one resource group vs several.
Each closes with the choice *and its price*. **No demo.**

**4. Address Planning and Naming**
CIDR from Day 9 applied for real: `10.20.0.0/16`, and why you carve it up before you create anything.
The four subnets, the `/26` recommended integration subnet and its `Microsoft.Web/serverFarms` delegation, the
`/26` reserved for a future `AzureBastionSubnet`, and Azure's five reserved addresses per subnet.
Then the naming convention, written on screen and used for the rest of the day.
*No demo — this part is a whiteboard and it should look like one.*

> **Break marker 1** — "that's the whole design. Everything from here is building it."

---

## Phase Two — Network and Data Tier (Parts 5–8)

**5. Resource Group, VNet, and a Budget Before Anything Else**
*Demo: create `rg-lwm-capstone`, then `vnet-capstone` (10.20.0.0/16) with all four subnets in one
pass, including the delegated web-integration subnet. Set a **budget alert** (Day 2) before creating
anything billable, and add a **resource lock** at the end of the part.* ✅

**6. NSGs — Making It Three Tiers Instead of One Flat Network**
The rules *are* the architecture. Default rules first (Day 10), then the deliberate set: app tier
accepts :80 **only from the web integration subnet**, data subnet accepts :1433 **only from the app
subnet**, everything else denied. **ASGs** get used properly here — `asg-app-tier` as source or
destination instead of hardcoded IPs.
*Demo: build `nsg-app` and `nsg-data`, attach them, and read the effective rules.* ✅

**7. The Data Tier: SQL Behind a Private Endpoint**
Create `sql-lwm-cap-<name>` / `db-lwm-expenses` from scratch in `rg-lwm-capstone` on the free offer,
with the AdventureWorksLT sample, public endpoint and client IP allowed. Query it from the portal once
so it visibly works. Then **turn public network access off** — the moment that makes this a
real architecture. Create the private endpoint into `snet-data`, watch Azure create the
`privatelink.database.windows.net` private DNS zone and link it to the VNet, and explain why the
connection string does not change (Day 13 and Day 11 paying off together).
*Demo: create the server and database, query it, disable public access, add the private endpoint,
inspect the DNS zone's A record.* ✅

**8. Prove the Database Is Private**
Three proofs, in order, because "trust me" is not an architecture:
1. From your **laptop**, connect with SSMS or the portal query editor → **fails**.
2. From `vm-app-1` via Bastion, `nslookup sql-lwm-cap-<name>.database.windows.net` → **10.20.3.x**.
3. From the same VM, `sqlcmd` → **connects**.
*Demo: all three. This is the emotional high point of the first half — don't rush it.* ✅

---

## Phase Three — App Tier and Web Tier (Parts 9–15)

**9. The App Tier VMs**
Sizing (B1s and why), **availability zones 1 and 2** and the SLA that buys (Day 5), **no public IP on
either VM**, and `cloud-init` installing Nginx with two endpoints: `/health` for the probe and
`/api/info` returning the hostname and zone so load balancing is visible.
*Demo: deploy `vm-app-1` and `vm-app-2` with cloud-init, into `asg-app-tier`.* ✅

**10. Getting In Without a Public IP**
Bastion **Developer SKU** — free, no subnet, one session at a time, no peering. State the limits
honestly and note that production uses Basic or Standard.
*Demo: connect to `vm-app-1` in the browser, confirm `curl localhost/health` works.* ✅

**11. The Internal Load Balancer**
Every component named from Day 12 — frontend (a **private** IP, 10.20.2.100), backend pool, health
probe on `/health`, rule on :80 — and why the frontend being private is the entire difference.
*Demo: build `lb-app-internal`, add both VMs, configure the probe and the rule.* ✅

**12. Prove the App Tier Works, Then Break It**
*Demo: from `vm-app-2` via Bastion, `curl 10.20.2.100/api/info` repeatedly and watch the hostname
alternate. Then **stop Nginx on `vm-app-1`**, watch the probe mark it down within ~15 seconds and
every response come from `vm-app-2`. Restart it and watch it rejoin.* ✅
The lesson to say out loud: **the health probe is what makes a load balancer a resilience feature
rather than a traffic splitter.**

**13. The Web Tier, and VNet Integration Properly Explained**
The one new concept of the day, and it deserves its own part. **Outbound only** — integration does
not make the app privately reachable, that's a private endpoint. Needs **Basic or above** (state the
Day 6 / Day 18 corrections plainly). One IP per plan instance, `/26` recommended subnet size, subnet
delegation, two integrations per plan maximum, and **no extra charge**.
*Demo: create the B1 plan and `app-lwm-capstone-<name>`, deploy a tiny app that calls
`http://10.20.2.100/api/info` and renders the result, then enable VNet integration into
`snet-web-integration`.* ✅

**14. Watch It Fail First, Then Work**
Deliberately load the site **before** integration is active — it times out reaching a 10.x address
from the public internet. Then enable integration and reload.
*Demo: the failure, the fix, and `WEBSITE_PRIVATE_IP` in the Kudu environment page proving the app
now has an address inside `snet-web-integration`.* ✅ This is the moment the three tiers become one
application, and it should be the loudest moment in the video.

**15. Locking the Front Door**
The web tier is now the only public surface, so treat it that way: **HTTPS only**, minimum TLS 1.2,
FTP disabled, and **access restrictions** explained (Day 6) with the note that a WAF in front is the
production answer, priced in Part 20.
*Demo: flip the platform settings, read the default `*.azurewebsites.net` certificate.* ✅

> **Break marker 2** — "the application works end to end. Everything from here makes it
> *production-shaped* rather than merely working."

---

## Phase Four — Secrets and a Real Domain (Parts 16–17)

**16. Secrets: The Connection String Leaves the Config File**
Day 18's pattern in its natural habitat. **System-assigned managed identity** on the web app,
`kv-lwm-capstone-<name>`, the SQL connection string stored as a secret, **Key Vault Secrets User** at
vault scope only, and a **`@Microsoft.KeyVault(...)` reference** in an app setting.
*Demo: the green "Key Vault Reference" tick, then `Edit` the setting to show the only stored value is
the reference string itself.* ✅
Then the same idea on IaaS: **enable managed identity on both VMs** and read the secret from
`vm-app-1` over the **IMDS endpoint** (`169.254.169.254`) with curl — identical concept, different
plumbing, no credential either way.

**17. A Real Domain and a Free Certificate** *(as recorded — replaced the planned private DNS part)*
An **Azure DNS public zone** for `learnwithmithran.com`, **delegated from GoDaddy** by replacing its
name servers with the zone's four Azure name servers. A **CNAME** `www` → the app's
`azurewebsites.net` name and a **TXT** `asuid.www` record with the app's verification ID, then
**Add custom domain** with an **App Service Managed Certificate** (SNI). Optional bare domain via an
A record + `asuid` TXT, since a CNAME isn't allowed at the apex and App Service isn't an alias target.
*Demo: zone, delegation, records, validation, binding, padlock, HTTP→HTTPS redirect.* ✅ for the zone
and records; 💳 for delegation, binding and certificate (needs an owned domain).

> **End of the recording.** Everything below is written up in the published notes, not recorded.

## Homework — Monitor the Whole Workspace (formerly Parts 18–20)

**Task 1. One Workspace, Every Tier** — diagnostic settings on the App Service, vault, load balancer
and SQL database into `law-lwm-capstone`, plus Application Insights. ✅
**Task 2. Following One Request Across Three Tiers** — four KQL queries and the application map. ✅
**Task 3. Alerts That Fire While You Watch** — action group, three alert rules, fired by stopping
Nginx on both VMs, then resolved. ✅

## Add-ons — On Your Own (formerly Parts 20–22)

**Add-on A. Grade Your Own Work** — the priced gap list (WAF, single region, autoscale, backup,
patching, slots, default hostname, apex A record, vault RG, IaC), then Defender for Cloud Secure
Score on this resource group. ✅
**Add-on B. The Bill and the Cleanup** — Cost analysis, remove the lock, delete `rg-lwm-capstone`,
purge the vault, **repoint GoDaddy's name servers** so no dangling delegation is left, and confirm
the subscription is empty. ✅ / 💳 for the registrar step.

---

## Hands-On Demo Summary

| Part | Demo | Tier |
|---|---|---|
| 5 | Budget alert, resource group, VNet, four subnets, lock | ✅ |
| 6 | NSGs + ASG, tier-to-tier rules, effective rules view | ✅ |
| 7 | SQL public access off, private endpoint, private DNS zone | ✅ |
| 8 | Three proofs the database is private | ✅ |
| 9 | Two zone-spread VMs, no public IP, cloud-init Nginx API | ✅ |
| 10 | Bastion Developer SKU connection | ✅ free |
| 11 | Internal Load Balancer, probe, rule | ✅ ~$0.72/day |
| 12 | Load balancing proved, health probe failure and recovery | ✅ |
| 13 | App Service B1, the calling app, VNet integration | ✅ ~$0.018/hr |
| 14 | The failure before integration, then the working three-tier request | ✅ |
| 15 | HTTPS only, TLS 1.2, FTP off, access restrictions | ✅ |
| 16 | Managed identity → Key Vault reference (PaaS) and IMDS read (IaaS) | ✅ |
| 17 | Azure DNS public zone, GoDaddy delegation, custom domain, managed certificate | ✅ zone / 💳 domain |
| Homework 1–3 | Diagnostic settings, KQL, application map, three alerts fired live | ✅ |
| Add-on A | Priced gap list + Defender for Cloud Secure Score | ✅ |
| Add-on B | Cost analysis, full cleanup, vault purge, registrar name servers reverted | ✅ / 💳 |

**The only 💳 steps are in Part 17**, and only because they need an owned domain. Application
Gateway, WAF, Front Door, VMSS autoscale, Azure Backup and multi-region failover are discussed with
prices attached; none are deployed.

## Summary

Good architecture is not using every service — it is using the smallest set that satisfies the brief,
and being able to defend each choice. Today that set is a VNet with four planned subnets, two VMs
across two availability zones behind an internal load balancer, an App Service that reaches them
privately, a database with **no public endpoint at all**, a vault holding the one secret, managed
identities so nothing stores a password, and one workspace that sees all of it. Every piece was
taught in the previous nineteen days. The capstone's real lesson is that the hard part was never the
clicking — it was deciding, in Part 3, which trade-offs to make, and then in Part 20, being honest
about which ones are still unmade.
