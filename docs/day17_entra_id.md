# Day 17 — Microsoft Entra ID

**Phase 5 — Identity, Security + Monitoring**

> For sixteen days you have been building things. Virtual machines, web apps, storage accounts, virtual networks, load balancers, DNS zones, VPN gateways, a SQL database. And for sixteen days there has been exactly one person allowed to touch any of it: you. Every single click you made in the portal was authorised because of one identity — the account you signed up with on Day 1. You never thought about it, because it just worked. Today we open that box. Because the moment a second person joins your project, or a second application needs to read your database, or a laptop gets stolen in an airport, "it just works" stops being good enough. Today you'll learn what actually happens in the half-second between typing your password and seeing the Azure portal — and you'll learn how to control it. Users, groups, multi-factor authentication, Conditional Access, app registrations, and the single best idea in Azure security: **managed identities**, which let one Azure service authenticate to another with no password existing anywhere at all. By the end of today, the SQL database you built yesterday will accept a connection from an app that does not know, and cannot know, any password.

---

## What You'll Learn

- **What Microsoft Entra ID actually is** — and the critical distinction between the *identity plane* and the *resource plane* that explains almost every confusing thing in Azure
- **Tenant, directory, subscription** — how the three fit together, and why a subscription can move between tenants but a user cannot
- **Why Entra ID is not "Active Directory in the cloud"** — they're genuinely different products with different protocols, and confusing them will cost you an exam question
- **The editions** — Free, P1, P2, ID Governance, Entra Suite — what each one actually unlocks and where the paywalls sit
- **Users** — member vs guest, cloud-only vs synced, UPNs, the `onmicrosoft.com` domain, custom domains, and the 30-day undo button
- **Groups** — security vs Microsoft 365, assigned vs dynamic membership, group owners, and why you should never assign permissions to a person
- **Entra roles vs Azure RBAC roles** — the single most misunderstood topic in Azure, and the toggle that bridges them
- **Authentication** — passwords, MFA, number matching, the Authentication methods policy, Temporary Access Pass, and the **September 2026 shift to passkeys by default**
- **Security defaults** — six protections, free, on by default in new tenants, and what each one blocks
- **Conditional Access** — the if/then engine that runs enterprise identity, plus report-only mode and break-glass accounts
- **Mandatory MFA for Azure** — what Microsoft turned on across every tenant, and what it means for you today
- **App registrations and service principals** — how applications get identities, client secrets vs certificates vs federated credentials, delegated vs application permissions
- **Managed identities** — system-assigned vs user-assigned, and a live passwordless connection from App Service to yesterday's SQL database
- **External identities** — inviting guests with B2B, and the retirement of Azure AD B2C
- **Devices, sign-in logs, and audit logs** — where the evidence lives when something goes wrong
- **Hybrid identity** — Entra Connect Sync, Cloud Sync, and the three authentication methods, explained for exams
- Exam framing: how AZ-104 and AZ-305 phrase identity questions, and the giveaway keywords

---

## Before We Begin

**Today is almost entirely free.** Identity is where Microsoft gives away the fundamentals and charges for the enterprise controls.

- **Microsoft Entra ID Free:** included with every Azure subscription — no separate sign-up, nothing to enable. Users, groups, security defaults, MFA via the Authenticator app, SSPR for cloud users, app registrations, enterprise applications, B2B guests, sign-in and audit logs. **✅ Free**
- **Managed identities:** free. There is no charge for creating them, using them, or the tokens they issue. Microsoft states this explicitly in the licensing docs. **✅ Free**
- **Conditional Access:** requires **Entra ID P1** (~$6/user/month list). This is the one real paywall today. **💳 Instructor demo**
- **Identity Protection, risk-based Conditional Access, PIM, access reviews:** require **Entra ID P2** (~$9/user/month list). PIM gets its own treatment on Day 18. **💳 Instructor demo**
- **Dynamic group membership, group-based licensing, custom Entra roles, SSPR with on-premises writeback:** all **P1**. **💳 Instructor demo**
- **The P2 free trial:** Microsoft offers a 30-day Entra ID P2 trial with 100 licences. If you want to follow the Conditional Access sections hands-on rather than watching, activate it — but set a calendar reminder to cancel. **✅ / 💳**

!!! warning "The one thing that can genuinely lock you out today"
    Identity is the only area of Azure where a misconfiguration can lock **you** out of **your own tenant** — and there is no "delete the resource group" undo for that. Three rules, non-negotiable:

    1. **Never** create a Conditional Access policy targeting *All users* with a *Block* control, even in report-only mode, without excluding your own account.
    2. **Always** exclude a break-glass account from every policy you create.
    3. Before you enable MFA on your admin account, **have your phone in your hand** with the Microsoft Authenticator app already installed.

    Nothing today will cost you money. But it can cost you an afternoon and a support ticket.

**Set this up before you start:**

- A resource group called `rg-day17-demo` for the compute resources we create.
- The **Microsoft Authenticator** app installed on your phone (iOS or Android, free).
- A **second browser profile or InPrivate/Incognito window** — we'll be signing in as a test user repeatedly, and you do not want to keep logging out of your admin session.
- Yesterday's `db-lwm-demo` SQL database still alive. If you deleted it, recreate it on the free offer before Part 9 — it takes about three minutes.

---

## Part 1 — What Microsoft Entra ID Actually Is

### The Half-Second You Never Think About

You type `portal.azure.com`. You enter an email and a password. You approve a prompt on your phone. The portal loads.

In that half-second, two completely separate questions were answered:

1. **Who are you?** — This is **authentication**. Proving identity. Microsoft Entra ID answers this.
2. **What are you allowed to do?** — This is **authorisation**. Granting permission. Azure RBAC answers this (Day 18).

Every security problem in every cloud reduces to those two questions. Today is question one — plus the machinery that decides *when* to ask it more forcefully.

**Microsoft Entra ID is Microsoft's cloud identity and access management service.** It's the directory that holds your users, your groups, and the identities of your applications. It's the thing that issues **tokens** — short-lived signed proofs of identity that every Azure service, every Microsoft 365 service, and thousands of third-party SaaS applications accept as evidence of who you are.

!!! note "The name"
    You will see this product called **Azure Active Directory**, **Azure AD**, and **AAD** all over the internet, in older tutorials, in PowerShell module names, and in some API responses. Microsoft renamed it to **Microsoft Entra ID** in 2023. It is the same product — no migration happened, no features changed. The rename is purely branding, because Microsoft grouped several identity products under the "Entra" family umbrella. When you read older content, mentally substitute. When you take an exam, expect the new name.

### The Two Planes — The Idea That Explains Everything

This is the single most important mental model today. Get this and half the confusing things in Azure stop being confusing.

Azure has **two separate control planes**, and they are managed in different places, by different services, with different permission systems:

| | **Identity plane** | **Resource plane** |
|---|---|---|
| **Service** | Microsoft Entra ID | Azure Resource Manager (ARM) |
| **Contains** | Users, groups, app registrations, devices | VMs, storage accounts, VNets, databases |
| **Organised by** | Tenant → directory | Subscription → resource group → resource |
| **Permissions via** | Entra roles (e.g. *User Administrator*) | Azure RBAC roles (e.g. *Contributor*) |
| **Scope of a role** | The whole tenant (or an administrative unit) | Management group / subscription / resource group / resource |
| **Lives in a region?** | **No** — it's global | **Yes** — every resource has a region |
| **Costs money?** | Free tier included; premium per-user | Per-resource, per-hour |

Look at that "lives in a region?" row. Every single thing you built for sixteen days had a region — you picked Central India or East US and the resource lived there. **Entra ID has no region.** Your tenant is a global service replicated across Microsoft's datacentres worldwide. You will never see a "location" field on a user. That's not an omission; it's the architecture.

And look at the permissions row, because this is the trap: **being a Global Administrator in Entra ID does not give you access to a single Azure resource.** Not one VM. Not one storage account. They're different planes with different role systems. We'll prove it in Part 5.

### Tenant, Directory, Subscription

Three words, constantly confused. Here they are, precisely:

- **Tenant** — your organisation's dedicated, isolated instance of Entra ID. You got one automatically when you signed up on Day 1, whether you noticed or not. It has a globally unique **Tenant ID** (a GUID) and at least one domain name ending in `.onmicrosoft.com`. A tenant is a hard security boundary: users in one tenant cannot see anything in another tenant unless explicitly invited.
- **Directory** — the actual store of objects inside the tenant: users, groups, applications, devices, service principals. In practice "tenant" and "directory" are used interchangeably, and the portal itself does so. Don't lose sleep over the distinction.
- **Subscription** — a billing and resource container that **trusts exactly one tenant** for authentication. Your resources live in the subscription; the identities allowed to touch them live in the tenant.

The relationship rules, which show up on exams almost verbatim:

- **One subscription trusts exactly one tenant.** Never two.
- **One tenant can have many subscriptions.** A large company might have a `Production` subscription, a `Dev` subscription and a `Sandbox` subscription, all trusting one corporate tenant.
- **A subscription can be transferred to a different tenant.** The resources stay; all the RBAC assignments break, because the users they referenced don't exist in the new tenant.
- **A user cannot be transferred between tenants.** Identities are tenant-bound. You invite them as a guest instead (Part 10).
- **One person can have accounts in, or be a guest in, many tenants** — that's what the directory switcher in the portal is for.

### Entra ID Is Not Active Directory

If you've ever worked in a corporate IT department, you've met **Active Directory Domain Services (AD DS)** — domain controllers, Group Policy, OUs, `DOMAIN\username`. It is tempting to assume Entra ID is that, hosted by Microsoft.

It is not. They are different products solving overlapping problems with completely different technology:

| | **Active Directory Domain Services** | **Microsoft Entra ID** |
|---|---|---|
| **Protocols** | Kerberos, NTLM, LDAP | OAuth 2.0, OpenID Connect, SAML, WS-Fed |
| **Designed for** | Devices and servers on a corporate LAN | Applications on the internet |
| **Structure** | Hierarchical — forests, domains, OUs | Flat — a single directory of objects |
| **Group Policy** | Yes | No (Intune does device policy instead) |
| **Query interface** | LDAP | Microsoft Graph REST API |
| **You join** | Domain-joined machines | Entra-joined or Entra-registered devices |

You cannot domain-join a machine to Entra ID in the AD DS sense, and there are no OUs to build a hierarchy from. If you genuinely need LDAP or Kerberos in Azure — usually for an old application that can't speak modern protocols — Azure has a separate service called **Microsoft Entra Domain Services**, which spins up managed domain controllers in your VNet. That's a niche, paid product, and it is not what we're covering today.

Most real organisations run **both**: AD DS on-premises for the office network, Entra ID in the cloud for Microsoft 365 and Azure, and a sync tool joining them. That's hybrid identity, and it's Part 13.

### Two Portals, One Tenant

Entra ID is administered from two places, and this trips up every beginner:

- **`portal.azure.com`** — the Azure portal you've used for sixteen days. Search for "Microsoft Entra ID" and you get an embedded view of the identity plane. Everything resource-related lives here.
- **`entra.microsoft.com`** — the **Microsoft Entra admin center**. A dedicated, identity-only console. Newer features land here first, the navigation is better organised, and Microsoft's own documentation now writes every set of steps as *"Sign in to the Microsoft Entra admin center → Entra ID → ..."*.

Same tenant, same data, two front doors. **I'll use the Entra admin center for identity work today** because that's where the docs point and where new features appear. Where the Azure portal path differs meaningfully, I'll say so.

### Hands-On: Find Your Tenant ✅

Let's look at the identity you've been using without thinking for sixteen days.

1. Go to **`entra.microsoft.com`** and sign in with your Azure account. **✅**
2. You land on the **Home** page. In the left navigation, expand **Entra ID** and select **Overview**. **✅**
3. On the **Overview** tab, find and note down: **✅**
   - **Tenant name** — probably something like "Default Directory" unless you changed it
   - **Tenant ID** — a GUID. Copy it somewhere. You will need this constantly: connection strings, app configuration, Terraform, CLI logins. Every tenant has exactly one and it never changes.
   - **Primary domain** — something like `yourname.onmicrosoft.com`
   - **License** — almost certainly **Microsoft Entra ID Free**
4. Click through the tabs across the top of the Overview page: **✅**
   - **Monitoring** — a live activity chart. Sign-ins in the last 24 hours, by application.
   - **Properties** — tenant display name, technical contact, and two settings we'll come back to: **Access management for Azure resources** (Part 5) and **Manage security defaults** (Part 7).
   - **Recommendations** — Microsoft's automated posture advice for your tenant. Read what it says. On a fresh tenant it usually recommends MFA for admins and designating break-glass accounts. Those recommendations are the syllabus for today.
5. Now open **`portal.azure.com`** in another tab and search for **Microsoft Entra ID**. Same tenant, same tenant ID, different chrome. **✅**
6. In the Azure portal, click your account picture (top right) → **Switch directory**. If you only have one tenant you'll see one entry — but note that this control exists. Consultants and MSPs live in this menu. **✅**

!!! tip "Write your Tenant ID down now"
    Genuinely. Put it in the same notes file as yesterday's SQL admin password. It appears in App Service configuration, Key Vault URIs, Terraform providers, `az login --tenant`, and roughly every connection string that involves Entra authentication. Looking it up thirty times is a tax you can avoid once.

---

## Part 2 — Editions: What's Free and What Isn't

Entra ID has one of the clearest free/paid splits in Azure, and knowing exactly where the line falls is worth real exam marks.

### The Four Tiers

| Edition | Cost (list) | Headline capability |
|---|---|---|
| **Entra ID Free** | Included with Azure/M365 | Users, groups, MFA via Authenticator, security defaults, SSPR for cloud users, app registrations, B2B guests, sign-in & audit logs |
| **Entra ID P1** | ~$6/user/month | **Conditional Access**, dynamic groups, group-based licensing, custom roles, SSPR with on-prem writeback, application proxy, Connect Health |
| **Entra ID P2** | ~$9/user/month | Everything in P1 plus **ID Protection** (risk-based policies), **PIM**, access reviews |
| **Entra ID Governance** | Add-on | Entitlement management, lifecycle workflows, advanced access reviews |

P1 also arrives bundled with Microsoft 365 E3 and Business Premium; P2 arrives with Microsoft 365 E5. If your organisation already pays for E5, you already own P2 and quite possibly aren't using it. **Microsoft Entra Suite** bundles P2 with Internet Access, Private Access, Verified ID and Governance.

### The Line That Matters

For this course, one line matters more than all the others:

> **Multi-factor authentication is free. Deciding *when* to require it is paid.**

You can turn MFA on for everyone in your tenant, right now, at no cost — that's security defaults. What you cannot do for free is say *"require MFA only when the user is off the corporate network, on a non-compliant device, accessing the finance app."* That conditional logic is Conditional Access, and it's P1.

Everything else follows the same pattern: the capability is free, the **granularity** is paid.

### Free Tier Object Limits

Worth knowing, and occasionally an exam detail:

- A tenant on Entra ID Free is limited to **50,000 directory objects** by default (users + groups + devices + app registrations combined).
- Add and verify a **custom domain name** and that limit rises to **300,000** automatically.
- Beyond that requires a support request.

For learning purposes, 50,000 is not a constraint you will ever brush against. For a real organisation deploying Entra-joined devices, it's closer than it sounds — every device is an object too.

### Hands-On: Check Your Licence, and Start a Trial ✅/💳

1. In the Entra admin center, go to **Entra ID → Overview** and confirm **License: Microsoft Entra ID Free**. **✅**
2. Go to **Entra ID → Licenses → All products**. On a free tenant this list is empty. This blade is where premium licences appear once purchased, and where you assign them to users or (with P1) to groups. **✅**
3. **Instructor / optional:** click **+ Try / Buy**. Microsoft offers a **30-day Entra ID P2 trial with 100 licences**. Activate it if you want to follow along with Conditional Access hands-on. **💳**
4. If you activate the trial, go to **Licenses → All products → Microsoft Entra ID P2 → Assign** and assign a licence to your own account. Features don't light up until a licence is actually assigned. **💳**

!!! warning "Set a reminder before you activate a trial"
    A 30-day trial that lapses into a paid subscription because nobody cancelled it is one of the most common ways people get an unexpected Azure bill. If you activate P2, put a calendar reminder at day 25. This is exactly the discipline we practise every day with resource-group cleanup — it applies to licences too.

---

## Part 3 — Users

### What a User Actually Is

A **user object** in Entra ID is a set of attributes — display name, user principal name, job title, department, manager, and so on — plus a set of credentials, plus a set of role assignments. It's the thing that can sign in.

The attribute you must understand is the **User Principal Name (UPN)**. It looks exactly like an email address — `priya@lwmdemo.onmicrosoft.com` — and it is the user's globally unique sign-in name. It may or may not match their actual mailbox address. The domain part **must** be a domain your tenant owns and has verified.

### Two Ways to Classify Every User

**By source:**

- **Cloud-only** — created directly in Entra ID. Entra ID owns the object; you edit it here. Everything we do today is cloud-only.
- **Synced (hybrid)** — the user exists in an on-premises Active Directory and is copied into Entra ID by a sync tool. Critical consequence: **most attributes of a synced user are read-only in the portal.** You edit them on-premises, and the change flows up on the next sync cycle. If you've ever wondered why a name field is greyed out, this is why. (Part 13.)
- **Guest (external)** — a user whose identity lives in a *different* tenant or a personal Microsoft account, invited into yours. (Part 10.)

**By type:**

- **Member** — a full member of your organisation. Default permissions let them read most directory objects.
- **Guest** — deliberately restricted. By default, a guest cannot enumerate your users and groups.

### The Domain Question

Every new tenant gets a free `something.onmicrosoft.com` domain. It works perfectly — it's a real, routable, verified domain. It just looks unprofessional on a business card.

To use your own domain (`@learnwithmithran.com`) for user sign-in names, you add it as a **custom domain** and prove ownership by creating a **TXT record** in that domain's public DNS. This is exactly the delegation-and-verification pattern you did on Day 13 with Azure DNS zones — Entra ID gives you a value, you publish it in DNS, Entra ID queries DNS and confirms you control the domain.

!!! note "Custom domains and cost"
    Adding a custom domain to Entra ID is **free** and doesn't require a premium licence. It also automatically raises your free-tier object limit from 50,000 to 300,000. If you own a domain from Day 13, adding it is a genuinely useful five-minute exercise.

### Deleting a User Is Reversible (For 30 Days)

Delete a user and they go to a **Deleted users** holding area for **30 days**, during which you can restore them completely — same object ID, same group memberships, same role assignments. After 30 days they're permanently gone.

This is one of the most useful facts on this page and a recurring exam question. Somebody leaves the company, you delete their account, three weeks later legal needs their mailbox: it's recoverable.

### Hands-On: Create a User ✅

1. In the Entra admin center: **Entra ID → Users → All users → + New user → Create new user**. **✅**
2. On the **Basics** tab: **✅**
   - **User principal name:** `priya` — then select your `.onmicrosoft.com` domain from the dropdown next to it
   - **Mail nickname:** tick *Derive from user principal name*
   - **Display name:** `Priya Sharma`
   - **Password:** click the eye icon to reveal the auto-generated password and **copy it now** — this screen never comes back. Or untick *Auto-generate password* and set your own.
   - Leave **Account enabled** ticked
3. On the **Properties** tab, fill in a few real attributes — these matter more than they look, because dynamic groups and Conditional Access can target them: **✅**
   - **Job title:** `Financial Analyst`
   - **Department:** `Finance`
   - **Usage location:** your country — **this one is not optional if you ever assign licences**, and it's the most common cause of "why won't this licence assign" tickets
4. Skip **Assignments** for now (that's Day 18). **Review + create → Create**. **✅**
5. The user appears in the list within a few seconds. Click into `Priya Sharma` and look at the profile blade: **Object ID** (her unique GUID — the thing RBAC assignments actually reference), **User principal name**, **Identities** showing *Microsoft Entra ID* as the source, **Account status**. **✅**
6. Explore the left menu of the user object — every one of these is a real administrative task you'll do eventually: **✅**
   - **Assigned roles** — Entra roles (Part 5)
   - **Groups** — memberships (Part 4)
   - **Applications** — apps she's been assigned
   - **Licenses** — premium licences
   - **Devices** — devices she's registered
   - **Authentication methods** — her registered MFA methods; as an admin you can **Reset password** and **Require re-register multifactor authentication** here. This is the helpdesk blade.
   - **Monitoring → Sign-in logs** — every sign-in she's attempted, filtered to her

### Hands-On: Sign In as Priya — and Discover She Can See Nothing ✅

This is the demo that makes the two-planes model click. Do not skip it.

1. Open a **private/incognito window** (or a second browser profile). **✅**
2. Go to `portal.azure.com` and sign in as `priya@yourtenant.onmicrosoft.com` with the password you copied. **✅**
3. You'll be forced to change the password on first sign-in. Set a new one and note it. **✅**
4. If security defaults are enabled on your tenant (very likely — Part 7), you'll be prompted to **register for MFA**. You can complete it with the Authenticator app, or click **Skip for now** if that option appears. **✅**
5. You're in the Azure portal. Now go to **Subscriptions**. **✅**

**The list is empty.**

Priya is a fully valid, fully authenticated user in your tenant. She got a token. Entra ID has confirmed exactly who she is. And she can see **nothing** — not one resource group, not one VM, not the SQL database.

That's the two planes, live. **Authentication succeeded. Authorisation was never granted.** Entra ID said "this is definitely Priya." Azure RBAC said "and Priya has no permissions on anything." Both are working perfectly.

Fixing that is Day 18. Understanding *why* it happens is today.

6. While you're signed in as Priya, go to **`myaccount.microsoft.com`**. **✅** This is the **My Account** portal — the self-service page every end user in every Entra tenant gets. From here she can manage her own **Security info** (MFA methods), see her **Devices**, view her **Organisations**, and check her **Sign-in activity**. Knowing this URL exists saves your helpdesk enormous time.
7. Sign out of the private window, but keep it handy — we'll use it again.

### Hands-On: Add a Custom Domain ✅ (optional — needs a real domain)

Skip this if you don't own a domain. If you followed Day 13 and have one, this takes five minutes.

1. **Entra ID → Domain names → + Add custom domain**. Enter your domain, e.g. `learnwithmithran.com`. Click **Add domain**. **✅**
2. Entra ID shows you a **TXT record** to create — a hostname (`@`) and a value starting `MS=ms########`. **✅**
3. Go to your DNS provider — or your **Azure DNS public zone** from Day 13 — and create that TXT record. **✅**
4. Wait for DNS propagation (usually a minute or two; TTL applies exactly as you learned on Day 13), then come back and click **Verify**. **✅**
5. Once verified, you can create users with `@learnwithmithran.com` UPNs, and you can set it as the **primary domain** so new users default to it. **✅**

---

## Part 4 — Groups

### Never Assign Permissions to a Person

That's the rule. Here's why.

Assign permissions directly to users, and after eighteen months you have four hundred individual assignments scattered across sixty resource groups, nobody knows who has access to what, someone leaves and their access lingers in nine places, and the new starter's onboarding is a two-hour archaeology exercise.

Assign permissions to **groups**, and access becomes a membership question. New analyst joins? Add to `grp-finance-readers`. Leaves? Remove. Their access appears and disappears everywhere, instantly, in one action. Auditing becomes "who is in this group" instead of "let me check sixty resource groups."

This scales from a two-person team to a two-hundred-thousand-person enterprise, and it's the correct answer to a surprising number of exam questions.

### Two Group Types

| | **Security group** | **Microsoft 365 group** |
|---|---|---|
| **Purpose** | Granting access — RBAC, app assignment, Conditional Access targeting | Collaboration — shared mailbox, calendar, SharePoint site, Teams team |
| **Can contain** | Users, devices, service principals, other groups | Users only (including guests) |
| **Creates M365 resources?** | No | Yes — mailbox, SharePoint site, and more |
| **Use in Azure RBAC?** | **Yes** | Not the normal path |
| **Use for Azure work** | **Always this one** | Rarely |

For everything in this course, **security group** is the answer. Microsoft 365 groups are a collaboration construct that happens to live in the same directory.

### Two Membership Types

- **Assigned** — you add and remove members by hand. Available on **all editions, including Free**.
- **Dynamic** — you write a rule, and Entra ID maintains membership automatically. Requires **Entra ID P1**.

Dynamic rules are query expressions over user attributes:

```text
(user.department -eq "Finance")

(user.department -eq "Finance") and (user.country -eq "India")

(user.jobTitle -contains "Analyst") and (user.accountEnabled -eq true)

(user.userPrincipalName -contains "#EXT#@")   // all guest users
```

The power is operational: HR changes someone's department in the HR system, the change syncs to Entra ID, and within minutes their group memberships — and therefore their access — reshape automatically. Nobody files a ticket. Nobody forgets.

**Dynamic device groups** work the same way over device attributes (`device.deviceOSType -eq "Windows"`), which is how Intune targets policy at fleets.

!!! note "Dynamic groups are not instant"
    Rule evaluation typically takes a few minutes and can take longer in large tenants. Don't design a workflow that assumes membership updates in real time.

### Group Owners and Nesting

- **Owners** can manage membership without being a directory admin. This is proper delegation — the Finance manager owns `grp-finance`, adds and removes their own people, and never needs User Administrator rights.
- **Nesting** — a group can contain another group. Supported, and useful, but be careful: Azure RBAC honours nested groups, while some other services do not. Two levels is plenty; deeper nesting becomes impossible to reason about.

### Hands-On: Create a Security Group ✅

1. **Entra ID → Groups → All groups → + New group**. **✅**
2. Configure: **✅**
   - **Group type:** *Security*
   - **Group name:** `grp-finance-team`
   - **Group description:** `Finance department — read access to finance resources`
   - **Membership type:** *Assigned* — open the dropdown first and note that *Dynamic User* and *Dynamic Device* are listed but greyed out or flagged as requiring a premium licence. **That greyed-out dropdown is the P1 paywall, visible.**
3. Under **Owners**, click **No owners selected** → add yourself. **✅**
4. Under **Members**, click **No members selected** → add `Priya Sharma`. **✅**
5. **Create**. **✅**
6. Open the group and note the **Object ID** — this is what an RBAC assignment will reference tomorrow. Explore the left menu: **Members**, **Owners**, **Roles and administrators** (a group can hold an Entra role), **Azure role assignments** (Day 18 lives here), **Licenses** (group-based licensing, P1). **✅**

### Hands-On: Try a Dynamic Group 💳

1. **+ New group** → **Group type:** *Security*, **Name:** `grp-dynamic-finance`. **✅**
2. Set **Membership type** to **Dynamic User**. Without a premium licence, the portal blocks you here — that's the paywall, and it's worth showing on camera. **💳**
3. With P1/P2, click **Add dynamic query**. You get a rule builder: **Property** = `department`, **Operator** = `Equals`, **Value** = `Finance`. **💳**
4. Switch to the **Rule syntax** editor to see what the builder generated:
   ```text
   (user.department -eq "Finance")
   ```
   **💳**
5. Click **Validate Rules**, add `Priya Sharma` as a test user, and confirm she matches — she has `Department: Finance` from Part 3. **💳**
6. **Save → Create.** Wait a few minutes and check **Members**. Priya appears without anyone adding her. **💳**
7. To really land it: edit Priya's department to `Marketing`, wait, and watch her leave the group by herself. **💳**

---

## Part 5 — Entra Roles vs Azure RBAC Roles

Day 18 is entirely about Azure RBAC. But there's one distinction that belongs today, because it is *the* classic Azure misunderstanding and it is fundamentally an identity concept.

### Two Role Systems, Not One

| | **Entra ID roles** (directory roles) | **Azure RBAC roles** |
|---|---|---|
| **Control** | The identity plane | The resource plane |
| **Examples** | Global Administrator, User Administrator, Application Administrator, Billing Administrator | Owner, Contributor, Reader, Storage Blob Data Reader |
| **Let you** | Create users, reset passwords, register apps, configure Conditional Access | Create VMs, read blobs, restart App Services, delete resource groups |
| **Assigned at** | Tenant scope (or an administrative unit) | Management group / subscription / resource group / resource |
| **Managed in** | Entra ID → Roles & admins | Any resource → Access control (IAM) |

**A Global Administrator has total control over identity and zero access to Azure resources.** They can create a hundred users, reset every password, and delete the entire directory — and still not be able to see a single virtual machine.

Conversely, a subscription **Owner** can do anything to every resource in that subscription, and cannot create a user.

Two systems. Two blades. Two answers.

### The Bridge: The Elevation Toggle

There is exactly one connection between them, and it's a switch:

**Entra ID → Overview → Properties → Access management for Azure resources.**

Flip this to **Yes** for your account, and your Global Administrator identity is granted the **User Access Administrator** RBAC role at the **root management group** — meaning across every subscription in the tenant.

This is the emergency door. Its purpose is exactly one scenario: someone left the company, they were the only Owner on a subscription, and nobody else can get in. A Global Admin elevates, grants themselves Owner, fixes the situation, and **switches the toggle back off**.

!!! warning "Turn the toggle back off"
    Leaving elevation on permanently means every Global Administrator silently holds User Access Administrator over every subscription in your tenant — an enormous, invisible privilege escalation. Elevate, fix, revert. Microsoft's own guidance treats this as a break-glass mechanism, not a configuration.

### Hands-On: Explore Roles ✅

1. **Entra ID → Roles & admins → All roles**. Scan the list — there are over a hundred built-in Entra roles. Read a few names: *User Administrator*, *Application Administrator*, *Authentication Administrator*, *Conditional Access Administrator*, *Global Reader*. **✅**
2. Click **Global Administrator → Assignments** and confirm your own account is listed. That's how you've been able to do everything for sixteen days. **✅**
3. Click **Global Reader**. Read the description: everything Global Admin can *see*, nothing it can *change*. **This is the single most underused role in Azure.** Auditors, analysts and new team members should get Global Reader, not Global Administrator. **✅**
4. Assign a role, properly scoped: open **User Administrator → + Add assignments** → select `Priya Sharma` → **Add**. Priya can now create and manage users, and still cannot see a single Azure resource. **✅**
5. Now the bridge: **Entra ID → Overview → Properties**. Find **Access management for Azure resources**. Read the description carefully. **Leave it set to No.** **✅**
6. To feel the difference, open **Subscriptions → your subscription → Access control (IAM) → Role assignments**. Completely different blade, completely different role list, completely different service. **✅**

!!! tip "The exam giveaway"
    If a question mentions creating users, resetting passwords, registering applications, or configuring Conditional Access → **Entra role**. If it mentions creating, reading, modifying or deleting Azure resources → **Azure RBAC role**. If it says "manage users *and* virtual machines," the answer is **two separate assignments**.

---

## Part 6 — Authentication: Passwords, MFA, and the Move to Passkeys

### Why Passwords Lose

A password is a shared secret, and shared secrets leak. They get phished, reused across sites, breached in bulk from some unrelated service, sprayed against your tenant, and typed into convincing fake login pages.

Microsoft's published figure is the one to remember: **multi-factor authentication blocks over 99% of identity-based attacks.** Not because the second factor is unbreakable, but because it breaks the attacker's economics — a stolen password alone stops being enough.

### What Counts as a Factor

Real MFA needs factors from **different categories**:

- **Something you know** — password, PIN
- **Something you have** — phone, hardware security key, certificate
- **Something you are** — fingerprint, face

Password plus security question is *not* MFA — both are things you know. Password plus Authenticator push *is*.

### The Authentication Methods, Ranked

Not all second factors are equal. Ranked from strongest to weakest:

| Method | Strength | Notes |
|---|---|---|
| **Passkeys (FIDO2)** — security key, phone, or Windows Hello | **Phishing-resistant** | The credential is cryptographically bound to the real domain. A fake login page cannot use it, because the browser won't release it. |
| **Certificate-based authentication (CBA)** | Phishing-resistant | Smart cards; common in government and regulated industries |
| **Microsoft Authenticator (push + number matching)** | Strong | The free default for most organisations |
| **OATH TOTP codes** (Authenticator or any authenticator app) | Strong | 6-digit rotating codes; works offline |
| **SMS / voice call** | **Weak** | Vulnerable to SIM swapping and interception. **Being retired — see below.** |

**Number matching** deserves a word. Older push notifications had one button: Approve. Attackers exploited that with "MFA fatigue" — spam someone with prompts at 3am until they tap Approve to make it stop. Number matching fixes it: the sign-in screen shows a two-digit number, and you must type *that number* into the app. You can't approve a prompt you didn't initiate, because you can't see the number. This has been on by default since 2023.

!!! info "The big one: passkeys become the default from September 2026"
    This is happening right now, so it belongs in your head as current, not future:

    - From **1 September 2026**, Microsoft begins rolling out **passkeys as the default authentication experience** in Entra ID. Users currently enabled for SMS or voice will be automatically enabled for passkeys via the Authentication methods policy and prompted to register at their next MFA.
    - Microsoft is **retiring SMS and voice MFA** as native Entra methods on this timeline.
    - From **1 February 2027**, users relying solely on SMS or voice will get a **blocking** passkey-registration prompt.

    The practical takeaway for everything you build from here: **do not design around SMS.** Register a passkey or the Authenticator app. If you're advising an organisation, this migration is a 2026 project, not a someday project.

### Self-Service Password Reset (SSPR)

Password resets are, by a wide margin, the largest single category of helpdesk tickets in most organisations. SSPR removes them: the user proves identity with registered methods and resets their own password at `aka.ms/sspr`.

- **Cloud-only users:** SSPR is **free**.
- **Synced users writing back to on-premises AD:** requires **P1** (that's "SSPR with writeback").

Configuration decisions that matter: how many methods are required to reset (one or two), which methods count, and whether users are forced to register at next sign-in.

### Temporary Access Pass (TAP)

A **Temporary Access Pass** is a time-limited passcode an admin generates for a user who has no working credentials — a brand-new starter on day one, or someone who lost their phone and their only MFA method with it.

The TAP satisfies MFA on its own, exactly once (or for a limited window), specifically so the user can register a real method. It's the answer to the passwordless chicken-and-egg problem: how do you register a passkey when registering requires MFA and you have none? A TAP.

### Hands-On: The Authentication Methods Policy ✅

1. **Entra ID → Authentication methods → Policies**. This is the modern, unified control panel for every credential type in your tenant. **✅**
2. Look at the list: **Passkey (FIDO2)**, **Microsoft Authenticator**, **SMS**, **Temporary Access Pass**, **Third-party software OATH tokens**, **Voice call**, **Email OTP**, **Certificate-based authentication**, **Hardware OATH tokens**, **QR code (preview)**. Each has an Enabled state and a Target. **✅**
3. Click **Microsoft Authenticator**. Set **Enable** to *Yes*, **Target** to *All users*. Expand **Configure** and look at the options: **✅**
   - **Require number matching for push notifications** — leave enabled
   - **Show application name in push** and **Show geographic location in push** — enable both; they let a user see *"someone in Brazil is trying to sign in to the Azure portal"* and reject it
   - **Authentication mode:** *Any*, *Push*, or *Passwordless*
4. Click **Passkey (FIDO2)**. Enable it, target *All users*. Note the settings: **Allow self-service set up**, **Enforce attestation**, and **Enforce key restrictions** for organisations that mandate specific hardware. **✅**
5. Click **Temporary Access Pass**. Set **Enable** to *Yes*, target *All users*, and read the options: **Minimum/Maximum lifetime**, **Default lifetime** (1 hour), **One-time use** (a strong default), **Length**. **✅**
6. Click **SMS** and **Voice call**. Read the state they're in. Given the September 2026 change, this is the moment on camera to say plainly: *if these are your organisation's primary methods, you have a migration project.* **✅**

!!! warning "Don't disable methods while security defaults are on"
    Microsoft's documentation is explicit: if security defaults are enabled, leave the available methods alone. Disabling methods in that state can lock every user — including you — out of the tenant.

### Hands-On: Register MFA on Your Own Account ✅

1. In your normal browser, go to **`aka.ms/mfasetup`** (or `myaccount.microsoft.com` → **Security info**). **✅**
2. Click **+ Add sign-in method**. **✅**
3. Choose **Microsoft Authenticator** → **Add**, and follow the QR-code flow in the phone app. **✅**
4. Complete the test prompt — note the **number matching** screen: the browser shows a two-digit number, and you type it into the app. **✅**
5. If you have a phone or laptop that supports it, also add **Passkey** or **Face/Fingerprint** — this is the method Microsoft is standardising on, and being able to demo it on camera is worth the two minutes. **✅**
6. Back in **Security info**, set your **default sign-in method** to the strongest one you registered. **✅**

### Hands-On: Configure SSPR ✅

1. **Entra ID → Password reset → Properties**. **✅**
2. Set **Self service password reset enabled** to **Selected**, then choose `grp-finance-team` — the group you built in Part 4. This is groups doing their job: a policy targeted at a group instead of a list of names. **✅**
3. Go to **Authentication methods** (within Password reset): set **Number of methods required to reset** to **1** for the demo (**2** is the correct production answer), and tick **Mobile app notification** and **Email**. **✅**
4. Go to **Registration**: set **Require users to register when signing in** to **Yes**, with a **60 day** re-confirmation interval. **✅**
5. Go to **Notifications**: enable **Notify users on password resets** and **Notify all admins when other admins reset their password**. Both are free early-warning signals for account takeover. **✅**
6. **Save**, then test it: in your private window, go to **`aka.ms/sspr`** and run the reset flow as Priya. **✅**

---

## Part 7 — Security Defaults and Conditional Access

### Security Defaults: Free, Blunt, and Better Than Nothing

**Security defaults** is a single on/off switch that applies a fixed bundle of protections to your entire tenant. It's free, it requires no configuration, and if your tenant was created after October 2019 it's very likely **already on**.

What it enforces — the current list, and each item is worth knowing:

1. **All users must register for MFA** (no grace period since July 2024)
2. **Administrators must do MFA every time they sign in** — covering 14+ privileged roles
3. **Users must do MFA when necessary** — Microsoft decides, based on risk signals like location and device
4. **Legacy authentication protocols are blocked** — IMAP, POP3, SMTP AUTH, old Office clients. Over 99% of password-spray attacks arrive over these, and they cannot do MFA, so they're a permanent MFA bypass. Blocking them is the single highest-value item on this list.
5. **Device code flow is blocked** — an authentication flow designed for input-limited devices (smart TVs, conference room systems) and heavily abused in phishing. *New tenants created from 1 July 2026 block this by default.*
6. **Privileged activity is protected** — MFA required for the Azure portal, Entra admin center, Azure PowerShell and Azure CLI

Its limitation is that it's binary. On or off, everyone or nobody. No exclusions, no exceptions, no service accounts, no trusted locations. Which is fine for a small organisation and unworkable for a large one.

!!! note "Security defaults and Conditional Access are mutually exclusive"
    You cannot run both. Turning on a Conditional Access policy requires turning security defaults off — and vice versa. That's a real exam question. If you disable security defaults, **enable equivalent Conditional Access policies immediately**; the gap between the two is a window with the door open.

### Mandatory MFA for Azure — Already Here

Independently of security defaults, Microsoft has rolled out **mandatory MFA across all tenants** for Azure administration:

- **Phase 1** (from October 2024, 100% of tenants by March 2025): MFA required for sign-in to the **Azure portal**, **Microsoft Entra admin center**, and **Intune admin center**.
- **Phase 2** (from October 2025): MFA required at the **Azure Resource Manager layer** — meaning **Azure CLI, Azure PowerShell, the REST APIs, the mobile app, SDKs, and Infrastructure-as-Code tools like Terraform and Bicep**. Tenants with technical blockers could postpone to **1 July 2026**, which has now passed.

Read that Phase 2 list again, because it matters for the rest of this course. When we reach **Bicep on Day 26** and **Terraform on Day 27**, those deployments authenticate through ARM, and **that authentication requires MFA**. Read-only operations may not prompt, but anything that creates or modifies will.

This is not something you opt into. It's the floor now, for everybody.

### Conditional Access: The If/Then Engine

Conditional Access is where enterprise identity actually lives. Requires **P1**.

The model is a sentence:

> **IF** these *assignments* are true, **THEN** apply these *access controls*.

**Assignments — the IF side:**

- **Users and groups** — who it applies to, and critically, who is **excluded**
- **Target resources** — which cloud apps, user actions, or authentication contexts
- **Conditions:**
  - **Sign-in risk** and **user risk** (P2 — Identity Protection)
  - **Device platforms** — iOS, Android, Windows, macOS, Linux
  - **Locations** — named IP ranges or countries
  - **Client apps** — browser, mobile app, or *legacy authentication clients*
  - **Device state / filter for devices** — compliant, Entra hybrid joined, or a custom device filter

**Access controls — the THEN side:**

- **Grant:** Block access, or Grant with requirements — require MFA, require **authentication strength** (e.g. *phishing-resistant MFA only*), require a compliant device, require an Entra hybrid joined device, require an approved client app, require terms of use
- **Session:** limit what happens after sign-in — sign-in frequency (re-authenticate every N hours), persistent browser session, app-enforced restrictions, Conditional Access App Control

Some policies you'd find in essentially every mature tenant:

| Policy | What it does |
|---|---|
| Require MFA for administrators | Privileged roles always do MFA |
| Block legacy authentication | Kills the MFA bypass permanently |
| Require MFA for Azure management | Protects ARM specifically |
| Require compliant device for corporate apps | Unmanaged laptops can't reach company data |
| Block access from unexpected countries | If you have no staff there, no one signs in from there |
| Require phishing-resistant MFA for admins | Passkeys or CBA only — no SMS, no push |
| Sign-in frequency for high-risk apps | Re-authenticate every 4 hours in the finance app |

### The Two Rules That Prevent Disaster

**1. Report-only mode.** Every new policy should be created in **Report-only** first. It evaluates against real sign-ins and logs *what would have happened*, without enforcing anything. Let it run for a few days, check the sign-in logs, confirm you haven't caught the backup service account or the CEO's iPad, and only then flip to On.

**2. Break-glass accounts.** Create **two** cloud-only Global Administrator accounts with long random passwords, using the `.onmicrosoft.com` domain, belonging to no individual, stored securely offline. **Exclude them from every Conditional Access policy.** Monitor them for any sign-in at all — a break-glass account signing in should page someone.

These exist for the day a policy misfires, a federation provider goes down, or the last admin's phone dies in a different timezone. This is not paranoia; it's Microsoft's documented recommendation and standard practice in every serious organisation.

### Microsoft-Managed Policies

Microsoft now deploys **Microsoft-managed Conditional Access policies** directly into eligible tenants. They arrive in **Report-only** state, and if left untouched Microsoft enables them no sooner than **30 days** later, with email and Message Center notice two weeks ahead.

Current ones include: block legacy authentication, block device code flow, MFA for admins accessing admin portals, MFA for all users, MFA for per-user-MFA users, MFA and reauthentication for risky sign-ins, and block/remediate high-risk users.

You **can** exclude accounts from them and turn them on or off. You **cannot** rename or delete them. They show `Microsoft` in the **Created by** column. If you need more control, duplicate one and edit the copy.

This matters for your break-glass discipline: **exclude your emergency accounts from Microsoft-managed policies too.**

### Hands-On: Security Defaults ✅

1. **Entra ID → Overview → Properties → Manage security defaults**. **✅**
2. Read the current state. On a tenant created any time recently, it's **Enabled**. **✅**
3. Read every bullet on the panel — this is the free security baseline for the entire Microsoft cloud, and it's worth reading aloud on camera. **✅**
4. **If you don't have P1: leave it Enabled and move on.** It's doing real work. **✅**
5. If you're following the Conditional Access lab with a P1/P2 trial, set it to **Disabled** — and understand you have just removed your tenant's baseline protection and must replace it in the next ten minutes. **💳**

### Hands-On: Build a Conditional Access Policy 💳

Requires P1 or P2. Students without it: watch, then open the blade read-only and look at the templates — the blade is visible even without a licence.

1. **Entra ID → Conditional Access → Overview**. Note the summary tiles and the policy count. **💳**
2. Go to **Policies**. If Microsoft-managed policies are present, open one and note **Created by: Microsoft**, the **Report-only** state, and the **Policy impact** tab. **💳**
3. Click **+ New policy from template**. Browse the template gallery — Microsoft ships ~20 templates grouped by scenario (*Secure foundation*, *Zero Trust*, *Remote work*, *Protect administrators*, *Emerging threats*). **Templates are the right starting point** — they encode Microsoft's recommended configuration. **💳**
4. Now build one by hand so you understand every field. **+ New policy**: **💳**
   - **Name:** `CA01 - Require MFA for Admins - ReportOnly`
   - **Users → Include → Directory roles** → select *Global Administrator*, *User Administrator*, *Security Administrator*
   - **Users → Exclude → Users and groups** → **select your own account**. Say this out loud on camera. Every single time.
   - **Target resources → Resources (formerly cloud apps) → Include → All resources**
   - **Conditions** — open each one and read it: *User risk* and *Sign-in risk* (greyed out without P2), *Device platforms*, *Locations*, *Client apps*, *Filter for devices*. Don't set any yet.
   - **Grant → Grant access → Require multifactor authentication**
   - Open **Require authentication strength** instead and look at the built-in strengths: *MFA*, *Passwordless MFA*, *Phishing-resistant MFA*. **This is how you mandate passkeys** — and given the September 2026 direction, this is the control to know.
   - **Enable policy: Report-only**
5. **Create.** **💳**
6. Sign in as Priya in the private window, then come back to **Entra ID → Monitoring & health → Sign-in logs**, open her sign-in event, and click the **Conditional Access** tab. You'll see your policy listed with a **Report-only** result — *would have required MFA*. Nothing was enforced. That's the whole point of report-only. **💳**
7. Create a **named location**: **Conditional Access → Named locations → + Countries location** → name it `Blocked Countries` and select a few countries where you have no staff. Then note you could build a policy that blocks sign-ins from it. **Don't enable a block policy today.** **💳**
8. Explore **Conditional Access → Insights and reporting** — a workbook showing exactly how your policies would affect real sign-ins, before you enforce anything. **💳**

---

## Part 8 — Applications: App Registrations and Service Principals

### Applications Need Identities Too

Everything so far has been about humans. But most authentication in a modern system isn't human at all — it's a web app calling an API, a script reading a storage account, a CI/CD pipeline deploying infrastructure at 2am with nobody watching.

Those need identities. In Entra ID, that's **workload identities**, and there are three flavours: app registrations, service principals, and managed identities.

### App Registration vs Service Principal vs Enterprise Application

This confuses everybody, so let's be precise.

- **App registration** — the **global definition** of your application. Its name, its redirect URIs, its permissions, its credentials, its supported account types. There is exactly **one** app registration, and it lives in the tenant where the app was created. Think of it as the blueprint.
- **Service principal** — the **local instance** of that application in a specific tenant. It's what actually gets permissions and role assignments. Registering an app in your own tenant creates both the registration and a service principal automatically. When a *different* organisation uses your multi-tenant app, a service principal is created in *their* tenant, pointing at your registration.
- **Enterprise application** — what the portal calls a service principal in the UI. Same object, different blade. **App registrations** is the developer view; **Enterprise applications** is the administrator view.

The blueprint-and-instance analogy is the one that sticks: one blueprint, many buildings, one per tenant.

### Three Ways an App Proves Who It Is

| Credential | Security | Reality |
|---|---|---|
| **Client secret** | Weakest | A string. Expires (max 24 months, and Microsoft now defaults new secrets to short lifetimes). Ends up in config files, source control, and Slack messages. |
| **Certificate** | Better | Private key, harder to leak accidentally. Still something you must store, protect and rotate. |
| **Federated credential** (workload identity federation) | **Best** | **No secret exists at all.** An external issuer — GitHub Actions, Kubernetes, another Azure identity — presents its own token, Entra ID trusts the issuer, and exchanges it for an Azure token. |

**Workload identity federation is the modern answer** for CI/CD. When we build pipelines on Days 23–24, this is how a GitHub Actions workflow or an Azure DevOps service connection should authenticate to Azure — no secret in the pipeline, nothing to rotate, nothing to leak. Note that a managed identity can also be used as a federated credential on an app registration (up to 20 per app).

### Delegated vs Application Permissions

When an app requests access to an API (Microsoft Graph, say), there are two fundamentally different kinds of permission:

- **Delegated permissions** — the app acts **on behalf of a signed-in user**. Effective access is the **intersection** of what the app was granted and what the user is allowed. A delegated `Files.Read` lets the app read the files *that user* can read — not everyone's.
- **Application permissions** — the app acts **as itself**, with no user present. An application `Files.Read.All` reads **every file in the tenant**. There's no user to constrain it.

Application permissions always require **admin consent**, and they're the ones to scrutinise in a security review. An over-permissioned application permission is one of the most common serious findings in a real Entra tenant.

### Hands-On: Register an Application ✅

1. **Entra ID → App registrations → + New registration**. **✅**
2. Configure: **✅**
   - **Name:** `app-lwm-demo`
   - **Supported account types:** *Accounts in this organizational directory only (Single tenant)* — the right default for almost everything
   - **Redirect URI:** leave blank; we're not building a sign-in flow today
3. **Register.** **✅**
4. On the **Overview** page, note the two GUIDs you'll use constantly: **✅**
   - **Application (client) ID** — the app's public identifier
   - **Directory (tenant) ID** — your tenant, same GUID from Part 1
   - Also note **Object ID** and the link to **Managed application in local directory** — click it and you land in **Enterprise applications**, looking at the service principal. Same app, other view. Go back.
5. **Certificates & secrets → Client secrets → + New client secret**. Description `demo-secret`, expiry 3 months. Click **Add**. **✅**
6. **Copy the Value immediately.** Not the Secret ID — the **Value**. It is displayed exactly once and never again; if you lose it you delete the secret and create another. Say this on camera, because everyone learns it the hard way. **✅**
7. Look at the other two tabs on that blade: **Certificates** and **Federated credentials**. Click **+ Add credential** under Federated credentials and read the scenario dropdown — *GitHub Actions deploying Azure resources*, *Kubernetes accessing Azure resources*, *Managed identity*, *Other issuer*. **Bookmark this screen mentally for Day 23.** **✅**
8. **API permissions**: note `Microsoft Graph → User.Read` was added automatically (delegated). Click **+ Add a permission → Microsoft Graph** and compare the two tiles side by side: **✅**
   - **Delegated permissions** — "your application needs to access the API as the signed-in user"
   - **Application permissions** — "your application runs as a background service or daemon without a signed-in user"
   
   Select **Application permissions → User.Read.All**, add it, and observe the warning: **Not granted**. Click **Grant admin consent for &lt;tenant&gt;** and watch the status go green. **You just gave a background application the ability to read every user in your directory.** Sit with that for a second — this is how over-privileged apps happen. **✅**
9. **Enterprise applications → All applications** and find `app-lwm-demo`. From here an admin controls **Users and groups** (who can use it), **Single sign-on**, **Provisioning**, **Conditional Access** (yes — CA policies can target applications), and **Permissions**. **✅**

### Hands-On: Sign In as the Application ✅

A rare, justified use of the CLI — because there's genuinely no portal equivalent for "authenticate as an application."

1. Open **Cloud Shell** (the `>_` icon in the Azure portal), Bash. **✅**
2. Sign in as the service principal:
   ```bash
   az login --service-principal \
     --username <application-client-id> \
     --password <the-secret-value-you-copied> \
     --tenant <your-tenant-id>
   ```
   **✅**
3. It succeeds — Entra ID authenticated a non-human identity. Now try to do something:
   ```bash
   az group list
   ```
   **The list is empty.** Exactly like Priya in Part 3. **Authenticated, unauthorised.** The app has an identity and zero RBAC. Same lesson, different identity type. **✅**
4. Return to your own identity:
   ```bash
   az login
   ```
   **✅**

!!! warning "You just handled a real credential"
    That secret string, in your Cloud Shell history, is a password to your tenant. In production it belongs in **Key Vault** (Day 19) — or better, it shouldn't exist at all, which is exactly what the next part is about. Delete this app registration at cleanup.

---

## Part 9 — Managed Identities: The Best Idea in Azure Security

### The Problem, Stated Plainly

You just created a client secret. Now answer these honestly:

- Where does it live in production? A config file? An environment variable? A pipeline variable?
- Who has read it? Who could read it?
- What happens in 3 months when it expires — is there a calendar reminder, or does the app just break at 2am?
- If a developer leaves, do you rotate it? Do you rotate *all* of them?
- Is it in your git history right now?

Every one of those is a real operational burden, and every one of them is a real breach vector. Credentials in configuration are the single most common cause of cloud compromise.

**Managed identities delete the entire problem.**

### What a Managed Identity Is

A managed identity is a **service principal that Azure creates and manages for an Azure resource**, where the credentials are handled entirely by the platform, are automatically rotated, and are **never exposed to you or to anyone else**.

You cannot read a managed identity's credential. There's no secret to copy, no certificate to download, nothing to put in a config file, nothing to leak, nothing to rotate. The credential material is not accessible — by design, to you included.

And they cost **nothing**. Microsoft's licensing documentation states it explicitly: managed identities have no licensing requirement and no extra cost.

### How It Works Under the Hood

Worth understanding, because it makes the whole thing concrete:

1. You enable a managed identity on a resource — a VM, App Service, Function App, Logic App, AKS cluster, Container App, Data Factory, and so on.
2. Azure creates a service principal in your Entra tenant, tied to that resource.
3. The Azure platform injects the credential into the resource's environment where only that resource can reach it. On a VM, that's the **Instance Metadata Service** at the link-local address `169.254.169.254` — reachable from inside the VM and from nowhere else.
4. Code inside the resource calls that local endpoint (usually via the Azure Identity SDK or `DefaultAzureCredential`, which handles it transparently) and receives an **Entra ID access token**.
5. The code presents that token to the target service — Storage, SQL, Key Vault, Cosmos DB, anything supporting Entra authentication.
6. The target service validates the token and checks the identity's RBAC permissions.

No password crosses the wire. No password exists in your code. Token acquisition and refresh are handled by the SDK.

### System-Assigned vs User-Assigned

| | **System-assigned** | **User-assigned** |
|---|---|---|
| **Created** | On the resource, as part of it | As a standalone Azure resource |
| **Lifecycle** | Dies with the resource — delete the VM, the identity is deleted automatically | Independent — you create and delete it yourself |
| **Shared?** | **No** — exactly one resource | **Yes** — attach to many resources |
| **Name** | Always the same as the resource | Whatever you name it |
| **Best for** | A single workload on a single resource | Fleets — a scale set, several apps sharing one permission set, or a blue/green pattern |

Microsoft's current recommendation leans toward **user-assigned** for most service scenarios, and the reason is operational: permissions survive the resource. Recycle your VMs weekly and a system-assigned identity means re-granting RBAC every time. With user-assigned, the identity and its permissions persist while the compute comes and goes.

For a single app talking to a single database, system-assigned is simpler and perfectly correct.

### Hands-On: Create a User-Assigned Managed Identity ✅

1. In the Azure portal, search **Managed Identities** → **+ Create**. **✅**
2. **Resource group:** `rg-day17-demo`, **Region:** your usual, **Name:** `id-lwm-shared`. **Review + create → Create**. **✅**
3. Open it. Note the **Client ID** and **Object (principal) ID** — and note what's *missing*: **there is no credentials blade, no secrets tab, nothing to copy.** That absence is the entire product. **✅**
4. Look at **Azure role assignments** (empty — Day 18 fills it) and **Federated credentials** (the modern CI/CD path from Part 8). **✅**

### Hands-On: Passwordless App Service → SQL Database ✅

The main event. We take the SQL database from yesterday and connect to it from an App Service **with no password anywhere**.

**Step 1 — Create an App Service** (skip if you still have one from Day 6):

1. Search **App Services → + Create → Web App**. **✅**
2. **Resource group:** `rg-day17-demo`, **Name:** `app-lwm-day17-<yourname>` (globally unique), **Publish:** Code, **Runtime:** .NET 8 (LTS) or Node 20 LTS, **OS:** Linux, **Region:** same region as your SQL server. **✅**
3. **Pricing plan:** **Free F1**. **✅**
4. **Review + create → Create.** **✅**

**Step 2 — Give the app an identity:**

5. Open the App Service → **Settings → Identity**. **✅**
6. On the **System assigned** tab, flip **Status** to **On** → **Save** → **Yes**. **✅**
7. An **Object (principal) ID** appears. Behind the scenes, Entra ID just created a service principal named exactly `app-lwm-day17-<yourname>`. **✅**
8. Click the **User assigned** tab and note you *could* attach `id-lwm-shared` here instead. A resource can have both. **✅**

**Step 3 — Make the database trust Entra ID:**

9. Go to your SQL **logical server** (`sql-lwm-<yourname>`) → **Settings → Microsoft Entra ID**. **✅**
10. Confirm an **Entra admin** is set — you set this on Day 16. If not: **Set admin** → pick your account → **Save**. **✅**
11. Check **Support only Microsoft Entra authentication for this server**. Leave it **unchecked** today — ticking it disables SQL logins entirely, which is the right production posture but would break yesterday's `sqladmin` login mid-lesson. Mention that it exists and what it does. **✅**

**Step 4 — Create the database user for the identity:**

12. Go to the **database** `db-lwm-demo` → **Query editor (preview)**. **✅**
13. **Sign in with Microsoft Entra authentication** — the button beneath the SQL login fields. You must be signed in as the Entra admin for this to work. **✅**
14. Run this, substituting your app's exact name:
    ```sql
    CREATE USER [app-lwm-day17-<yourname>] FROM EXTERNAL PROVIDER;
    ALTER ROLE db_datareader ADD MEMBER [app-lwm-day17-<yourname>];
    ALTER ROLE db_datawriter ADD MEMBER [app-lwm-day17-<yourname>];
    ```
    **✅**

    Read that first line carefully, because it's the whole idea: **`FROM EXTERNAL PROVIDER` means "this user's credentials are managed by Microsoft Entra ID, not by SQL Server."** There is no `WITH PASSWORD` clause. There is no password. The database is being told to trust Entra ID's word about who this is.

15. Verify:
    ```sql
    SELECT name, type_desc, authentication_type_desc
    FROM sys.database_principals
    WHERE type IN ('E','X');
    ```
    Your app appears with `authentication_type_desc = EXTERNAL`. **✅**

**Step 5 — Wire up the connection string:**

16. Go back to the App Service → **Settings → Environment variables → Connection strings → + Add**. **✅**
17. **Name:** `DefaultConnection`, **Type:** `SQLAzure`, **Value:**
    ```text
    Server=tcp:sql-lwm-<yourname>.database.windows.net,1433;Database=db-lwm-demo;Authentication=Active Directory Default;Encrypt=True;
    ```
    **✅**
18. **Apply → Apply** to save and restart. **✅**

Now compare that string to yesterday's:

```text
Server=tcp:...;Database=...;User ID=sqladmin;Password=Sup3rS3cret!;Encrypt=True;
```
```text
Server=tcp:...;Database=...;Authentication=Active Directory Default;Encrypt=True;
```

**The password is gone.** Not hidden, not encrypted, not vaulted — *gone*. There is no password. `Authentication=Active Directory Default` tells the SQL client library to use `DefaultAzureCredential`, which detects it's running in App Service, grabs a token from the managed identity endpoint, and presents that token to SQL. Nothing to rotate, nothing to leak, nothing to commit to git by accident.

**Step 6 — The shortcut, for real projects:**

19. Still in the App Service, go to **Settings → Service Connector → + Create**. **✅**
20. Select **Service type:** *SQL Database*, pick your server and database, and for **Authentication type** choose **System assigned managed identity**. **✅**
21. Walk through the wizard. Service Connector does all of Steps 2–5 for you — enables the identity, creates the database user, and writes the connection string. **Do the manual version first so you understand what it's doing**, then use this in real life. **✅**

### Hands-On: See the Token (Optional, Needs a VM) ✅

If you have a Linux VM from Day 3 still around, this makes the abstraction physical.

1. Open the VM → **Security → Identity → System assigned → On → Save**. **✅**
2. SSH into the VM and run:
   ```bash
   curl -s -H "Metadata: true" \
     "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"
   ```
   **✅**
3. You get back JSON containing an `access_token` — a JWT, valid roughly 24 hours, issued by Entra ID to this VM's identity. **✅**
4. Paste it into **`jwt.ms`** (Microsoft's token decoder) and read the claims: `aud` (audience — who it's for), `iss` (issuer — your tenant), `oid` (the identity's object ID), `exp` (expiry). **✅**
5. Now the important part: **that endpoint is only reachable from inside the VM.** `169.254.169.254` is a link-local address. From your laptop, from another VM, from anywhere else on earth — unreachable. That's the isolation the whole design rests on. **✅**

!!! tip "The rule to take away"
    **If both ends are Azure services, use a managed identity.** App Service to SQL, Function to Storage, VM to Key Vault, AKS to Container Registry, Data Factory to a data lake — managed identity, every time. Reach for a client secret only when one end genuinely isn't an Azure resource, and even then, check whether workload identity federation covers it first.

---

## Part 10 — External Identities: Guests and B2B

### The Problem

You hire a contractor. An auditor needs read access for six weeks. A partner company's developers need to use your app. Do you create them accounts in your tenant, with your passwords, that you now have to remember to delete?

No. **Entra External ID / B2B collaboration** lets you invite someone using **their own existing identity** — their work account in their own tenant, or a personal Microsoft account. They authenticate **at home**, in their own organisation, under their own organisation's MFA and Conditional Access policies. You just grant access.

The consequences are excellent:

- You never manage their password
- Their home organisation disables their account when they leave — and their access to your resources dies with it, automatically
- Their MFA is enforced by their employer
- You can still apply *your* Conditional Access policies on top

### How It Works

1. You invite `someone@partnercompany.com`.
2. A **guest user object** is created in your tenant with UPN `someone_partnercompany.com#EXT#@yourtenant.onmicrosoft.com` — note the `#EXT#` marker, which is how you identify guests in reports and dynamic group rules.
3. They receive an invitation email and redeem it.
4. From then on they sign in with *their own* credentials, and a token from their tenant is accepted by yours.

Guests are **restricted by default** — the default guest permission level prevents them enumerating your users and groups. Tighten it further under **External Identities → External collaboration settings**.

**Cross-tenant access settings** are the enterprise control layer: per-partner-tenant inbound and outbound rules, and — genuinely useful — the ability to **trust MFA claims from the partner's tenant**, so guests aren't forced to register a *second* MFA method with you.

### Azure AD B2C Is Retired

Worth knowing because it appears in older material and older exam banks:

- **Azure AD B2C** was the consumer-facing identity product — sign-up/sign-in for your customers, with social logins.
- It went **end-of-sale to new customers on 1 May 2025**, and **existing tenants retired on 15 March 2026**.
- The successor is **Microsoft Entra External ID**, which handles both B2B (partners) and B2C-style customer identity in one product. Core features are free for the first **50,000 monthly active users**.

If you read a tutorial telling you to create an Azure AD B2C tenant, it's out of date.

### Hands-On: Invite a Guest ✅

1. **Entra ID → Users → All users → + New user → Invite external user**. **✅**
2. **Email:** use a personal address you control (Gmail, Outlook.com — anything). **Display name:** `External Auditor`. Add a personal message. **✅**
3. Optionally add them to `grp-finance-team` right here on the **Assignments** tab — invite and grant in one step. **✅**
4. **Review + invite → Invite**. **✅**
5. Check the inbox, open the invitation, click **Accept**, and complete the redemption flow. **✅**
6. Back in **All users**, note the new entry: **User type: Guest**, and the `#EXT#` UPN. **✅**
7. Go to **External Identities → External collaboration settings** and read the controls: **✅**
   - **Guest user access restrictions** — three levels, from "same as members" to "limited access, cannot enumerate directory objects". The most restrictive is the right default.
   - **Guest invite settings** — who is allowed to invite guests. On a fresh tenant, *anyone in the organization* can. Restricting this to admins is a common hardening step.
   - **Collaboration restrictions** — allow-list or deny-list specific partner domains
8. Go to **External Identities → Cross-tenant access settings** and look at **Default settings** for inbound and outbound, plus the **Trust settings** where MFA-claim trust lives. **✅**

---

## Part 11 — Devices

Briefly, because it matters for Conditional Access and shows up on AZ-104.

Entra ID doesn't only hold users and apps — it holds **devices**. Three states:

| State | What it means | Typical use |
|---|---|---|
| **Entra registered** | Personal device, added to Entra ID with a work account. The user owns it; the org gets limited management. | BYOD — personal phones and laptops |
| **Entra joined** | Corporate device joined directly to Entra ID. No on-premises AD involved. Sign in with your work account. | Cloud-first organisations |
| **Microsoft Entra hybrid joined** | Device joined to on-premises AD **and** registered in Entra ID. | Organisations with existing AD infrastructure |

Why it matters: a Conditional Access policy can **require a compliant device** or **require an Entra hybrid joined device**. That's how an organisation says *"our data is reachable only from laptops we manage"* — the single most effective control against credential theft, because a stolen password on an unmanaged machine gets nowhere.

Compliance itself is evaluated by **Microsoft Intune**, which is outside this course's scope but sits right next door.

### Hands-On: Device Settings ✅

1. **Entra ID → Devices → All devices**. Probably empty on a lab tenant — that's fine. **✅**
2. **Devices → Device settings**. Read the controls: who may join devices, whether MFA is required to join, the maximum devices per user (default 50), and **local administrator settings** — including the Entra-joined *Global Administrator/Device Administrator* local-admin behaviour and **Azure AD Joined Device Local Administrator** role. **✅**

---

## Part 12 — Sign-In Logs and Audit Logs

When something goes wrong with identity, this is where the evidence lives. Both are **free**.

### Sign-In Logs

Every authentication attempt, successful or failed. Per entry you get: user, application, timestamp, IP address, geographic location, client app, device, **Conditional Access policies evaluated and their results**, MFA method used, and — most valuable — a **failure reason with an error code**.

Four categories, worth knowing by name because the blade separates them:

- **Interactive user sign-ins** — a human typed credentials
- **Non-interactive user sign-ins** — a client used a cached token or refresh token on the user's behalf
- **Service principal sign-ins** — an application authenticated with its own credential
- **Managed identity sign-ins** — an Azure resource used its managed identity

That third and fourth category are why the App Service demo in Part 9 is auditable: **you can see your app's identity signing in**, with timestamps.

Two error codes you will meet:

- **AADSTS50126** — invalid username or password
- **AADSTS50076** — MFA required by a Conditional Access policy (this one usually means everything is working)

### Audit Logs

Every **change** made to the directory: user created, group membership modified, role assigned, app registered, policy updated, secret added. Each entry records the initiator, the target, the timestamp and the result. This is your answer to *"who added that person to the admin group?"*

### Retention — The Catch

Free tenants retain sign-in and audit logs for **7 days**. Premium tenants get **30 days**. That's it.

Seven days is not an audit trail. For anything real, configure **Diagnostic settings** on the logs to export them to a **Log Analytics workspace** (query them with KQL), a **storage account** (cheap long-term archive), or **Event Hub** (feed a SIEM).

That's **Day 20 — Azure Monitor**, and it's exactly the same diagnostic-settings pattern you'll use for every other Azure service. Identity logs are just another source.

### Hands-On: Read the Logs ✅

1. **Entra ID → Monitoring & health → Sign-in logs**. **✅**
2. Find your own sign-in from the start of the session. Click it and work through the tabs: **Basic info**, **Location** (map), **Device info**, **Authentication Details** (which method satisfied MFA and when), **Conditional Access** (which policies evaluated and their results), **Report-only** results. **✅**
3. Find **Priya's** sign-in from Part 3. Note the app she signed into and the result. **✅**
4. Switch to the **Service principal sign-ins** tab and find `app-lwm-demo` from Part 8 — your CLI service-principal login is right there. **✅**
5. If you did the App Service demo, check **Managed identity sign-ins** too. **✅**
6. Go to **Audit logs** and filter **Service: Core Directory**. You'll see today's entire session as an audit trail: *Add user*, *Add group*, *Add member to group*, *Add application*, *Add service principal*, *Update policy*. **This is what an auditor reads.** **✅**
7. Open **Monitoring & health → Diagnostic settings → + Add diagnostic setting**. Look at the log categories available (`SignInLogs`, `AuditLogs`, `NonInteractiveUserSignInLogs`, `ServicePrincipalSignInLogs`, `ManagedIdentitySignInLogs`) and the destinations. **Don't create it today** — we'll do it properly with a workspace on Day 20. **✅**

---

## Part 13 — Hybrid Identity (Concepts)

Most enterprises don't start in the cloud. They have twenty years of on-premises Active Directory, and they are not throwing it away. Hybrid identity connects the two so a user has **one identity and one password** across both.

### The Sync Tools

- **Microsoft Entra Connect Sync** — the mature, feature-rich agent installed on a Windows Server on-premises. Supports complex filtering, attribute transformation, and password writeback. One instance per tenant (plus a staging server).
- **Microsoft Entra Cloud Sync** — a lightweight agent where most configuration lives in the cloud. Supports **multiple disconnected AD forests** (a big deal after mergers), needs no Windows Server infrastructure beyond the agent, and is Microsoft's direction of travel. Doesn't yet cover every advanced Connect Sync scenario.

### The Three Authentication Methods

This distinction is a reliable exam question:

| Method | Where the password is verified | Key characteristic |
|---|---|---|
| **Password Hash Synchronisation (PHS)** | **In the cloud** — a hash of the password hash is synced to Entra ID | Simplest, most resilient. If your on-prem environment is offline, cloud sign-in still works. **Microsoft's recommended default.** |
| **Pass-Through Authentication (PTA)** | **On-premises** — an agent validates against your domain controller in real time | No password material in the cloud. But if on-prem is down, cloud sign-in fails. |
| **Federation (AD FS)** | **On-premises**, by a dedicated federation farm | Most control, most infrastructure, most to maintain. Microsoft actively encourages moving off it. |

**Seamless SSO** can be layered onto PHS or PTA so domain-joined machines on the corporate network sign in without a prompt.

The remembering trick: **PHS = cloud verifies. PTA = on-prem verifies, no farm. Federation = on-prem verifies, with a farm.** And when a question says "must continue to work if the on-premises environment is unavailable," the answer is **PHS**.

---

## Part 14 — Exam Framing

Identity is heavily weighted on both AZ-104 and AZ-305. The questions are pattern-matched to keywords:

| If the question says… | The answer is… |
|---|---|
| "Assign permissions to Azure resources" | **Azure RBAC role** — not an Entra role |
| "Create users / reset passwords / register apps" | **Entra ID role** (User Administrator, Application Administrator) |
| "Membership must update automatically based on department" | **Dynamic group** (requires P1) |
| "Require MFA only from outside the corporate network" | **Conditional Access** with a named location (requires P1) |
| "Simplest way to require MFA for everyone, no premium licence" | **Security defaults** |
| "An Azure service must access another Azure service securely, no credentials" | **Managed identity** |
| "Identity must persist across VM recreation / be shared by several resources" | **User-assigned managed identity** |
| "Identity should be deleted when the resource is deleted" | **System-assigned managed identity** |
| "Grant an external partner access without creating them an account" | **B2B guest invitation** |
| "Just-in-time, time-bound admin access with approval" | **PIM** (P2 — Day 18) |
| "Automatically respond to risky sign-ins" | **Identity Protection / risk-based Conditional Access** (P2) |
| "Must keep working if on-premises servers are unavailable" | **Password Hash Synchronisation** |
| "No password material may be stored in the cloud" | **Pass-Through Authentication** |
| "Retain sign-in logs beyond 30 days" | **Diagnostic settings → Log Analytics / storage account** |
| "Recover a user deleted last week" | **Deleted users** — restorable for 30 days |
| "A pipeline must deploy to Azure with no stored secret" | **Workload identity federation** (federated credential) |
| "Grant an application read access to all mailboxes without a user" | **Application permission** + admin consent |
| "App should only see files the signed-in user can see" | **Delegated permission** |

---

## Part 15 — Cleanup ✅

Identity cleanup is different from resource cleanup. Deleting a resource group does **not** remove anything from the directory — users, groups, app registrations and guests all live in the tenant, not in a subscription.

1. **Delete the resource group:** search **Resource groups** → `rg-day17-demo` → **Delete resource group**. This removes the App Service, the App Service plan, and the user-assigned managed identity. **✅**
2. **Delete the app registration:** **Entra ID → App registrations → app-lwm-demo → Delete**. This kills the client secret you created. Do it. **✅**
3. **Check Enterprise applications** for a leftover service principal with the same name and delete it too if present. **✅**
4. **Decide about Priya.** She costs nothing and she's genuinely useful tomorrow — Day 18 assigns her an RBAC role and proves the difference between the two role systems. **My recommendation: keep her.** Same for `grp-finance-team`. **✅**
5. **Remove the Entra role assignment** you gave Priya in Part 5 (**Roles & admins → User Administrator → Assignments → remove**), unless you want to keep it for tomorrow's comparison. **✅**
6. **Remove the guest** if you don't want it: **Users → External Auditor → Delete**. **✅**
7. **If you enabled MFA on your own account — keep it.** Genuinely. It's free, it's the single highest-value security change you can make to your Azure account, and Microsoft now mandates it for the portal and for ARM anyway. **✅**
8. **If you activated a P1/P2 trial**, put the cancellation reminder in your calendar right now, on camera. **💳**
9. **If you disabled security defaults** for the Conditional Access lab and you're *not* keeping premium licences: **turn security defaults back on**. Leaving a tenant with neither security defaults nor Conditional Access is worse than where you started. **💳**
10. **Keep `db-lwm-demo`.** It's on the free offer and Day 19 (Key Vault) and Day 31 (Capstone) both use it. **✅**

---

## Summary

Today you stopped asking "how do I build it?" and started asking "who is allowed to touch it?"

**Microsoft Entra ID is the identity plane** — a global, region-less service holding users, groups, applications and devices, completely separate from the resource plane where your VMs and databases live. Those two planes have different containers, different role systems and different administrative blades. The moment you internalise that split, Azure permissions stop being confusing: **a Global Administrator can't see a virtual machine, and a subscription Owner can't create a user.** We proved it twice — once with Priya, once with a service principal, both perfectly authenticated and completely unauthorised.

**Users and groups** are the raw material, and the operating rule is permanent: **never assign permissions to a person.** Assign to a group, manage the group. Dynamic membership (P1) takes it further, letting HR data drive access automatically.

**Authentication is where the value is.** MFA blocks over 99% of identity attacks and it's free. Security defaults deliver six real protections — MFA for everyone, MFA for admins, blocked legacy authentication, blocked device code flow, protected privileged activity — with one switch, at no cost. And Microsoft has now made MFA **mandatory** for the Azure portal and for the Resource Manager layer, which means Azure CLI, PowerShell, Terraform and Bicep, on every tenant. That's the floor now.

The direction of travel is clear and it's happening this month: **passkeys become the default authentication experience from 1 September 2026**, and SMS and voice MFA are being retired. Design for phishing-resistant methods, not for text messages.

**Conditional Access** (P1) is the enterprise engine — if these conditions, then these controls — and the two habits that keep you safe with it are **report-only mode** and **break-glass accounts excluded from every policy**.

**Applications get identities too**, through app registrations and service principals, authenticating with a client secret, a certificate, or — the modern answer — a federated credential with no secret at all.

And **managed identities** are the best security idea in Azure. You watched a connection string lose its password entirely. Not encrypted, not vaulted — *gone*. `Authentication=Active Directory Default`, a `CREATE USER ... FROM EXTERNAL PROVIDER` in the database, and an App Service that authenticates to SQL with a credential that no human can read, that rotates itself, and that costs nothing. Every time both ends of a connection are Azure services, that's the answer.

### What's Next

Priya can sign in and see nothing. Your app has an identity with database permissions but no Azure permissions. Both are stuck at exactly the same place: **authenticated, unauthorised.**

Tomorrow, **Day 18 — Azure RBAC** finishes the sentence. Roles, role definitions, role assignments, scope and inheritance from management group down to individual resource, the difference between control-plane and data-plane permissions (which is why a subscription Owner still can't read a blob), custom roles, deny assignments, and **Privileged Identity Management** — where admin rights are handed out just-in-time, for a fixed window, with approval and an audit trail, instead of permanently.

Every identity you created today is about to get something to do.

---

## Key Takeaways

- **Two planes.** Entra ID (identity) and Azure Resource Manager (resources) are separate systems with separate role models. Global Administrator ≠ Owner. Entra ID has no region; it's global.
- **Tenant / directory / subscription:** one subscription trusts exactly one tenant; one tenant can hold many subscriptions. Subscriptions can move between tenants (breaking all RBAC); users cannot.
- **Entra ID is not Active Directory.** OAuth/OIDC/SAML vs Kerberos/LDAP, flat vs hierarchical, Graph vs LDAP, no Group Policy. Different products.
- **Authentication ≠ authorisation.** A valid user with no RBAC sees an empty portal. That's both systems working correctly.
- **The licensing line: MFA is free; deciding when to require it is P1.** Conditional Access, dynamic groups, group-based licensing, custom roles = P1. Identity Protection, PIM, access reviews = P2.
- **Free tier limit:** 50,000 directory objects, rising to 300,000 automatically once you verify a custom domain.
- **Never assign permissions to a person — assign to a group.** Access becomes a membership question, and offboarding becomes one action.
- **Security group for RBAC**, Microsoft 365 group for collaboration. Assigned membership is free; dynamic membership is P1.
- **Deleted users are recoverable for 30 days**, with object ID, group memberships and role assignments intact.
- **Security defaults enforce six things** free: MFA registration for all, MFA for admins, risk-based MFA for users, blocked legacy authentication, blocked device code flow, and MFA for privileged/ARM access. Mutually exclusive with Conditional Access.
- **MFA is mandatory for Azure now.** Phase 1 covered the portals; Phase 2 (from October 2025, postponement deadline 1 July 2026) covers the ARM layer — CLI, PowerShell, REST, SDKs, Terraform and Bicep.
- **Passkeys become the default from 1 September 2026**, with SMS and voice MFA being retired; a blocking registration prompt lands 1 February 2027. Don't design around SMS.
- **Number matching defeats MFA fatigue** — you can't approve a prompt whose number you can't see.
- **Conditional Access = IF (users + resources + conditions) THEN (grant/session controls).** Always start in **report-only**. Always keep **two break-glass Global Admin accounts excluded from every policy**, including Microsoft-managed ones.
- **App registration = the blueprint** (one, global). **Service principal = the instance** (one per tenant). **Enterprise application = the portal's name for the service principal.**
- **Client secret value is shown exactly once.** Prefer certificates; prefer **federated credentials** (workload identity federation) over both — no secret exists at all.
- **Delegated permission** = acts as the signed-in user (intersection of app and user rights). **Application permission** = acts as itself, tenant-wide, always needs admin consent.
- **Managed identities are free, have no readable credential, and rotate themselves.** System-assigned dies with its resource and can't be shared; user-assigned is independent and shareable — and is the better default when compute is recycled frequently.
- **`Authentication=Active Directory Default` plus `CREATE USER [name] FROM EXTERNAL PROVIDER`** is the complete passwordless recipe for App Service → Azure SQL.
- **Tokens come from `169.254.169.254`** — a link-local address reachable only from inside the resource. That isolation is what makes the whole model safe.
- **B2B guests use their own identity from their own tenant** — their employer manages their password and MFA, and their access dies when their home account does. Guest UPNs contain `#EXT#`.
- **Azure AD B2C is retired** (existing tenants ended 15 March 2026). **Microsoft Entra External ID** is the successor.
- **Sign-in and audit logs are free but retained only 7 days** (30 on premium). Export via diagnostic settings to Log Analytics for anything real — that's Day 20.
- **Hybrid: PHS = cloud verifies** (most resilient, Microsoft's recommendation), **PTA = on-prem verifies with an agent**, **Federation = on-prem verifies with an AD FS farm**.
