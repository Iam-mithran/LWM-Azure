# Day 17 — Microsoft Entra ID & Azure RBAC

**Phase 5 — Identity, Security + Monitoring**

> For sixteen days you have been building things. Virtual machines, web apps, storage accounts, virtual networks, load balancers, DNS zones, VPN gateways, a SQL database. And for sixteen days there has been exactly one person allowed to touch any of it: you. Every single click you made in the portal was authorised because of one identity — the account you signed up with on Day 1. You never thought about it, because it just worked. Today we open that box, and we open it all the way. Because the moment a second person joins your project, or a second application needs to read your database, or a laptop gets stolen in an airport, "it just works" stops being good enough. Today you'll learn what actually happens in the half-second between typing your password and seeing the Azure portal — **and then what happens in the half-second after that**, when Azure decides whether you're allowed to do the thing you just clicked. Those are two different questions, answered by two different systems, and almost every confusing thing in Azure security comes from mixing them up. By the end of today you will have created a user who can sign in and see nothing, given her exactly the access she needs and not one permission more, watched her hit a wall trying to read a file she can technically "contribute" to, built a custom role from scratch, constrained an admin so they can only hand out one specific role to one specific group, and made an App Service talk to a SQL database with **no password existing anywhere in the world**.

---

## What You'll Learn

**Part One — Identity (who are you?)**

- **What Microsoft Entra ID actually is** — and the critical distinction between the *identity plane* and the *resource plane* that explains almost every confusing thing in Azure
- **Tenant, directory, subscription** — how the three fit together, and why a subscription can move between tenants but a user cannot
- **Why Entra ID is not "Active Directory in the cloud"** — genuinely different products, different protocols
- **The editions** — Free, P1, P2, ID Governance, Entra Suite — and exactly where the paywalls sit
- **Users** — member vs guest, cloud-only vs synced, UPNs, custom domains, and the 30-day undo button
- **Groups** — security vs Microsoft 365, assigned vs dynamic membership, and why you should never assign permissions to a person
- **Authentication** — MFA, number matching, the Authentication methods policy, Temporary Access Pass, and the **September 2026 shift to passkeys by default**
- **Security defaults and Conditional Access** — the free baseline and the paid if/then engine
- **App registrations, service principals, and managed identities** — how software proves who it is, ending with a live passwordless connection from App Service to yesterday's SQL database
- **External identities, devices, sign-in logs, and hybrid identity**

**Part Two — Authorisation (what are you allowed to do?)**

- **The three-part sentence** — security principal + role definition + scope = role assignment, and why every RBAC question reduces to it
- **Scope and inheritance** — management group → subscription → resource group → resource, and the additive evaluation model
- **Role definitions under the hood** — `Actions`, `NotActions`, `DataActions`, `NotDataActions`, `AssignableScopes`, and the subtraction that produces your effective permissions
- **Built-in roles** — Owner, Contributor, Reader, User Access Administrator, and the **Role Based Access Control Administrator** role you should be using instead
- **Job function roles vs privileged administrator roles** — Microsoft's own classification, and why it matters for your security posture
- **Control plane vs data plane** — the single most surprising thing in Azure: why a Contributor on a storage account still cannot read a blob
- **RBAC for workload identities** — giving a managed identity a role, and watching a VM read a file with no credential
- **Custom roles** — cloning, building from scratch, the JSON, and the limits
- **ABAC conditions** — attribute-based access control, condition templates, and **constrained delegation** of role assignment rights
- **Deny assignments, Azure Policy, and the retirement of Azure Blueprints**
- **Privileged Identity Management** — just-in-time, time-bound, approval-gated admin access
- **Auditing, troubleshooting and the hard limits** — 4,000, 500, 5,000, and the ten-minute rule
- Exam framing for AZ-104 and AZ-305, with the giveaway keywords for both halves

---

## Before We Begin

**Today is almost entirely free.** Identity and access control are where Microsoft gives away the fundamentals and charges for the enterprise controls.

- **Microsoft Entra ID Free:** included with every Azure subscription. Users, groups, security defaults, MFA via the Authenticator app, SSPR for cloud users, app registrations, enterprise applications, B2B guests, sign-in and audit logs. **✅ Free**
- **Azure RBAC in its entirety:** role assignments, all built-in roles, custom roles, ABAC conditions, constrained delegation. Microsoft's docs say it plainly — *"Using this feature is free and included in your Azure subscription."* There is no premium tier for RBAC. **✅ Free**
- **Azure Policy:** free for the built-in policy definitions and effects we'll use. **✅ Free**
- **Managed identities:** free. No charge to create them, use them, or issue their tokens. **✅ Free**
- **Conditional Access:** requires **Entra ID P1** (~$6/user/month list). **💳 Instructor demo**
- **Identity Protection, PIM, access reviews:** require **Entra ID P2** (~$9/user/month list) or Entra ID Governance. **💳 Instructor demo**
- **Dynamic group membership, group-based licensing, custom *Entra* roles:** all **P1**. (Custom *Azure RBAC* roles are free — don't confuse the two.) **💳 Instructor demo**
- **The P2 free trial:** Microsoft offers a 30-day Entra ID P2 trial with 100 licences. Activate it if you want to follow the Conditional Access and PIM sections hands-on — but set a calendar reminder to cancel. **✅ / 💳**

!!! warning "The one thing that can genuinely lock you out today"
    Identity is the only area of Azure where a misconfiguration can lock **you** out of **your own tenant** — and there is no "delete the resource group" undo for that. Four rules, non-negotiable:

    1. **Never** create a Conditional Access policy targeting *All users* with a *Block* control, even in report-only mode, without excluding your own account.
    2. **Always** exclude a break-glass account from every policy you create.
    3. Before you enable MFA on your admin account, **have your phone in your hand** with the Microsoft Authenticator app already installed.
    4. **Never remove your own Owner role assignment** at subscription scope "just to see what happens." Azure will usually stop you — the last Owner assignment on a subscription is protected — but don't go looking for the edge case.

    Nothing today will cost you money. But it can cost you an afternoon and a support ticket.

**Set this up before you start:**

- A resource group called `rg-day17-demo` for the resources we create.
- The **Microsoft Authenticator** app installed on your phone (iOS or Android, free).
- A **second browser profile or InPrivate/Incognito window** — we'll be signing in as a test user constantly, and you do not want to keep logging out of your admin session. Genuinely, set this up. Half of today's payoff moments happen in that window.
- Yesterday's `db-lwm-demo` SQL database still alive. If you deleted it, recreate it on the free offer before Part 8 — it takes about three minutes.
- Optional but excellent: a **Linux VM** from Day 3 still around. Part 17 uses it for the best demo of the day.

!!! tip "This is a long one — and it's meant to be"
    Entra ID and Azure RBAC used to be two separate videos in this course. They're one video now, because teaching them apart is teaching them wrong. Every single time I explained authentication without authorisation, the same question came back: *"I created the user, why can't they see anything?"* That question has an answer, and the answer is the second half of this video. Take it in two sittings if you need to — Part 12 is the natural halfway point.

---

# PART ONE — IDENTITY

---

## Part 1 — What Microsoft Entra ID Actually Is

### The Half-Second You Never Think About

You type `portal.azure.com`. You enter an email and a password. You approve a prompt on your phone. The portal loads.

In that half-second, two completely separate questions were answered:

1. **Who are you?** — This is **authentication**. Proving identity. Microsoft Entra ID answers this.
2. **What are you allowed to do?** — This is **authorisation**. Granting permission. Azure RBAC answers this.

Every security problem in every cloud reduces to those two questions. Part One is question one. Part Two is question two. And the reason they're in the same video is that neither one is useful on its own — an identity with no permissions can't do anything, and a permission with no identity to attach it to isn't a permission at all.

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
| **Organised by** | Tenant → directory | Management group → subscription → resource group → resource |
| **Permissions via** | Entra roles (e.g. *User Administrator*) | Azure RBAC roles (e.g. *Contributor*) |
| **Scope of a role** | The whole tenant (or an administrative unit) | Management group / subscription / resource group / resource |
| **Managed in** | Entra ID → Roles & admins | Any resource → Access control (IAM) |
| **Lives in a region?** | **No** — it's global | **Yes** — every resource has a region |
| **Costs money?** | Free tier included; premium per-user | Per-resource, per-hour (RBAC itself is free) |

Look at that "lives in a region?" row. Every single thing you built for sixteen days had a region — you picked Central India or East US and the resource lived there. **Entra ID has no region.** Your tenant is a global service replicated across Microsoft's datacentres worldwide. You will never see a "location" field on a user. That's not an omission; it's the architecture.

Interestingly, **Azure RBAC data is global too** — role definitions, role assignments and deny assignments are replicated to every region, even though the resources they protect are regional. Microsoft's reasoning is straightforward: ARM has a global endpoint, so authorisation has to be enforceable everywhere. If your teammate in the United States needs access to a VM you created in East Asia, the role assignment has to be readable from wherever they're calling.

And look at the permissions row, because this is the trap: **being a Global Administrator in Entra ID does not give you access to a single Azure resource.** Not one VM. Not one storage account. They're different planes with different role systems. We'll prove it in Part 12.

### Tenant, Directory, Subscription

Three words, constantly confused. Here they are, precisely:

- **Tenant** — your organisation's dedicated, isolated instance of Entra ID. You got one automatically when you signed up on Day 1, whether you noticed or not. It has a globally unique **Tenant ID** (a GUID) and at least one domain name ending in `.onmicrosoft.com`. A tenant is a hard security boundary: users in one tenant cannot see anything in another tenant unless explicitly invited.
- **Directory** — the actual store of objects inside the tenant: users, groups, applications, devices, service principals. In practice "tenant" and "directory" are used interchangeably, and the portal itself does so. Don't lose sleep over the distinction.
- **Subscription** — a billing and resource container that **trusts exactly one tenant** for authentication. Your resources live in the subscription; the identities allowed to touch them live in the tenant.

The relationship rules, which show up on exams almost verbatim:

- **One subscription trusts exactly one tenant.** Never two.
- **One tenant can have many subscriptions.** A large company might have a `Production` subscription, a `Dev` subscription and a `Sandbox` subscription, all trusting one corporate tenant.
- **A subscription can be transferred to a different tenant.** The resources stay; **all the RBAC assignments are permanently deleted**, because the users they referenced don't exist in the new tenant. Managed identities have to be recreated by hand too. This is one of the nastiest surprises in Azure and it's worth saying out loud twice.
- **A user cannot be transferred between tenants.** Identities are tenant-bound. You invite them as a guest instead (Part 9).
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

Most real organisations run **both**: AD DS on-premises for the office network, Entra ID in the cloud for Microsoft 365 and Azure, and a sync tool joining them. That's hybrid identity, and it's Part 11.

### Two Portals, One Tenant

Entra ID is administered from two places, and this trips up every beginner:

- **`portal.azure.com`** — the Azure portal you've used for sixteen days. Search for "Microsoft Entra ID" and you get an embedded view of the identity plane. Everything resource-related — and therefore everything in Part Two of today — lives here.
- **`entra.microsoft.com`** — the **Microsoft Entra admin center**. A dedicated, identity-only console. Newer features land here first, the navigation is better organised, and Microsoft's own documentation now writes every set of steps as *"Sign in to the Microsoft Entra admin center → Entra ID → ..."*.

Same tenant, same data, two front doors. **I'll use the Entra admin center for identity work and the Azure portal for RBAC work today** — which is exactly the split you'll use in real life.

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
   - **Properties** — tenant display name, technical contact, and two settings we'll come back to: **Access management for Azure resources** (Part 12) and **Manage security defaults** (Part 6).
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
| **Entra ID P1** | ~$6/user/month | **Conditional Access**, dynamic groups, group-based licensing, custom Entra roles, SSPR with on-prem writeback, application proxy, Connect Health |
| **Entra ID P2** | ~$9/user/month | Everything in P1 plus **ID Protection** (risk-based policies), **PIM**, access reviews |
| **Entra ID Governance** | Add-on | Entitlement management, lifecycle workflows, advanced access reviews |

P1 also arrives bundled with Microsoft 365 E3 and Business Premium; P2 arrives with Microsoft 365 E5. If your organisation already pays for E5, you already own P2 and quite possibly aren't using it. **Microsoft Entra Suite** bundles P2 with Internet Access, Private Access, Verified ID and Governance.

### The Line That Matters

For this course, one line matters more than all the others:

> **Multi-factor authentication is free. Deciding *when* to require it is paid.**

You can turn MFA on for everyone in your tenant, right now, at no cost — that's security defaults. What you cannot do for free is say *"require MFA only when the user is off the corporate network, on a non-compliant device, accessing the finance app."* That conditional logic is Conditional Access, and it's P1.

Everything else follows the same pattern: the capability is free, the **granularity** is paid.

!!! warning "Don't let this confuse you in Part Two"
    Entra ID has a paid tier. **Azure RBAC does not.** Every role assignment, every built-in role, every custom role, every ABAC condition in Part Two of this video is included free with your subscription. The only paid thing in the second half is **PIM**, and PIM isn't RBAC — it's a P2 governance layer *on top of* RBAC. If an exam question asks "which licence do you need to create a custom RBAC role," the answer is **none**.

### Free Tier Object Limits

Worth knowing, and occasionally an exam detail:

- A tenant on Entra ID Free is limited to **50,000 directory objects** by default (users + groups + devices + app registrations combined).
- Add and verify a **custom domain name** and that limit rises to **300,000** automatically.
- Beyond that requires a support request.

For learning purposes, 50,000 is not a constraint you will ever brush against. For a real organisation deploying Entra-joined devices, it's closer than it sounds — every device is an object too.

### Hands-On: Check Your Licence, and Start a Trial ✅/💳

1. In the Entra admin center, go to **Entra ID → Overview** and confirm **License: Microsoft Entra ID Free**. **✅**
2. Go to **Entra ID → Licenses → All products**. On a free tenant this list is empty. This blade is where premium licences appear once purchased, and where you assign them to users or (with P1) to groups. **✅**
3. **Instructor / optional:** click **+ Try / Buy**. Microsoft offers a **30-day Entra ID P2 trial with 100 licences**. Activate it if you want to follow along with Conditional Access and PIM hands-on. **💳**
4. If you activate the trial, go to **Licenses → All products → Microsoft Entra ID P2 → Assign** and assign a licence to your own account. Features don't light up until a licence is actually assigned. **💳**

!!! warning "Set a reminder before you activate a trial"
    A 30-day trial that lapses into a paid subscription because nobody cancelled it is one of the most common ways people get an unexpected Azure bill. If you activate P2, put a calendar reminder at day 25. This is exactly the discipline we practise every day with resource-group cleanup — it applies to licences too.

---

## Part 3 — Users

### What a User Actually Is

A **user object** in Entra ID is a set of attributes — display name, user principal name, job title, department, manager, and so on — plus a set of credentials, plus a set of role assignments. It's the thing that can sign in.

The attribute you must understand is the **User Principal Name (UPN)**. It looks exactly like an email address — `priya@lwmdemo.onmicrosoft.com` — and it is the user's globally unique sign-in name. It may or may not match their actual mailbox address. The domain part **must** be a domain your tenant owns and has verified.

The attribute that matters most for Part Two is the **Object ID** — the user's GUID. When you create a role assignment, Azure does not store "Priya Sharma." It stores that GUID. Which is why, as we'll see in Part 22, deleting a user without cleaning up their role assignments leaves behind ghosts labelled *Identity not found*.

### Two Ways to Classify Every User

**By source:**

- **Cloud-only** — created directly in Entra ID. Entra ID owns the object; you edit it here. Everything we do today is cloud-only.
- **Synced (hybrid)** — the user exists in an on-premises Active Directory and is copied into Entra ID by a sync tool. Critical consequence: **most attributes of a synced user are read-only in the portal.** You edit them on-premises, and the change flows up on the next sync cycle. If you've ever wondered why a name field is greyed out, this is why. (Part 11.)
- **Guest (external)** — a user whose identity lives in a *different* tenant or a personal Microsoft account, invited into yours. (Part 9.)

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

This is one of the most useful facts on this page and a recurring exam question. Somebody leaves the company, you delete their account, three weeks later legal needs their mailbox: it's recoverable. And because the **object ID is preserved**, their Azure role assignments come back working — which they would not if you simply created a new user with the same name.

### Hands-On: Create a User ✅

1. In the Entra admin center: **Entra ID → Users → All users → + New user → Create new user**. **✅**
2. On the **Basics** tab: **✅**
   - **User principal name:** `priya` — then select your `.onmicrosoft.com` domain from the dropdown next to it
   - **Mail nickname:** tick *Derive from user principal name*
   - **Display name:** `Priya Sharma`
   - **Password:** click the eye icon to reveal the auto-generated password and **copy it now** — this screen never comes back. Or untick *Auto-generate password* and set your own.
   - Leave **Account enabled** ticked
3. On the **Properties** tab, fill in a few real attributes — these matter more than they look, because dynamic groups, Conditional Access and (in Part 19) ABAC conditions can all target them: **✅**
   - **Job title:** `Financial Analyst`
   - **Department:** `Finance`
   - **Usage location:** your country — **this one is not optional if you ever assign licences**, and it's the most common cause of "why won't this licence assign" tickets
4. Skip **Assignments** for now — that whole tab is Part Two of this video, and I want you to feel its absence first. **Review + create → Create**. **✅**
5. The user appears in the list within a few seconds. Click into `Priya Sharma` and look at the profile blade: **Object ID** (her unique GUID — **the thing every RBAC assignment will actually reference**), **User principal name**, **Identities** showing *Microsoft Entra ID* as the source, **Account status**. **Copy that Object ID somewhere** — we'll use it in Part 22. **✅**
6. Explore the left menu of the user object — every one of these is a real administrative task you'll do eventually: **✅**
   - **Assigned roles** — Entra roles (Part 12)
   - **Groups** — memberships (Part 4)
   - **Applications** — apps she's been assigned
   - **Licenses** — premium licences
   - **Devices** — devices she's registered
   - **Authentication methods** — her registered MFA methods; as an admin you can **Reset password** and **Require re-register multifactor authentication** here. This is the helpdesk blade.
   - **Monitoring → Sign-in logs** — every sign-in she's attempted, filtered to her
   - **Azure role assignments** — currently empty. This is the exact blade we fill in Part 13.

### Hands-On: Sign In as Priya — and Discover She Can See Nothing ✅

This is the demo that makes the two-planes model click, and it's the hinge the entire video turns on. Do not skip it.

1. Open a **private/incognito window** (or a second browser profile). **✅**
2. Go to `portal.azure.com` and sign in as `priya@yourtenant.onmicrosoft.com` with the password you copied. **✅**
3. You'll be forced to change the password on first sign-in. Set a new one and note it. **✅**
4. If security defaults are enabled on your tenant (very likely — Part 6), you'll be prompted to **register for MFA**. You can complete it with the Authenticator app, or click **Skip for now** if that option appears. **✅**
5. You're in the Azure portal. Now go to **Subscriptions**. **✅**

**The list is empty.**

Priya is a fully valid, fully authenticated user in your tenant. She got a token. Entra ID has confirmed exactly who she is. And she can see **nothing** — not one resource group, not one VM, not the SQL database.

That's the two planes, live. **Authentication succeeded. Authorisation was never granted.** Entra ID said "this is definitely Priya." Azure RBAC said "and Priya has no permissions on anything." Both systems are working perfectly, and the result is a user who can log in and do absolutely nothing.

Hold onto that empty screen. In Part 13 we fix it in about forty seconds, and the contrast is the whole lesson.

6. While you're signed in as Priya, go to **`myaccount.microsoft.com`**. **✅** This is the **My Account** portal — the self-service page every end user in every Entra tenant gets. From here she can manage her own **Security info** (MFA methods), see her **Devices**, view her **Organisations**, and check her **Sign-in activity**. Knowing this URL exists saves your helpdesk enormous time.
7. Leave that private window open. We'll come back to it repeatedly.

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

That's the rule. It's the most important operational habit in this entire video, and it belongs here — in the identity half — even though its payoff is entirely in the authorisation half.

Here's why.

Assign permissions directly to users, and after eighteen months you have four hundred individual assignments scattered across sixty resource groups, nobody knows who has access to what, someone leaves and their access lingers in nine places, and the new starter's onboarding is a two-hour archaeology exercise.

Assign permissions to **groups**, and access becomes a membership question. New analyst joins? Add to `grp-finance-readers`. Leaves? Remove. Their access appears and disappears everywhere, instantly, in one action. Auditing becomes "who is in this group" instead of "let me check sixty resource groups."

And there's a hard technical reason too, which we'll meet properly in Part 22: **a subscription supports a maximum of 4,000 role assignments, and that limit cannot be raised.** Assigning roles to individuals burns through it. Assigning roles to groups means one assignment can serve ten thousand people. Microsoft's own documented remedy for hitting the limit is, literally, "replace principal-based role assignments with group-based role assignments."

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

The power is operational: HR changes someone's department in the HR system, the change syncs to Entra ID, and within minutes their group memberships — **and therefore their Azure permissions** — reshape automatically. Nobody files a ticket. Nobody forgets. That's the full loop: HR data driving cloud authorisation with no human in the middle.

**Dynamic device groups** work the same way over device attributes (`device.deviceOSType -eq "Windows"`), which is how Intune targets policy at fleets.

!!! note "Dynamic groups are not instant"
    Rule evaluation typically takes a few minutes and can take longer in large tenants. Don't design a workflow that assumes membership updates in real time.

### Group Owners and Nesting

- **Owners** can manage membership without being a directory admin. This is proper delegation — the Finance manager owns `grp-finance`, adds and removes their own people, and never needs User Administrator rights.
- **Nesting** — a group can contain another group. **Azure RBAC honours nested groups**, and Microsoft's docs are explicit that role assignments are *transitive* for groups: if you're in group A, and group A is in group B, and group B has a role assignment, you have that role. Useful — but be careful. Some other Azure services do *not* honour nesting. Two levels is plenty; deeper nesting becomes impossible to reason about, and impossible to audit.

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
6. Open the group and note the **Object ID** — **this is what the RBAC assignment in Part 13 will reference.** Explore the left menu: **Members**, **Owners**, **Roles and administrators** (a group can hold an Entra role), **Azure role assignments** (Part 13 fills this in), **Licenses** (group-based licensing, P1). **✅**

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
7. To really land it: edit Priya's department to `Marketing`, wait, and watch her leave the group by herself. Now imagine that group has an Owner role assignment on your production subscription, and you'll understand both the power and the danger. **💳**

---

## Part 5 — Authentication: Passwords, MFA, and the Move to Passkeys

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

## Part 6 — Security Defaults and Conditional Access

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

Read that Phase 2 list again, because it matters for the rest of this course. When we reach **Bicep on Day 25** and **Terraform on Day 26**, those deployments authenticate through ARM, and **that authentication requires MFA**. Read-only operations may not prompt, but anything that creates or modifies will.

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

!!! tip "Conditional Access and RBAC are complementary, not alternatives"
    This confuses people constantly, so let's kill it now. **Conditional Access decides whether you get a token at all.** **RBAC decides what that token lets you do.** A Conditional Access policy cannot say "Priya may restart VMs" — it has no concept of a VM. An RBAC role assignment cannot say "only from a corporate laptop" — it has no concept of a device. Real security uses both: CA gates the sign-in, RBAC gates the action. If an exam question mentions *devices, locations, risk, or MFA requirements*, it's Conditional Access. If it mentions *resources, actions, or scope*, it's RBAC.

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

## Part 7 — Applications: App Registrations and Service Principals

### Applications Need Identities Too

Everything so far has been about humans. But most authentication in a modern system isn't human at all — it's a web app calling an API, a script reading a storage account, a CI/CD pipeline deploying infrastructure at 2am with nobody watching.

Those need identities. In Entra ID, that's **workload identities**, and there are three flavours: app registrations, service principals, and managed identities.

And here's why this section sits in the identity half but pays off in the authorisation half: **a service principal is a security principal.** Everything you learn about RBAC in Part Two applies to applications exactly as it applies to people. Same role assignments, same scopes, same evaluation. Azure genuinely does not care whether the thing holding a token has a face.

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

**Workload identity federation is the modern answer** for CI/CD. When we build pipelines on Days 22–23, this is how a GitHub Actions workflow or an Azure DevOps service connection should authenticate to Azure — no secret in the pipeline, nothing to rotate, nothing to leak. Note that a managed identity can also be used as a federated credential on an app registration (up to 20 per app).

### Delegated vs Application Permissions

When an app requests access to an API (Microsoft Graph, say), there are two fundamentally different kinds of permission:

- **Delegated permissions** — the app acts **on behalf of a signed-in user**. Effective access is the **intersection** of what the app was granted and what the user is allowed. A delegated `Files.Read` lets the app read the files *that user* can read — not everyone's.
- **Application permissions** — the app acts **as itself**, with no user present. An application `Files.Read.All` reads **every file in the tenant**. There's no user to constrain it.

Application permissions always require **admin consent**, and they're the ones to scrutinise in a security review. An over-permissioned application permission is one of the most common serious findings in a real Entra tenant.

!!! note "Graph permissions are not Azure RBAC"
    One more distinction, and it's the same two-planes idea wearing a different hat. **Microsoft Graph permissions** (`User.Read.All`, `Directory.ReadWrite.All`) control access to *directory data* — users, groups, mail, calendars. **Azure RBAC roles** control access to *Azure resources* — VMs, storage, databases. An app can have `Directory.ReadWrite.All` and still not be able to read a single blob. Different API, different permission model, different blade.

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
7. Look at the other two tabs on that blade: **Certificates** and **Federated credentials**. Click **+ Add credential** under Federated credentials and read the scenario dropdown — *GitHub Actions deploying Azure resources*, *Kubernetes accessing Azure resources*, *Managed identity*, *Other issuer*. **Bookmark this screen mentally for Day 22.** **✅**
8. **API permissions**: note `Microsoft Graph → User.Read` was added automatically (delegated). Click **+ Add a permission → Microsoft Graph** and compare the two tiles side by side: **✅**
   - **Delegated permissions** — "your application needs to access the API as the signed-in user"
   - **Application permissions** — "your application runs as a background service or daemon without a signed-in user"

   Select **Application permissions → User.Read.All**, add it, and observe the warning: **Not granted**. Click **Grant admin consent for &lt;tenant&gt;** and watch the status go green. **You just gave a background application the ability to read every user in your directory.** Sit with that for a second — this is how over-privileged apps happen. **✅**
9. **Enterprise applications → All applications** and find `app-lwm-demo`. From here an admin controls **Users and groups** (who can use it), **Single sign-on**, **Provisioning**, **Conditional Access** (yes — CA policies can target applications), and **Permissions**. **✅**

### Hands-On: Sign In as the Application — and Watch It Fail Exactly Like Priya ✅

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
   **The list is empty.** Exactly like Priya in Part 3. **Authenticated, unauthorised.** The app has an identity, full admin-consented Graph permissions to read every user in your directory, and **zero** Azure RBAC. It cannot see one resource group. **✅**

   That contrast is worth pausing on: this application can enumerate every human being in your organisation, and it cannot see an empty resource group. Two planes, two permission systems, completely independent.

4. Return to your own identity:
   ```bash
   az login
   ```
   **✅**

!!! warning "You just handled a real credential"
    That secret string, in your Cloud Shell history, is a password to your tenant. In production it belongs in **Key Vault** (Day 18) — or better, it shouldn't exist at all, which is exactly what the next part is about. Delete this app registration at cleanup.

---

## Part 8 — Managed Identities: The Best Idea in Azure Security

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
6. The target service validates the token and **checks the identity's RBAC permissions**.

No password crosses the wire. No password exists in your code. Token acquisition and refresh are handled by the SDK.

Note step 6 carefully. A managed identity solves *authentication* completely and solves *authorisation* not at all. A brand-new managed identity, like Priya, can do nothing. It needs a role assignment. We give it one in Part 17.

### System-Assigned vs User-Assigned

| | **System-assigned** | **User-assigned** |
|---|---|---|
| **Created** | On the resource, as part of it | As a standalone Azure resource |
| **Lifecycle** | Dies with the resource — delete the VM, the identity is deleted automatically | Independent — you create and delete it yourself |
| **Shared?** | **No** — exactly one resource | **Yes** — attach to many resources |
| **Name** | Always the same as the resource | Whatever you name it |
| **Best for** | A single workload on a single resource | Fleets — a scale set, several apps sharing one permission set, or a blue/green pattern |

Microsoft's current recommendation leans toward **user-assigned** for most service scenarios, and the reason is operational: **permissions survive the resource.** Recycle your VMs weekly and a system-assigned identity means re-granting RBAC every time. With user-assigned, the identity and its role assignments persist while the compute comes and goes.

For a single app talking to a single database, system-assigned is simpler and perfectly correct.

### Hands-On: Create a User-Assigned Managed Identity ✅

1. In the Azure portal, search **Managed Identities** → **+ Create**. **✅**
2. **Resource group:** `rg-day17-demo`, **Region:** your usual, **Name:** `id-lwm-shared`. **Review + create → Create**. **✅**
3. Open it. Note the **Client ID** and **Object (principal) ID** — and note what's *missing*: **there is no credentials blade, no secrets tab, nothing to copy.** That absence is the entire product. **✅**
4. Look at **Azure role assignments** (empty — Part 17 fills it) and **Federated credentials** (the modern CI/CD path from Part 7). **✅**

### Hands-On: Passwordless App Service → SQL Database ✅

The main event of Part One. We take the SQL database from yesterday and connect to it from an App Service **with no password anywhere**.

**Step 1 — Create an App Service** (skip if you still have one from Day 6):

1. Search **App Services → + Create → Web App**. **✅**
2. **Resource group:** `rg-day17-demo`, **Name:** `app-lwm-day17-<yourname>` (globally unique), **Publish:** Code, **Runtime:** .NET 8 (LTS) or Node 20 LTS, **OS:** Linux, **Region:** same region as your SQL server. **✅**
3. **Pricing plan:** **Free F1**. **✅**
4. **Review + create → Create.** **✅**

**Step 2 — Give the app an identity:**

5. Open the App Service → **Settings → Identity**. **✅**
6. On the **System assigned** tab, flip **Status** to **On** → **Save** → **Yes**. **✅**
7. An **Object (principal) ID** appears. Behind the scenes, Entra ID just created a service principal named exactly `app-lwm-day17-<yourname>`. **Copy that principal ID** — Part 17 needs it. **✅**
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

    And notice the second and third lines — `db_datareader`, `db_datawriter`. That's SQL Server's *own* role system doing authorisation, completely separately from Azure RBAC. Even inside a single connection, authentication and authorisation stay two different jobs.

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

!!! tip "The rule to take away"
    **If both ends are Azure services, use a managed identity.** App Service to SQL, Function to Storage, VM to Key Vault, AKS to Container Registry, Data Factory to a data lake — managed identity, every time. Reach for a client secret only when one end genuinely isn't an Azure resource, and even then, check whether workload identity federation covers it first.

---

## Part 9 — External Identities: Guests and B2B

### The Problem

You hire a contractor. An auditor needs read access for six weeks. A partner company's developers need to use your app. Do you create them accounts in your tenant, with your passwords, that you now have to remember to delete?

No. **Entra External ID / B2B collaboration** lets you invite someone using **their own existing identity** — their work account in their own tenant, or a personal Microsoft account. They authenticate **at home**, in their own organisation, under their own organisation's MFA and Conditional Access policies. You just grant access.

The consequences are excellent:

- You never manage their password
- Their home organisation disables their account when they leave — and their access to your resources dies with it, automatically
- Their MFA is enforced by their employer
- You can still apply *your* Conditional Access policies on top
- **And they can hold Azure RBAC role assignments exactly like any member user** — a guest can absolutely be a Reader on your subscription

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

!!! tip "The auditor pattern — remember this for Part 21"
    "An external auditor needs read-only access to production for six weeks" is the canonical scenario that combines everything in this video: **B2B guest invitation** (identity), **Reader role at subscription scope** (authorisation), and **a time-bound PIM eligible assignment that expires automatically** (governance). If you can architect that sentence, you understand Day 17.

---

## Part 10 — Devices

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

## Part 11 — Hybrid Identity (Concepts)

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

!!! note "Synced users get RBAC exactly like cloud users"
    A user synced from on-premises AD is a normal security principal in Entra ID with a normal object ID. You assign Azure roles to them identically. What you *can't* do is manage their name, department or password in the Azure portal — those live on-premises. But their access to your VMs is pure Azure RBAC, configured in the same IAM blade as everyone else.

---

# PART TWO — AUTHORISATION

> **This is the halfway point.** If you're splitting this video across two sittings, stop here. Everything above answered *"who are you?"* Everything below answers *"and what exactly are you allowed to do?"* You have a user who can sign in and see nothing, a group with no permissions, an application with a token and no access, and a managed identity that can reach a database but not a single Azure resource. Four identities, four empty permission sets. Let's fix all four.

---

## Part 12 — Two Role Systems, and the Bridge Between Them

### The Classic Azure Misunderstanding

Before we build anything, we have to clear up the single most misunderstood topic in Azure. There are **two completely separate role systems**, and people mix them up constantly — in interviews, in exams, and in production incidents.

| | **Entra ID roles** (directory roles) | **Azure RBAC roles** |
|---|---|---|
| **Control** | The identity plane | The resource plane |
| **Examples** | Global Administrator, User Administrator, Application Administrator, Billing Administrator | Owner, Contributor, Reader, Storage Blob Data Reader |
| **Let you** | Create users, reset passwords, register apps, configure Conditional Access | Create VMs, read blobs, restart App Services, delete resource groups |
| **Assigned at** | Tenant scope (or an administrative unit) | Management group / subscription / resource group / resource |
| **Managed in** | Entra ID → Roles & admins | Any resource → Access control (IAM) |
| **Custom roles cost** | **P1 licence required** | **Free** |
| **Number of built-in roles** | ~100+ | 900+ and growing weekly |

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

    There's a second, legitimate use worth knowing: if you transfer a subscription between tenants and lose access to it (remember — **all role assignments are permanently deleted** on transfer), this toggle is how you get back in.

### Hands-On: Explore Both Role Systems Side by Side ✅

1. **Entra ID → Roles & admins → All roles**. Scan the list — there are over a hundred built-in Entra roles. Read a few names: *User Administrator*, *Application Administrator*, *Authentication Administrator*, *Conditional Access Administrator*, *Global Reader*. **✅**
2. Click **Global Administrator → Assignments** and confirm your own account is listed. That's how you've been able to do everything for sixteen days. **✅**
3. Click **Global Reader**. Read the description: everything Global Admin can *see*, nothing it can *change*. **This is the single most underused role in Azure.** Auditors, analysts and new team members should get Global Reader, not Global Administrator. **✅**
4. Assign an Entra role: open **User Administrator → + Add assignments** → select `Priya Sharma` → **Add**. **✅**
5. **Now go prove the split.** In your private window, refresh Priya's session and go to **Entra ID → Users**. She can see the user list and create users — she's a User Administrator. Now go to **Subscriptions**. **✅**

   **Still empty.** She can create a hundred users and cannot see one resource group. That's the two planes, demonstrated in a single browser session, and it's the clearest thirty seconds in this video.

6. Back in your admin session, open **Subscriptions → your subscription → Access control (IAM) → Role assignments**. Completely different blade, completely different role list, completely different service. Look at the **Roles** tab and note the count — it's in the hundreds. **✅**
7. Go to **Entra ID → Overview → Properties**. Find **Access management for Azure resources**. Read the description carefully. **Leave it set to No.** **✅**

!!! tip "The exam giveaway"
    If a question mentions creating users, resetting passwords, registering applications, or configuring Conditional Access → **Entra role**. If it mentions creating, reading, modifying or deleting Azure resources → **Azure RBAC role**. If it says "manage users *and* virtual machines," the answer is **two separate assignments**.

---

## Part 13 — Azure RBAC: The Three-Part Sentence

### The Whole Model, In One Line

Azure RBAC is the authorisation system built into Azure Resource Manager. Every access decision it makes comes from one construct: the **role assignment**. And a role assignment is always, without exception, three things joined together:

> **WHO** can do **WHAT**, **WHERE**.

- **WHO** — the **security principal**: a user, a group, a service principal, or a managed identity. Four types. That's the complete list. Notice that every one of those four is something you created in Part One.
- **WHAT** — the **role definition**: a named collection of permissions. *Reader*, *Contributor*, *Virtual Machine Contributor*, *Storage Blob Data Reader*, or one you write yourself.
- **WHERE** — the **scope**: the level at which those permissions apply. Management group, subscription, resource group, or an individual resource.

Bolt those three together and you have granted access. Delete the assignment and you have revoked it. There is no other mechanism. Every permission in Azure — yours included — exists because somewhere there is a row saying *this principal, this role, this scope*.

Say it out loud a few times, because it's the answer to roughly a third of the RBAC questions on any Azure exam: **security principal, role definition, scope.**

### Scope: The Four Levels

Scope is a hierarchy, and permissions **inherit downwards**:

```text
Root management group
   └── Management group  (e.g. "Production")
          └── Subscription  (e.g. "LWM-Prod")
                 └── Resource group  (e.g. "rg-day17-demo")
                        └── Resource  (e.g. one storage account)
```

Assign a role at any level and it applies to that level **and everything beneath it**. Nothing flows upward.

- **Reader at management group scope** → the user can read everything in every subscription under that management group.
- **Contributor at subscription scope** → full management of every resource in that subscription.
- **Contributor at resource group scope** → full management inside that one resource group, invisible everywhere else.
- **Storage Blob Data Reader on one storage account** → exactly that one storage account.

**Management groups** deserve a sentence, since we haven't met them properly. A management group is a container for subscriptions, and management groups can nest inside each other (up to six levels). Every tenant has a **root management group** at the top that contains everything. They exist for exactly this: applying one role assignment or one policy across many subscriptions at once, instead of repeating yourself per subscription.

### The Golden Rule

> **Assign the least privileged role, at the narrowest scope, to a group rather than a person.**

That single sentence is the answer to more exam questions than any other in this video, and it's also just correct engineering. Every time you widen a scope for convenience, you widen the blast radius of a compromised account.

Microsoft adds a hard number to it: **you should have a maximum of three subscription Owners.** Defender for Cloud will actively flag you if you have more.

### RBAC Is Additive — And Deny Assignments Are the Exception

Your effective permissions are the **union** of every role assignment that applies to you, from every scope, including ones you inherit through group membership (and through nested groups — remember, assignments are transitive).

There is no "deny" in a role assignment. If you have Reader at subscription scope and Contributor on one resource group, you are a Contributor on that resource group — the Reader assignment doesn't restrain you. Adding a *more restrictive* role never takes anything away.

The one thing that *can* take something away is a **deny assignment**, which we'll cover in Part 20. Deny assignments beat everything — but you can't create them by hand.

### How Azure Actually Evaluates a Request

Worth knowing precisely, because it explains every "why can't I do this?" moment you'll ever have. When you click a button in the portal, this happens:

1. You acquire a token for Azure Resource Manager. **The token contains your group memberships**, including transitive ones.
2. Your client calls the ARM REST API with that token attached.
3. ARM gathers **all role assignments and deny assignments** that apply to the target resource — at that scope and every scope above it.
4. **If a deny assignment matches, access is blocked. Full stop.** Nothing overrides it.
5. ARM narrows to the assignments matching you or your groups, producing your effective role set.
6. ARM checks whether the requested action is permitted:
   - `Actions − NotActions = effective management permissions`
   - `DataActions − NotDataActions = effective data permissions`
7. No matching action → **denied**.
8. If the matching assignment carries a **condition** (Part 19), the condition is evaluated.
9. Condition satisfied → **allowed**. Otherwise denied.

Step 1 is quietly important: because group memberships are baked into the token, **adding someone to a group doesn't take effect until they get a new token.** That's a chunk of the "I granted access and it didn't work" complaints, and the fix is usually to sign out and back in.

### Hands-On: The Payoff — Give Priya Access ✅

Priya has been staring at an empty portal for an hour. Let's fix it, and watch the exact moment authorisation arrives.

1. In your **admin** window, go to **Resource groups → `rg-day17-demo`**. **✅**
2. Click **Access control (IAM)** in the left menu. Spend a moment here — this blade is where you will spend the rest of your Azure career: **✅**
   - **Check access** — "what can this specific person do here?"
   - **Role assignments** — everything granted at this scope, plus everything inherited from above
   - **Roles** — the full catalogue of role definitions available for assignment
   - **Deny assignments** — almost certainly empty (Part 20)
   - **Classic administrators** — gone. Classic administrator roles were **fully retired in May 2026**; if you see this tab in an old tutorial, that tutorial predates the retirement.
3. Look at the **Role assignments** tab as it stands. You'll see your own account as **Owner**, with scope shown as **This resource (Inherited)** — inherited from the subscription. **That word "Inherited" is the hierarchy made visible.** **✅**
4. Click **+ Add → Add role assignment**. Walk through the wizard tab by tab: **✅**
   - **Role tab** — search `Reader`. Note the two categories at the top: **Job function roles** and **Privileged administrator roles** (Part 15). Reader is a job function role. Select it and click **Next**.
   - **Members tab** — **Assign access to:** *User, group, or service principal*. Click **+ Select members** and choose **`grp-finance-team`** — **the group, not Priya**. This is the golden rule in action, on camera. Click **Next**.
   - **Conditions tab** — leave alone for now; Part 19.
   - **Review + assign** — read the summary out loud: *Reader · grp-finance-team · rg-day17-demo*. Who, what, where. Click **Review + assign**.
5. Now flip to the **private window** where Priya is signed in. **Sign her out and back in** (remember step 1 of the evaluation flow — she needs a fresh token with her group membership and the new assignment). **✅**
6. Go to **Resource groups**. **✅**

**`rg-day17-demo` is there.**

She can open it. She can see the App Service, the managed identity, every resource inside. She can click into the App Service and read its configuration. That empty portal from Part 3 is now populated — and it happened because of exactly one row in a table somewhere: *principal + role + scope*.

7. Now show her the limits. As Priya: **✅**
   - Try to **stop** the App Service — the button is greyed out or errors. Reader reads.
   - Go to **Resource groups** again and confirm she sees **only** `rg-day17-demo`. Your other resource groups are invisible. That's scope.
   - Try **Subscriptions** — she can now see the subscription name (she needs to, to navigate to the RG) but has no rights on anything else in it.
8. Back in the admin window, prove the inheritance model. Open a **resource inside** the RG — say the App Service — and go to its **Access control (IAM) → Role assignments**. The Reader assignment appears here too, marked **(Inherited)** with the scope pointing at the resource group. **You never assigned anything on the App Service. It flowed down.** **✅**
9. Use **Check access**: on the App Service IAM blade, click **Check access**, search `Priya Sharma`, and select her. Azure shows exactly what she has and where it came from. **This is the blade to open when someone says "I can't access X" — it answers the question in five seconds.** **✅**

!!! tip "Say the sentence every time"
    When you assign a role, narrate it: *"Reader, to grp-finance-team, at rg-day17-demo."* Who, what, where. Do that consistently and you will never write a sloppy role assignment, because saying "Owner, to Priya, at subscription" out loud makes the problem audible.

---

## Part 14 — Role Definitions Under the Hood

### A Role Is Just a List of Strings

A role definition looks impressive in the portal. Underneath, it's JSON with five interesting properties:

```json
{
  "Name": "Virtual Machine Operator",
  "Id": "88888888-8888-8888-8888-888888888888",
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Network/*/read",
    "Microsoft.Storage/*/read",
    "Microsoft.Authorization/*/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Support/*"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId}"
  ]
}
```

That's a complete, working Azure role. Every built-in role is the same shape.

### The Five Properties That Matter

**`Actions`** — control-plane operations this role permits. The format is always `{ResourceProvider}/{resourceType}/{operation}`:

```text
Microsoft.Compute/virtualMachines/read          ← view a VM
Microsoft.Compute/virtualMachines/write         ← create or modify a VM
Microsoft.Compute/virtualMachines/delete        ← delete a VM
Microsoft.Compute/virtualMachines/start/action  ← start a VM
Microsoft.Storage/storageAccounts/listKeys/action  ← read the account keys
```

Wildcards expand: `Microsoft.Compute/*/read` means "read anything in the Compute provider."

**`NotActions`** — a *subtraction*, not a deny. This is the most misread property in Azure. `NotActions` removes items from the `Actions` list; it does not block anything granted by a *different* role assignment. If Contributor has `Microsoft.Authorization/*/Delete` in NotActions but you also hold Owner somewhere above, you can still delete role assignments. **NotActions is arithmetic. Deny assignments are enforcement.** Don't confuse them.

This is exactly how **Contributor** is built: `Actions: ["*"]` — everything — minus a short `NotActions` list that strips out the authorisation write operations. That's the entire difference between Contributor and Owner. Contributor can do anything except grant access.

**`DataActions`** and **`NotDataActions`** — the same idea for *data-plane* operations. `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read` is a DataAction: it's about the bytes inside the container, not the container's configuration. This split is the subject of Part 16, and it's the most surprising thing in Azure RBAC.

**`AssignableScopes`** — where this role definition is *available to be assigned*. Built-in roles use `/` (the root), so they're assignable everywhere. Custom roles must list specific management groups, subscriptions, or resource groups.

### The Arithmetic

Your effective permissions are, precisely:

```text
Actions      −  NotActions      =  effective management permissions
DataActions  −  NotDataActions  =  effective data permissions
```

That's it. That's the whole calculation ARM performs at step 6 of the evaluation flow.

### Hands-On: Read a Built-in Role ✅

1. **Subscription → Access control (IAM) → Roles** tab. You're looking at the full catalogue. Use the **Type** filter to switch between *BuiltInRole* and *CustomRole*. **✅**
2. Find **Contributor**, click the ellipsis (**…**) → **View**. Then open the **JSON** tab. **✅**
3. Read the `NotActions` array out loud. You'll see entries like: **✅**
   ```text
   Microsoft.Authorization/*/Delete
   Microsoft.Authorization/*/Write
   Microsoft.Authorization/elevateAccess/Action
   Microsoft.Blueprint/blueprintAssignments/write
   Microsoft.Blueprint/blueprintAssignments/delete
   ```
   `Actions` is `["*"]`. Everything. And then those five-ish subtractions. **Contributor is literally "Owner minus the ability to grant access."** Once you've seen the JSON you will never forget the difference again.
4. Now open **Reader** → JSON. `Actions: ["*/read"]`. One line. **✅**
5. Now open **Storage Blob Data Reader** → JSON, and look at where the permissions live: **✅**
   ```json
   "actions": [
     "Microsoft.Storage/storageAccounts/blobServices/containers/read",
     "Microsoft.Storage/storageAccounts/blobServices/generateUserDelegationKey/action"
   ],
   "dataActions": [
     "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
   ]
   ```
   **The interesting permission is in `dataActions`, not `actions`.** Remember that shape — Part 16 is built on it.
6. Use the search box on the Roles tab to explore how granular this gets: search `Key Vault` (a dozen roles), `Kubernetes` (many), `SQL`, `Cosmos`. There are **900+ built-in roles** now and Microsoft adds them constantly. Microsoft's official line is "over 100," which was true several years ago — the real number is far higher. **The practical consequence: before you write a custom role, search the built-in list properly. It's probably already there.** **✅**

!!! tip "Assign by role ID, not role name, in automation"
    Role names can change — a preview role drops "(Preview)" when it ships, or you rename a custom role. **The role ID never changes.** Microsoft's explicit best practice is to reference the GUID in scripts, Bicep and Terraform. You'll thank yourself on Day 25.

---

## Part 15 — Built-in Roles: Job Function vs Privileged Administrator

### Microsoft's Own Two Buckets

When you open the Role tab in the assignment wizard, Azure sorts roles into two categories. This isn't cosmetic — it's a security classification, and it should change your behaviour.

**Job function roles** manage specific Azure resources. *Virtual Machine Contributor*, *Storage Blob Data Reader*, *Key Vault Secrets User*, *AcrPull*, *Website Contributor*. There are 900+ of these. They're narrow, they're boring, and they are what you should be assigning almost all the time.

**Privileged administrator roles** grant sweeping resource access or — crucially — **the ability to hand out permissions to other people**. There are exactly five, and they apply to all resource types:

| Role | What it grants |
|---|---|
| **Owner** | Full access to manage all resources **+ assign roles in Azure RBAC** |
| **Contributor** | Full access to manage all resources, **cannot** assign roles |
| **Role Based Access Control Administrator** | Manage user access to Azure resources — assign roles, **including Owner**. Cannot manage access by other means (e.g. Azure Policy). |
| **User Access Administrator** | Manage user access to Azure resources — assign roles, **including Owner** |
| **Reservations Administrator** | Manage all reservations in a tenant + assign roles for reservations |

Look at what puts a role in that list. It isn't "can do a lot" — Contributor can delete your entire production estate and it's on the list mainly for its breadth. **It's "can grant permissions."** The ability to create role assignments is the master key, because anyone who holds it can give themselves anything, including Owner.

### The Big Four, Properly Distinguished

| Role | Read resources | Create/modify/delete | Assign roles |
|---|---|---|---|
| **Reader** | ✅ | ❌ | ❌ |
| **Contributor** | ✅ | ✅ | ❌ |
| **Owner** | ✅ | ✅ | ✅ |
| **User Access Administrator** | ✅ | ❌ | ✅ |

That last row surprises people. User Access Administrator can hand out Owner to anybody, and cannot create a VM. It's a pure access-management role.

### Prefer RBAC Administrator Over User Access Administrator

Here's a genuinely useful, genuinely current recommendation that most tutorials haven't caught up with.

For years, "let someone else manage access" meant assigning **User Access Administrator**. Microsoft now ships a better role for that job: **Role Based Access Control Administrator**. It has strictly fewer permissions:

- Create a role assignment at the specified scope
- Delete a role assignment at the specified scope
- Read resources of all types, except secrets
- Create and update a support ticket

That's all. No ability to manage access through other mechanisms like Azure Policy. It's the same job with a smaller blast radius, and it's the role Microsoft's own troubleshooting docs now list first as the prerequisite for adding role assignments.

And in Part 19 we go one better and **constrain it with a condition**, so the delegate can only assign *specific roles* to *specific principals* — turning "you can grant anything to anyone here" into "you can grant Reader to the finance group, and nothing else."

### Microsoft's Rules for Privileged Roles

Straight from the best-practices documentation, and all four are worth following:

1. **Remove unnecessary privileged role assignments.** Audit them regularly.
2. **Prefer a job function role** whenever one exists. Need someone to restart VMs? *Virtual Machine Contributor*, not Contributor.
3. **If you must assign a privileged role, use a narrow scope** — resource group or resource, never management group or subscription if you can avoid it.
4. **If the role can create role assignments, add a condition** to constrain it.

Plus the number: **maximum three subscription Owners.**

### Hands-On: Explore and Right-Size ✅

1. **Subscription → Access control (IAM) → Role assignments**. Look at the **Privileged** filter / the privileged administrator role assignments view. On a lab subscription you'll see exactly one: you, as Owner. In a real tenant this list is the first thing a security auditor asks for. **✅**
2. Click **+ Add → Add role assignment**, go to the **Role** tab, and switch between the **Job function roles** and **Privileged administrator roles** tabs. Read the warning Azure shows on the privileged tab. **✅** Then **Cancel** — we're not assigning at subscription scope today.
3. Practise right-sizing. Priya (via `grp-finance-team`) has Reader on `rg-day17-demo`. Suppose she now needs to restart the App Service. **The lazy answer is Contributor. The correct answer is a job function role.** **✅**
   - Go to the **App Service → Access control (IAM) → + Add → Add role assignment**
   - Search the role list for `Website Contributor` — it manages web apps but not the App Service plan or other resource types
   - Assign it to `grp-finance-team` **at the App Service resource scope**, not the resource group
   - **Review + assign**
4. In the private window, have Priya refresh. She can now **Stop** and **Start** the App Service — and she still can't touch the managed identity or anything else in the resource group. **Narrowest role, narrowest scope.** **✅**
5. Look at the App Service **IAM → Role assignments** now. Two rows: *Reader (Inherited, from resource group)* and *Website Contributor (This resource)*. **Additive.** Her effective permission is the union. **✅**

---

## Part 16 — Control Plane vs Data Plane: The Demo That Surprises Everyone

### The Setup

This is my favourite five minutes in the whole video, because almost nobody predicts the result correctly the first time.

Here's the question: **if I make you a Contributor on a storage account — full management rights, you can delete the entire account — can you read a file inside it?**

Everybody says yes. The answer is **no**.

### The Two Planes, Again — But Inside a Single Resource

Azure splits operations into two categories:

- **Control plane** — operations on the *resource itself*. Create the storage account, change its replication setting, add a private endpoint, enable versioning, delete it. These go through Azure Resource Manager at `management.azure.com`. They're governed by **`Actions`**.
- **Data plane** — operations on the *data inside* the resource. Read a blob, write a message to a queue, get a secret from Key Vault, query a Cosmos container. These go straight to the service's own endpoint — `yourstorage.blob.core.windows.net` — never touching ARM. They're governed by **`DataActions`**.

**Contributor's `Actions` is `["*"]` and its `DataActions` is empty.** So a Contributor can do every management operation on a storage account and has literally zero data permissions. They can delete the account containing the file. They cannot open the file.

This is not a quirk. It's deliberate, it's the foundation of separation of duties in Azure, and it's why there are separate roles like *Storage Blob Data Reader*, *Key Vault Secrets User*, and *Cosmos DB Built-in Data Reader*.

!!! note "The escape hatch, and why it matters"
    A Contributor *can* call `Microsoft.Storage/storageAccounts/listKeys/action` — that's a control-plane action — and use the returned account key to access data anyway. That's why **`Storage Account Contributor` is more dangerous than it looks**, and why serious environments disable shared key access entirely (`allowSharedKeyAccess = false`) and force all data access through Entra ID and RBAC. Worth calling out on camera: RBAC data-plane roles only give you real separation of duties if the key-based back door is closed.

### Hands-On: Prove It ✅

**Step 1 — Create a storage account with a file in it (as admin):**

1. **Storage accounts → + Create**. **Resource group:** `rg-day17-demo`, **Name:** `stlwmday17<yourname>` (lowercase, globally unique), **Redundancy:** LRS. **Review + create → Create**. **✅**
2. Open it → **Data storage → Containers → + Container** → name `reports` → **Create**. **✅**
3. Open `reports` → **Upload** → upload any small text file. Name it `q3-results.txt`. **✅**

**Step 2 — Make Priya a Contributor on it:**

4. On the **storage account → Access control (IAM) → + Add → Add role assignment**. **✅**
5. **Role:** `Contributor`. **Members:** `grp-finance-team`. **Review + assign**. **✅**

**Step 3 — Watch it fail:**

6. In the private window, sign Priya out and back in (fresh token). Navigate to the storage account. **✅**
7. Confirm she has real power: she can see **Configuration**, change the **access tier**, open **Data protection**, and the **Delete** button on the account is live. She could destroy this storage account right now. **✅**
8. Now click **Containers → reports**. **✅**

**She gets an error.** Something along the lines of *"You do not have permissions to list the data using your user account with Microsoft Entra ID"* — often with an offer to switch to access key authentication instead.

Sit in that for a second on camera. **She can delete the container. She cannot list what's in it.** Control plane, data plane.

**Step 4 — Fix it properly:**

9. Back as admin: **storage account → Access control (IAM) → + Add → Add role assignment**. **✅**
10. **Role:** search `Storage Blob Data Reader`. Open its **JSON** first and show the `dataActions` array — this is the role that actually contains `blobs/read`. **✅**
11. **Members:** `grp-finance-team`. **Review + assign**. **✅**
12. Private window: refresh (this one usually works without re-authenticating, but give it a minute — role assignment changes can take **up to 10 minutes** to propagate). Click into `reports`. **✅**

**The blob list appears.** She can open `q3-results.txt` and read it. She still cannot upload or delete a blob — that would need *Storage Blob Data Contributor*.

13. To complete the picture, in the private window click **Switch to access key** in the container view. She *can* do that, because Contributor grants `listKeys`. **That's the back door** — and it's exactly why production storage accounts should have shared key access disabled. **✅**

!!! tip "The exam and interview version"
    *"A user has the Contributor role on a storage account but reports they cannot read blobs. What's the minimum change?"* → **Assign a data-plane role such as Storage Blob Data Reader.** Not Owner. Not Storage Account Contributor. The whole question is testing whether you know the two planes exist. The same pattern repeats across Azure: **Key Vault Secrets User** vs Contributor on the vault (Day 18 — you'll meet this again), **Cosmos DB Built-in Data Reader** vs Cosmos DB Account Reader, **AcrPull** vs Contributor on a container registry.

---

## Part 17 — RBAC for Workload Identities

### Same Model, No Human

Everything you just did for Priya applies identically to applications. The managed identity you created in Part 8 is a **security principal**. It can hold role assignments. Azure does not care that it has no face.

This is the moment the two halves of the video fuse: in Part 8 you gave an App Service an identity with no permissions. Now we give that identity a permission, and the loop closes — a piece of software authenticates with no credential and is authorised with no credential.

### Hands-On: Give the Managed Identity a Role ✅

1. Go to the **storage account** `stlwmday17<yourname>` → **Access control (IAM) → + Add → Add role assignment**. **✅**
2. **Role:** `Storage Blob Data Reader`. **✅**
3. On the **Members** tab, change **Assign access to** from *User, group, or service principal* to **Managed identity**. **✅**
4. Click **+ Select members**, set **Managed identity** to *App Service*, and pick `app-lwm-day17-<yourname>`. **✅**

   Note what you did *not* do: you did not paste a client ID, and there was no secret involved anywhere. The portal enumerated the managed identities in your subscription and you clicked one.

5. **Review + assign**. **✅**
6. Look at the storage account's **Role assignments** tab now. You have three rows: a group with Contributor, the same group with Storage Blob Data Reader, and an **App Service** with Storage Blob Data Reader. **Humans and software in the same list, in the same system, with the same model.** **✅**

### Hands-On: Watch a VM Read a File With No Credential ✅

This is the best demo of the day, and it needs a Linux VM. If you still have one from Day 3, use it. If not, watch — or spin up a B1s VM in `rg-day17-demo`.

1. Open the VM → **Security → Identity → System assigned** → **Status: On** → **Save** → **Yes**. **✅**
2. Go to the **storage account → Access control (IAM) → + Add → Add role assignment → Storage Blob Data Reader → Members → Managed identity → Virtual machine → [your VM] → Review + assign**. **✅**
3. SSH into the VM. First, prove the identity exists by asking the Instance Metadata Service for a token:
   ```bash
   curl -s -H "Metadata: true" \
     "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"
   ```
   You get JSON containing an `access_token` — a JWT, valid roughly 24 hours, issued by Entra ID to this VM's identity. **✅**
4. Paste it into **`jwt.ms`** and read the claims: `aud` (audience), `iss` (issuer — your tenant), `oid` (**the identity's object ID — this is the GUID your role assignment references**), `exp` (expiry). **✅**
5. Now use it for real. On the VM:
   ```bash
   az login --identity
   az storage blob list \
     --account-name stlwmday17<yourname> \
     --container-name reports \
     --auth-mode login \
     --output table
   ```
   **✅**

**The file is listed.** Read that command back: there is no username, no password, no connection string, no account key, no secret of any kind. `--identity` means "use whatever identity this machine has." `--auth-mode login` means "authorise with Entra ID and RBAC, not with an account key."

6. Download it, just to be sure it's real:
   ```bash
   az storage blob download \
     --account-name stlwmday17<yourname> \
     --container-name reports \
     --name q3-results.txt \
     --auth-mode login \
     --file ./downloaded.txt
   cat ./downloaded.txt
   ```
   **✅**
7. Now break it, because seeing the failure is as instructive as seeing the success. Back in the portal, **remove** the Storage Blob Data Reader assignment from the VM's identity. Wait a few minutes, then re-run the list command on the VM. **✅**

   `AuthorizationPermissionMismatch`. The VM still authenticates perfectly — it still gets a token — and it is now unauthorised. **Authentication and authorisation, decoupled, demonstrated on a machine.**

8. Re-add the role assignment if you want to keep playing. **✅**

!!! warning "Managed identity permission changes are slower than user ones"
    Two caching gotchas Microsoft documents explicitly, and both have burned people:

    - Role assignment changes can take **up to 10 minutes** to take effect generally.
    - If you add a **managed identity to a group** and assign the role to the group, the managed identity back-end caches group membership for **up to 24 hours**. If you need a managed identity's permissions to change quickly, **assign the role directly to the identity, not via a group.**

    This is the one place where "always assign to groups" has a genuine exception. Know it, and know why.

---

## Part 18 — Custom Roles

### When Built-in Isn't Right

With 900+ built-in roles, your first move should always be to search the catalogue. But sometimes nothing fits:

- The built-in role is too broad — *Virtual Machine Contributor* lets someone resize and delete VMs when you only want them to restart one that's hung.
- The built-in role is too narrow — you need permissions spanning two services and there's no single role covering both.
- You want to reduce your role assignment count by combining several built-in roles into one (a real, documented remedy for hitting the 4,000 limit).

Custom roles are **free**, they can be assigned to any principal type, and they behave identically to built-in roles once created. (Reminder: custom **Entra** roles need P1. Custom **Azure RBAC** roles are free. Different systems.)

### Three Ways to Build One

The portal gives you three starting points:

1. **Clone a role** — start from an existing built-in role and edit. Usually the right choice.
2. **Start from scratch** — empty definition, add permissions by hand.
3. **Start from JSON** — upload a definition file. This is how you keep roles in source control, which is the grown-up answer.

### The Limits — Know These

| Limit | Value |
|---|---|
| Custom roles per tenant | **5,000** |
| `AssignableScopes` entries per role | 2,000 |
| Management groups in `AssignableScopes` | **Exactly one** |
| Root scope (`/`) in `AssignableScopes` | **Not allowed** |
| Wildcards in `AssignableScopes` | **Not allowed** |
| Wildcards per action string | One |
| Custom roles with `DataActions` at management group scope | **Cannot be assigned there** |

A few of those deserve explanation. You can't use `/` or wildcards in `AssignableScopes` because that would let someone widen their own reach by editing a role definition. And custom roles with `DataActions` can be *defined* with a management group in scope but only *assigned* at subscriptions inside it — a useful trick if you need the same data role across many subscriptions without duplicating the definition.

Also: **you cannot delete a custom role while any role assignment still references it.** You'll get `RoleDefinitionHasAssignments`. Find and remove the assignments first — and remember to check PIM for *eligible* assignments too, which don't show up in the normal list.

### Hands-On: Build "VM Operator" ✅

The scenario: your NOC team needs to restart misbehaving VMs at 3am. They must **not** be able to create, resize, or delete VMs. No built-in role does exactly this — *Virtual Machine Contributor* is far too broad.

1. Go to the **subscription → Access control (IAM) → + Add → Add custom role**. **✅**

   (You can also start from a resource group, which sets a narrower default `AssignableScopes`.)

2. **Basics tab:** **✅**
   - **Custom role name:** `LWM VM Operator`
   - **Description:** `Can view, start, restart and stop virtual machines. Cannot create, resize or delete.`
   - **Baseline permissions:** *Clone a role* → select **Virtual Machine Contributor**
3. **Permissions tab.** You're looking at everything Virtual Machine Contributor can do. Now cut it down: **✅**
   - Remove `Microsoft.Compute/virtualMachines/write` (create/modify)
   - Remove `Microsoft.Compute/virtualMachines/delete`
   - Remove the disk, snapshot and image write/delete permissions
   - Keep the read permissions, and confirm these three survive:
     ```text
     Microsoft.Compute/virtualMachines/start/action
     Microsoft.Compute/virtualMachines/restart/action
     Microsoft.Compute/virtualMachines/powerOff/action
     ```
   - Click **+ Add permissions**, search `Microsoft.Insights`, and add `Microsoft.Insights/metrics/read` so they can see whether the restart helped
4. **Assignable scopes tab:** by default it lists your subscription. **Narrow it** — remove the subscription and **+ Add assignable scopes → `rg-day17-demo`**. Now this role can only ever be assigned inside that resource group. **✅**
5. **JSON tab.** This is the payoff for Part 14 — read the definition you just built. You can edit it directly here. Copy it into a file if you want it in source control. **✅**
   ```json
   {
     "properties": {
       "roleName": "LWM VM Operator",
       "description": "Can view, start, restart and stop virtual machines.",
       "assignableScopes": [
         "/subscriptions/<sub-id>/resourceGroups/rg-day17-demo"
       ],
       "permissions": [
         {
           "actions": [
             "Microsoft.Compute/virtualMachines/read",
             "Microsoft.Compute/virtualMachines/start/action",
             "Microsoft.Compute/virtualMachines/restart/action",
             "Microsoft.Compute/virtualMachines/powerOff/action",
             "Microsoft.Compute/virtualMachines/instanceView/read",
             "Microsoft.Insights/metrics/read",
             "Microsoft.Resources/subscriptions/resourceGroups/read"
           ],
           "notActions": [],
           "dataActions": [],
           "notDataActions": []
         }
       ]
     }
   }
   ```
6. **Review + create → Create.** Custom role creation takes a few minutes to propagate. **✅**
7. Assign it: **`rg-day17-demo` → Access control (IAM) → + Add → Add role assignment**. On the **Role** tab, use the **Type** filter → **CustomRole** to find `LWM VM Operator`. Assign it to `grp-finance-team`. **✅**
8. Test as Priya (VM required): she can **Restart** and **Stop** the VM. **Resize** is greyed out. **Delete** fails. **✅**

!!! warning "Don't reach for the wildcard"
    It's tempting to write `Microsoft.Compute/*` and be done. Microsoft's explicit guidance is to avoid wildcards in custom roles and list actions explicitly. The reason is future-proofing: a wildcard silently grants every permission Microsoft adds to that provider **later**, including ones that didn't exist when you wrote the role. That's how a "restart VMs" role quietly becomes something else eighteen months from now.

---

## Part 19 — Conditions (ABAC) and Constrained Delegation

### When Roles and Scopes Aren't Enough

RBAC answers *who, what, where*. **ABAC — attribute-based access control — adds *under what circumstances*.**

An **ABAC condition** is an optional extra check bolted onto a role assignment. It can only *narrow* what the role grants; it can never expand it, and it can never explicitly deny a specific resource. The syntax is an expression over attributes:

```text
<attribute> <operator> <value>
```

Three reasons Microsoft gives for using conditions:

1. **Finer-grained access** — "read blobs, but only ones tagged `Project=Blue`."
2. **Fewer role assignments** — one conditional assignment can replace hundreds, which directly helps with the 4,000-per-subscription limit.
3. **Business-meaningful attributes** — project names, classification levels, development stages.

### Where Conditions Work Today

This is the part to be precise about, because it's a common source of wrong answers:

- Conditions can currently be added to role assignments with **blob storage or queue storage data actions**. That's the supported surface — it is not yet universal across Azure.
- The **portal condition editor** is GA (since October 2022), as are PowerShell, CLI and REST.
- **Environment attributes** (private link, subnet, UTC time) went GA April 2024.
- **Custom security attributes on the principal** went GA November 2023 — that's the "user's `Project` attribute must match the blob's `Project` tag" pattern, which is genuinely powerful.
- The **visual editor supports 5 expressions**; go beyond that in the code editor.

Blob attributes you can build conditions on: account name, blob index tags, blob path, blob prefix, container name, encryption scope, is-current-version, is-private-link, snapshot, UTC now, version ID.

Example scenarios straight from the docs:

- Read access to blobs with the tag `Project=Cascade`
- Read, write or delete blobs only in containers named `blobs-example-container`
- Read access to blobs with a path of `logs`
- Read access to blobs **only during a specific date/time range**
- Write access to blobs **only over a private link or from a specific subnet**

### Hands-On: A Conditional Data Role ✅

Let's tighten Priya's blob access so she can only read from the `reports` container — not any other container we add later.

1. **Storage account → Access control (IAM) → Role assignments**. Find the **Storage Blob Data Reader** assignment for `grp-finance-team`. **✅**
2. Click the ellipsis (**…**) → **Edit** → go to the **Conditions** tab → **+ Add condition**. **✅**
3. You get two paths. Look at both: **✅**
   - **Condition templates** — pre-built scenarios Microsoft ships. Much easier, and correct by construction.
   - **Advanced condition editor** — full control.
4. Use the editor. Click **+ Add action** and select **Read a blob**. Then **+ Add expression**: **✅**
   - **Attribute source:** *Resource*
   - **Attribute:** `Container name`
   - **Operator:** `StringEquals`
   - **Value:** `reports`
5. Switch to the **code editor** tab and read what you built: **✅**
   ```text
   (
     (
       !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'})
     )
     OR
     (
       @Resource[Microsoft.Storage/storageAccounts/blobServices/containers:name]
         StringEquals 'reports'
     )
   )
   ```
   Read the shape aloud: *"either this isn't a blob-read action — in which case the condition doesn't apply — or the container is named reports."* That `!(ActionMatches...) OR (...)` pattern is how every condition is structured, and once you see it you can read any of them.
6. **Save**. **✅**
7. Test: as admin, create a second container called `payroll` and upload a file. As Priya, she can still read `reports` and **cannot list `payroll`** — despite holding Storage Blob Data Reader on the whole account. **✅**

### Constrained Delegation — The Best Feature Nobody Uses

Now the other half of conditions, and it's the more important one.

**The problem:** you want your team lead to hand out access without bothering you. Traditionally that means assigning them **User Access Administrator**, which lets them assign **any role to any principal** in that scope — including giving themselves Owner. Microsoft's own docs list the issues bluntly: unrestricted access at the scope, can assign any role to anyone including themselves, and can make *other* people access administrators too.

**The fix:** assign **Role Based Access Control Administrator** and attach a condition that constrains it. You can constrain:

- **Which roles** they may assign or remove
- **Which principal types** (user / group / service principal) they may assign to
- **Which specific principals** they may assign to
- **Different rules for adding vs removing** assignments

The documented example: Alice gives Dara the RBAC Administrator role with a condition allowing only *Backup Contributor* and *Backup Reader*, only to the *Marketing* and *Sales* groups. Any other combination fails.

That is genuine least privilege for access management, it's free, and it's GA.

!!! note "You've already seen this pattern built in"
    The **Key Vault Data Access Administrator** role ships with a built-in condition constraining it to the eight Key Vault roles — so someone can manage vault access without being able to grant themselves Owner. **Virtual Machine Data Access Administrator (preview)** does the same for VM login roles. When you get to Key Vault on Day 18, you'll recognise the shape.

### Hands-On: Constrain a Delegate ✅

Let's make Priya a limited access administrator: she may grant **Reader** to `grp-finance-team` in `rg-day17-demo`, and nothing else.

1. **`rg-day17-demo` → Access control (IAM) → + Add → Add role assignment**. **✅**
2. **Role tab:** *Privileged administrator roles* → **Role Based Access Control Administrator**. **✅**
3. **Members tab:** select `Priya Sharma` directly this time (delegation of admin rights to a named human is one of the few places a direct assignment is defensible — you want the audit trail to name a person). **✅**
4. **Conditions tab:** this role *requires* you to think about a condition — Azure prompts for one. Click **+ Add condition**. **✅**
5. Choose a **condition template**. Pick **"Constrain roles and principals"**: **✅**
   - **Roles:** select `Reader`
   - **Principals:** select `grp-finance-team`
6. **Review + assign**. **✅**
7. Test it, and this is the satisfying part. In the private window as Priya: **✅**
   - Go to `rg-day17-demo` → **Access control (IAM) → + Add → Add role assignment**. The option is now available to her at all — she has `roleAssignments/write`.
   - On the **Role** tab, **the list is reduced**. She sees Reader and essentially nothing else.
   - On the **Members** tab, **the principal list is reduced** to `grp-finance-team`.
   - Try to work around it — she cannot assign Owner, cannot assign to herself, cannot make anyone else an access administrator.

   Microsoft's troubleshooting docs even list *"Symptom — Roles or principals are not listed"* as an expected outcome of this feature, with the fix being "check whether a condition constrains you." Which is exactly what we just built.

8. **Remove this assignment when you're done** — Priya doesn't need standing access-admin rights, constrained or otherwise. **✅**

---

## Part 20 — Deny Assignments, Azure Policy, and the End of Blueprints

### Deny Assignments

A **deny assignment** blocks a set of actions for a set of principals at a scope, and **it beats every role assignment**. Remember step 4 of the evaluation flow: deny assignments are checked *first*, and if one matches, evaluation stops. Even an Owner is blocked.

The crucial fact: **you cannot create deny assignments directly.** There is no "+ Add deny assignment" button and no API for you to call. They are created *by Azure* on your behalf, in exactly three situations:

- **Azure managed applications** — the publisher's resources are protected from the customer
- **Deployment stacks** — you specify **deny settings** when creating the stack, and Azure creates deny assignments protecting the managed resources from modification or deletion outside the stack
- **Azure Blueprints** — historically, blueprint locking (and Blueprints is going away — see below)

Deployment stacks are the modern route, and they're the one you'll actually use. When we build IaC on Day 25, this is how you stop someone hand-editing a deployed resource in the portal.

The **Deny assignments** tab exists on every IAM blade, and on your lab subscription it will be empty. Look at it anyway so you know it's there.

### Azure Policy — A Different Question Entirely

Students constantly confuse Azure Policy with RBAC, so let's separate them with one line:

> **RBAC controls *who can perform actions*. Azure Policy controls *what resources are allowed to exist*.**

They're orthogonal, and you need both. RBAC says "Priya may create VMs in this resource group." Policy says "and any VM created here must be in an approved region, must have a `CostCenter` tag, and must not be a GPU size." Priya has permission; the resource still gets rejected.

**Policy effects**, in rough order of aggression:

| Effect | What it does |
|---|---|
| **Audit** | Logs non-compliance. Changes nothing. Always start here. |
| **AuditIfNotExists** | Audits when a related resource is missing (e.g. no diagnostic setting) |
| **Deny** | **Blocks the deployment outright** |
| **Modify** | Adds/updates tags or properties during creation |
| **DeployIfNotExists** | Auto-deploys a missing related resource — real remediation |
| **Disabled** | Turned off, kept for reference |

Policies are grouped into **initiatives** (policy sets) — for example the whole CIS or ISO 27001 benchmark as one assignable unit — and assigned at management group, subscription or resource group scope, inheriting downwards exactly like RBAC.

!!! note "DeployIfNotExists needs an identity — and RBAC"
    Here's where Policy and everything else in this video meet. `DeployIfNotExists` and `Modify` policies have to actually *do* something, so the policy assignment gets a **managed identity**, and that identity needs **role assignments** to perform the remediation. The portal offers to create them for you. This is a lovely closing loop: a governance rule, enforced by a managed identity, authorised by RBAC, using every concept from today at once.

### Azure Blueprints Is Retiring — Know the Dates

If you read an older Azure governance tutorial, or an older version of this course's own notes, you'll see **Azure Blueprints** presented as the way to package RBAC assignments, policies, ARM templates and resource groups into a repeatable governance artefact.

**Don't learn it. It's being retired.**

- Retirement was originally announced in September 2023.
- **From 31 July 2026, creation of new blueprint definitions is blocked.** Existing assignments keep running.
- **Full retirement is 31 January 2027.** At that point the API stops responding, the CLI and PowerShell commands stop working, and Blueprints disappears from the portal.
- Anything not exported before that date is **permanently deleted and unrecoverable**.

**The replacement path Microsoft prescribes:** migrate blueprint *definitions* to **Template Specs**, and blueprint *assignments* to **Azure Deployment Stacks**. Blueprint artefacts convert into ARM JSON or Bicep files that define the stack. And deployment stacks give you deny settings — which is how you get the resource-locking behaviour Blueprints used to provide.

We'll meet Template Specs and deployment stacks properly on **Day 25 (ARM & Bicep)**. For today, know the name, know it's dead, know what replaced it. That's an exam-relevant fact right now, because question banks lag reality by a year or two.

### Hands-On: A Real Policy ✅

1. Search **Policy** in the portal → **Authoring → Definitions**. Browse. There are hundreds of built-in definitions; use the **Category** filter (Tags, Compute, Storage, Key Vault, Security Center). **✅**
2. Find **`Require a tag on resources`**. Open it and read the JSON — note the `if` block (a tag is missing) and the `then` block (`"effect": "deny"`). Policy definitions are readable; spend a moment here. **✅**
3. Click **Assign**: **✅**
   - **Scope:** `rg-day17-demo`
   - **Parameters → Tag Name:** `CostCenter`
   - **Remediation:** leave off (this is a Deny policy, nothing to remediate)
   - **Review + create → Create**
4. Wait a few minutes for the assignment to activate — policy evaluation is not instant. **✅**
5. Test it: try to create any resource in `rg-day17-demo` **without** a `CostCenter` tag. **✅**

   **The deployment is rejected**, with a `RequestDisallowedByPolicy` error naming your policy assignment. You are the subscription **Owner**. You have every permission in Azure. **And it still failed** — because RBAC and Policy answer different questions, and Policy said no.

   That's the cleanest possible demonstration of why both systems exist.

6. Create the same resource **with** a `CostCenter` tag. It succeeds. **✅**
7. Go to **Policy → Compliance** and look at the compliance state of your scope. This is the dashboard a governance team lives in. **✅**
8. **Delete the policy assignment** before you move on, or every remaining demo in this video will fail. **✅**

---

## Part 21 — Privileged Identity Management (PIM)

### Standing Access Is the Problem

Think about who holds Owner or Global Administrator in a typical organisation. Now ask: how many hours per year do they actually *need* it?

Maybe twenty. And they hold it for 8,760.

Every hour of standing privilege is an hour that a phished session, a stolen laptop or a malicious insider can use. **Privileged Identity Management** shrinks that window: instead of holding the role, you're **eligible** for it, and you activate it when you need it — with justification, MFA, an approval step, and a hard expiry.

**PIM requires Entra ID P2 or Entra ID Governance.** It's the one paid thing in Part Two. **💳**

### The Core Vocabulary

Microsoft's terminology, which exams use verbatim:

| Term | Meaning |
|---|---|
| **Eligible** | The assignment exists but is dormant. The user must *activate* it to use it. |
| **Active** | A normal, working assignment. No action needed. |
| **Activate** | The act of turning an eligible assignment on — may require MFA, justification, approval, a ticket number |
| **Activated** | An eligible assignment that has been activated and is currently live |
| **Permanent eligible** | Always eligible to activate |
| **Time-bound eligible** | Eligible only between a start and end date |
| **Permanent active** | The old-fashioned standing assignment |
| **Time-bound active** | Active, but expires automatically on a date |
| **Just-in-time (JIT)** | The model: temporary permissions, granted at the moment of need |

### What PIM Gives You

- **Just-in-time** access to Entra roles **and** Azure resource roles
- **Time-bound** assignments with start and end dates — the contractor's access expires by itself
- **Approval workflows** — a named approver must say yes before activation
- **MFA on activation** — even if the original sign-in didn't require it
- **Justification** — a free-text reason, captured in the audit log
- **Notifications** when privileged roles are activated
- **Access reviews** — periodic "does this person still need this?" campaigns
- **Downloadable audit history** for compliance
- **Protection of the last Global Administrator** — PIM prevents removing the final one

PIM manages three things: **Microsoft Entra roles**, **Azure resource roles**, and **PIM for Groups** (making membership or ownership of a group itself eligible — which, combined with group-based RBAC, is extremely elegant: activate into the group, and inherit every role assignment the group holds).

### The Bonus Nobody Mentions

**Eligible role assignments do not count towards the 4,000-role-assignments-per-subscription limit.** Neither do assignments scheduled in the future. Microsoft lists "make role assignments eligible in PIM" as an official remedy for hitting the cap. So PIM improves your security posture *and* your scalability headroom.

### Who Can Manage What

A distinction that catches people out:

- **Entra roles in PIM** — managed by Privileged Role Administrator or Global Administrator. Global Admins, Security Admins, Global Readers and Security Readers can *view*.
- **Azure resource roles in PIM** — managed by a subscription administrator, resource Owner, or User Access Administrator. **Privileged Role Administrators do not get this by default.** Two planes, again, all the way down.

### Hands-On: Make Priya an Eligible Contributor 💳

1. Search **Privileged Identity Management** in the portal (or use the Entra admin center → **ID Governance → Privileged Identity Management**). **💳**
2. Note the three sections: **Microsoft Entra roles**, **Azure resources**, **Groups**. **💳**
3. Choose **Azure resources** → select your subscription → **Manage → Assignments → + Add assignments**. **💳**
4. Configure: **💳**
   - **Scope:** `rg-day17-demo`
   - **Role:** `Contributor`
   - **Member:** `Priya Sharma`
   - **Next → Assignment type: Eligible**
   - Untick **Permanently eligible** and set an end date **30 days** out — time-bound eligible, the strongest common configuration
   - **Assign**
5. Configure the role's activation rules: **Manage → Settings → Contributor → Edit**: **💳**
   - **Activation maximum duration:** 1 hour
   - **Require multifactor authentication on activation:** on
   - **Require justification on activation:** on
   - **Require approval to activate:** on → set **yourself** as the approver
   - Look at the **Notification** tab — who gets emailed on assignment, activation and approval
   - **Update**
6. Now be Priya. In the private window, go to **Privileged Identity Management → My roles → Azure resources → Eligible assignments**. **💳**
7. Click **Activate** next to Contributor: **💳**
   - Choose a duration (capped at your 1 hour)
   - Complete the **MFA** challenge
   - Type a **justification**: `Deploying hotfix for INC-4471`
   - **Activate**
8. She's told the request is **pending approval**. **💳**
9. Back as admin: **PIM → Approve requests** → review the justification → **Approve**. **💳**
10. As Priya: refresh. She is now a Contributor on `rg-day17-demo` — she can create and delete resources — **for one hour**. Check **`rg-day17-demo` → IAM → Role assignments** and you'll see the temporary assignment with an expiry time. **💳**
11. Let it expire (or **Deactivate** early). The permission vanishes with no human action. **💳**
12. Explore **PIM → Azure resources → Access reviews** and **Audit history** — the review campaign and the compliance evidence trail. **💳**

!!! tip "The architecture answer"
    Whenever a scenario says *"just-in-time,"* *"temporary elevation,"* *"approval workflow,"* *"time-bound admin access,"* or *"our auditors require evidence of why privileged access was used"* — the answer is **PIM**, and the licence is **P2**. Whenever it says *"a contractor needs access for six weeks and it must expire automatically"* — that's a **time-bound assignment**, and PIM is how you get one.

---

## Part 22 — Auditing, Troubleshooting, and the Hard Limits

### Where the Evidence Lives

Two different logs, for two different planes. Both free.

**Entra ID sign-in logs** — every authentication attempt. User, application, timestamp, IP, location, client app, device, Conditional Access policies evaluated, MFA method used, and a failure reason with an error code. Four categories, and the last two matter for everything you built today:

- **Interactive user sign-ins** — a human typed credentials
- **Non-interactive user sign-ins** — a cached or refresh token was used
- **Service principal sign-ins** — an application authenticated with its own credential
- **Managed identity sign-ins** — an Azure resource used its managed identity

Two error codes you will meet: **AADSTS50126** (invalid username or password) and **AADSTS50076** (MFA required by a Conditional Access policy — this one usually means everything is working).

**Entra ID audit logs** — every *change* to the directory: user created, group membership modified, Entra role assigned, app registered, policy updated, secret added.

**Azure Activity Log** — every control-plane operation on resources, **including role assignment writes and deletes**. This is where "who gave Priya Contributor, and when?" is answered. Different log, different plane, different blade.

**Retention is the catch.** Free tenants keep Entra sign-in and audit logs for **7 days**; premium tenants get **30**. The Activity Log keeps 90 days. Seven days is not an audit trail — for anything real, configure **Diagnostic settings** to export to a **Log Analytics workspace** (query with KQL), a **storage account** (cheap archive), or **Event Hub** (feed a SIEM). That's **Day 19 — Azure Monitor**, and it's the same diagnostic-settings pattern for every service in Azure.

### The Limits You Must Know

These are exam facts and real-world walls:

| Limit | Value | Notes |
|---|---|---|
| Role assignments per **subscription** | **4,000** | **Fixed. Cannot be increased.** Includes subscription, RG and resource scopes. |
| Role assignments per **management group** | **500** | Also fixed. Separate from the subscription limit. |
| Custom roles per **tenant** | **5,000** | 2,000 in 21Vianet |
| `AssignableScopes` per custom role | 2,000 | One management group max |
| Expressions per condition (visual editor) | 5 | More via the code editor |

**Eligible (PIM) assignments and future-scheduled assignments do not count** towards the 4,000.

Microsoft's five documented remedies when you hit the wall, in priority order:

1. **Replace per-user assignments with group assignments** — the biggest win by far
2. **Remove redundant assignments** — the same role for the same principal at both RG and subscription scope
3. **Combine several built-in role assignments into one custom role**
4. **Make assignments eligible in PIM** (P2)
5. Add another subscription

### The Gotchas That Waste Your Afternoon

**Propagation delay.** Role assignment changes take **up to 10 minutes**. Sign out and back in to force a token refresh. At management group scope with data actions, it can take *hours*.

**Managed identity group caching — up to 24 hours.** Covered in Part 17. Assign directly to the identity when speed matters.

**"Identity not found" / Unknown type.** You deleted a user, group or service principal without removing its role assignments. The assignment survives, pointing at a dead GUID. Harmless but messy — and it eats into your 4,000. Clean them up. (It can also just be replication lag on a brand-new principal; wait a minute and refresh.)

**PrincipalNotFound on brand-new principals.** Create a principal and immediately assign a role, and it can fail because the principal hasn't replicated to the region handling your assignment. In ARM/Bicep, set the `principalType` property to fix it. In PowerShell, set `-ObjectType`.

**Role assignments don't move with resources.** Move a resource to another resource group and any assignment made *directly on that resource* is orphaned. Recreate it — ideally remove it before the move.

**Subscription transfer wipes everything.** Transfer a subscription to a different tenant and **all** role assignments are permanently deleted and managed identities must be recreated by hand. Say it twice; people get bitten by this once.

**You cannot delete the last Owner** on a subscription by default. There's a feature flag to allow it, and a Global Admin can override, but the guardrail exists to stop you orphaning a subscription.

**Bicep/ARM role assignment names must be deterministic GUIDs.** Use `guid(scope, principalId, roleDefinitionId)`. Reuse a name and the deployment fails with `RoleAssignmentUpdateNotPermitted`, because role assignments are immutable — you can't change their properties, only delete and recreate. Remember this for Day 25.

### Hands-On: Audit What You Did Today ✅

1. **`rg-day17-demo` → Activity log**. Filter **Operation: Create role assignment**. Every assignment you made today is listed with who made it and when. Open one and read the JSON. **This is the answer to "who granted that?"** **✅**
2. **Entra ID → Monitoring & health → Sign-in logs**: **✅**
   - Find your own sign-in. Work through the tabs: **Basic info**, **Location**, **Device info**, **Authentication Details**, **Conditional Access**, **Report-only**.
   - Find **Priya's** sign-ins — including the failed authorisation moments.
   - Switch to **Service principal sign-ins** and find `app-lwm-demo` from Part 7.
   - Switch to **Managed identity sign-ins** and find your App Service or VM from Part 17. **Your infrastructure signing in, logged like a person.**
3. **Entra ID → Audit logs**, filter **Service: Core Directory**. Today's session as an audit trail: *Add user*, *Add group*, *Add member to group*, *Add application*, *Add service principal*, *Update policy*. **✅**
4. Count your role assignments. **Subscription → Access control (IAM) → Role assignments** and look at the **chart at the top of the page** showing your count against the 4,000 limit. **✅**
5. Optional, and genuinely useful in real life — **Azure Resource Graph Explorer**. Search for it in the portal, set **Scope: Directory**, and run: **✅**
   ```kusto
   authorizationresources
   | where type =~ "microsoft.authorization/roleassignments"
   | extend principalId = tostring(properties.principalId)
   | extend scope = tostring(properties.scope)
   | extend roleId = tolower(tostring(properties.roleDefinitionId))
   | join kind=leftouter (
       authorizationresources
       | where type =~ "microsoft.authorization/roledefinitions"
       | extend roleName = tostring(properties.roleName)
       | extend roleId = tolower(id)
       | project roleName, roleId
   ) on roleId
   | project roleName, principalId, scope
   | order by roleName asc
   ```
   Every role assignment in your tenant, in one table. This query — or a variant of it — is how real organisations audit access at scale, and it's the basis of every "find unused custom roles" and "find duplicate assignments" recipe in Microsoft's docs. **✅**
6. Open **Monitoring & health → Diagnostic settings → + Add diagnostic setting** in Entra ID. Look at the categories (`SignInLogs`, `AuditLogs`, `NonInteractiveUserSignInLogs`, `ServicePrincipalSignInLogs`, `ManagedIdentitySignInLogs`) and the destinations. **Don't create it today** — we'll do it properly with a workspace on Day 19. **✅**

---

## Part 23 — Exam Framing

Identity and access control are heavily weighted on AZ-104 and AZ-305. The questions are pattern-matched to keywords — here's the combined lookup table for both halves of today.

### Identity

| If the question says… | The answer is… |
|---|---|
| "Create users / reset passwords / register apps" | **Entra ID role** (User Administrator, Application Administrator) |
| "Membership must update automatically based on department" | **Dynamic group** (requires P1) |
| "Require MFA only from outside the corporate network" | **Conditional Access** with a named location (P1) |
| "Simplest way to require MFA for everyone, no premium licence" | **Security defaults** |
| "Grant an external partner access without creating them an account" | **B2B guest invitation** |
| "Must keep working if on-premises servers are unavailable" | **Password Hash Synchronisation** |
| "No password material may be stored in the cloud" | **Pass-Through Authentication** |
| "Recover a user deleted last week" | **Deleted users** — restorable for 30 days |
| "Retain sign-in logs beyond 30 days" | **Diagnostic settings → Log Analytics / storage account** |
| "A pipeline must deploy to Azure with no stored secret" | **Workload identity federation** (federated credential) |
| "Grant an application read access to all mailboxes with no user" | **Application permission** + admin consent |
| "App should only see files the signed-in user can see" | **Delegated permission** |

### Authorisation

| If the question says… | The answer is… |
|---|---|
| "Assign permissions to Azure resources" | **Azure RBAC role** — not an Entra role |
| "Grant the minimum access to manage VMs" | **Virtual Machine Contributor** at the narrowest scope — not Contributor |
| "Manage everything but must not grant access to others" | **Contributor** |
| "Grant access to others but must not manage resources" | **User Access Administrator**, or better, **Role Based Access Control Administrator** |
| "Delegate role assignment but only for specific roles/principals" | **RBAC Administrator + ABAC condition** (constrained delegation) |
| "Apply one assignment across many subscriptions" | **Management group scope** |
| "The user is Contributor but can't read blobs" | Assign a **data-plane role** — Storage Blob Data Reader |
| "Built-in roles don't fit our needs" | **Custom role** (free) |
| "Access based on a resource tag or the time of day" | **ABAC role assignment condition** |
| "Block an action regardless of role" | **Deny assignment** — created by deployment stacks / managed apps, not by hand |
| "All resources must carry a CostCenter tag" | **Azure Policy** with the **Deny** effect |
| "Automatically create a missing diagnostic setting" | **Azure Policy** with **DeployIfNotExists** (+ a managed identity with RBAC) |
| "Just-in-time, time-bound admin access with approval" | **PIM** (P2) |
| "Contractor access must expire automatically" | **Time-bound assignment** (PIM) |
| "An Azure service must access another Azure service securely, no credentials" | **Managed identity** + a role assignment |
| "Identity must survive VM recreation / be shared by several resources" | **User-assigned managed identity** |
| "Identity should be deleted when the resource is deleted" | **System-assigned managed identity** |
| "We're at 4,000 role assignments" | **Assign to groups**; remove redundant assignments; combine into custom roles; make them eligible in PIM |
| "Package governance artefacts for new subscriptions" | **Template Specs + Deployment Stacks** — *not* Blueprints (retiring 31 Jan 2027) |
| "Global Admin can't see any subscriptions" | **Elevate access** toggle — then turn it back off |

---

## Part 24 — Cleanup ✅

Cleanup today has two halves, because you created things in two different planes. **Deleting a resource group does not remove anything from the directory** — users, groups, app registrations and guests live in the tenant, not in a subscription.

**Resource plane:**

1. **Delete the policy assignment** if you haven't already: **Policy → Assignments → Require a tag on resources → Delete**. **✅**
2. **Remove the RBAC Administrator assignment** from Priya (`rg-day17-demo` → IAM → Role assignments). Nobody keeps standing access-admin rights. **✅**
3. **Delete the custom role**: `rg-day17-demo` → IAM → **Roles** tab → filter **CustomRole** → `LWM VM Operator` → **…** → **Delete**. **Remove its role assignment first** or you'll get `RoleDefinitionHasAssignments`. **✅**
4. **Delete the resource group** `rg-day17-demo`. This removes the App Service, the App Service plan, the storage account, the user-assigned managed identity, and — automatically — every role assignment scoped inside it. **✅**
5. If you enabled a system-assigned identity on a Day 3 VM and gave it a storage role, that role assignment is on the (now deleted) storage account, so it goes too. **Turn the VM's identity off** if you want a clean slate. **✅**
6. **Check the subscription IAM blade** for orphaned assignments showing *Identity not found* and remove them. **✅**

**Identity plane:**

7. **Delete the app registration:** **Entra ID → App registrations → app-lwm-demo → Delete**. This kills the client secret. Do it. **✅**
8. **Check Enterprise applications** for a leftover service principal with the same name and delete it too. **✅**
9. **Remove the Entra role assignment** you gave Priya in Part 12 (**Roles & admins → User Administrator → Assignments → remove**). **✅**
10. **Decide about Priya and `grp-finance-team`.** They cost nothing. **My recommendation: keep them.** Day 18 (Key Vault) uses exactly this pattern again — a group, a data-plane role, a managed identity — and having the principals already there saves you ten minutes. **✅**
11. **Remove the guest** if you don't want it: **Users → External Auditor → Delete**. **✅**
12. **If you enabled MFA on your own account — keep it.** Genuinely. It's free, it's the single highest-value security change you can make to your Azure account, and Microsoft mandates it for the portal and ARM anyway. **✅**
13. **If you activated a P1/P2 trial**, put the cancellation reminder in your calendar right now, on camera. **💳**
14. **If you disabled security defaults** for the Conditional Access lab and you're *not* keeping premium licences: **turn security defaults back on**. Leaving a tenant with neither security defaults nor Conditional Access is worse than where you started. **💳**
15. **If you created PIM assignments**, deactivate or remove them. **💳**
16. **Keep `db-lwm-demo`.** It's on the free offer and Day 18 (Key Vault) and Day 30 (Capstone) both use it. **✅**

---

## Summary

Today you stopped asking "how do I build it?" and started asking two much harder questions: **"who are you?"** and **"what exactly are you allowed to do?"**

**Microsoft Entra ID is the identity plane** — a global, region-less service holding users, groups, applications and devices, completely separate from the resource plane where your VMs and databases live. Those two planes have different containers, different role systems and different administrative blades. The moment you internalise that split, Azure permissions stop being confusing: **a Global Administrator can't see a virtual machine, and a subscription Owner can't create a user.** We proved it three times — once with Priya, once with a service principal, once with a User Administrator who could create a hundred users and not see one resource group.

**Users and groups** are the raw material, and the operating rule is permanent: **never assign permissions to a person.** It's not just tidiness — it's the documented remedy for the hard 4,000-assignment ceiling. Dynamic membership (P1) takes it further, letting HR data drive Azure access with no human in the loop.

**Authentication is where the free value is.** MFA blocks over 99% of identity attacks and costs nothing. Security defaults deliver six real protections with one switch. Microsoft has made MFA **mandatory** for the Azure portal and the ARM layer — which means CLI, PowerShell, Terraform and Bicep, on every tenant. And the direction of travel is unmistakable: **passkeys become the default from 1 September 2026**, with SMS and voice being retired. Don't design around text messages.

**Applications get identities too** — app registrations, service principals, and the best idea in Azure security, **managed identities**. You watched a connection string lose its password entirely. Not encrypted, not vaulted — *gone*. Then you watched a virtual machine list and download a file from storage with no username, no password, no key, and no connection string anywhere on the box.

Then everything flipped. **Azure RBAC is one sentence: who can do what, where.** Security principal, role definition, scope. Every permission in Azure is that sentence written down somewhere, and every access problem you will ever debug is one of those three words being wrong.

You saw **scope and inheritance** flow from management group down to a single resource, and you saw the exact instant Priya's empty portal filled up. You read the JSON and discovered that **Contributor is literally Owner minus the ability to grant access** — one `NotActions` list, nothing more. You made a Contributor on a storage account **who could delete the account and could not read a file inside it**, which is the single most surprising and most important fact in Azure authorisation. You gave a role to a machine. You built a custom role and cut it down to three actions and one resource group. You constrained an administrator so she could hand out exactly one role to exactly one group, and watched the portal's own dropdowns shrink to match.

And you saw the systems that sit alongside RBAC without being RBAC: **deny assignments** you can't create by hand, **Azure Policy** that rejected a deployment from a subscription Owner because a tag was missing, **Blueprints** on its way out the door by January 2027, and **PIM** turning permanent admin rights into a one-hour, approved, justified, audited, self-expiring loan.

Two questions. Two systems. One video, because in real life you never get to answer just one of them.

### What's Next

Every service in Azure you touch from here uses what you learned today. But one of them is built almost entirely out of it.

Tomorrow, **Day 18 — Azure Key Vault** is where secrets go to live properly. And it's the perfect sequel, because Key Vault has *both* permission models sitting side by side: the legacy **access policy** model and the modern **Azure RBAC** model, with data-plane roles like *Key Vault Secrets User* that behave exactly like the Storage Blob Data Reader you met in Part 16. You'll see **Key Vault Data Access Administrator** — the built-in constrained-delegation role from Part 19 — doing its job in the wild. And you'll wire a managed identity to a vault so an application reads a secret with no credential, using precisely the pattern you built today.

If today felt like a lot, that's because it was two videos. Everything after this is one.

---

## Key Takeaways

**Identity**

- **Two planes.** Entra ID (identity) and Azure Resource Manager (resources) are separate systems with separate role models. Global Administrator ≠ Owner. Entra ID has no region; it's global. RBAC data is global too.
- **Tenant / directory / subscription:** one subscription trusts exactly one tenant; one tenant can hold many subscriptions. **Transferring a subscription between tenants permanently deletes every role assignment** and orphans managed identities.
- **Entra ID is not Active Directory.** OAuth/OIDC/SAML vs Kerberos/LDAP, flat vs hierarchical, Graph vs LDAP, no Group Policy.
- **Authentication ≠ authorisation.** A valid user with no RBAC sees an empty portal. That's both systems working correctly.
- **The licensing line: MFA is free; deciding when to require it is P1.** Conditional Access, dynamic groups, custom *Entra* roles = P1. Identity Protection, PIM, access reviews = P2. **Azure RBAC in its entirety is free.**
- **Free tier limit:** 50,000 directory objects, rising to 300,000 once you verify a custom domain.
- **Never assign permissions to a person — assign to a group.** Security group for RBAC; M365 group for collaboration. Assigned membership free, dynamic membership P1.
- **Deleted users are recoverable for 30 days**, with object ID and role assignments intact.
- **Security defaults enforce six things free** and are mutually exclusive with Conditional Access. **MFA is now mandatory for the Azure portal and the ARM layer** — CLI, PowerShell, Terraform, Bicep included.
- **Passkeys become the default from 1 September 2026**; SMS and voice MFA are retiring, with a blocking prompt from 1 February 2027.
- **Conditional Access = IF (users + resources + conditions) THEN (grant/session controls).** Always start **report-only**. Always keep **two break-glass Global Admin accounts excluded from every policy**, Microsoft-managed ones included.
- **App registration = blueprint** (one, global). **Service principal = instance** (one per tenant). **Enterprise application = the portal's name for the service principal.**
- **Client secret value is shown exactly once.** Prefer certificates; prefer **federated credentials** over both.
- **Delegated permission** = acts as the signed-in user (intersection). **Application permission** = acts as itself, tenant-wide, always needs admin consent. **Graph permissions are not Azure RBAC.**
- **Managed identities are free, have no readable credential, and rotate themselves.** System-assigned dies with its resource; user-assigned is independent and shareable — better when compute is recycled often.
- **`Authentication=Active Directory Default` + `CREATE USER [name] FROM EXTERNAL PROVIDER`** is the complete passwordless recipe for App Service → Azure SQL.
- **Tokens come from `169.254.169.254`** — link-local, reachable only from inside the resource.
- **B2B guests use their own identity from their own tenant**, carry `#EXT#` in their UPN, and can hold Azure role assignments like anyone else. **Azure AD B2C is retired**; Entra External ID succeeds it.
- **Hybrid: PHS = cloud verifies** (most resilient, recommended), **PTA = on-prem verifies with an agent**, **Federation = on-prem verifies with an AD FS farm**.

**Authorisation**

- **Every role assignment is three things: security principal + role definition + scope.** Who, what, where. There is no other mechanism.
- **Four principal types:** user, group, service principal, managed identity. Software is a first-class citizen.
- **Four scope levels:** management group → subscription → resource group → resource. Permissions inherit **downwards only**.
- **RBAC is additive** — your effective access is the union of every assignment, including transitive group memberships. Adding a narrower role never removes access.
- **Golden rule:** least privileged role, narrowest scope, assigned to a group. **Maximum three subscription Owners.**
- **`Actions − NotActions` and `DataActions − NotDataActions`** produce your effective permissions. **NotActions is subtraction, not denial.**
- **Contributor = Owner minus `Microsoft.Authorization/*/write` and `/delete`.** That single `NotActions` list is the entire difference.
- **Control plane ≠ data plane.** A Contributor on a storage account **cannot read a blob** — that needs Storage Blob Data Reader. The same split applies to Key Vault, Cosmos DB and ACR. Watch for the `listKeys` back door.
- **Five privileged administrator roles:** Owner, Contributor, Reservations Administrator, **Role Based Access Control Administrator**, User Access Administrator. Everything else is a job function role — prefer those.
- **Prefer RBAC Administrator over User Access Administrator** — same job, fewer permissions. Then constrain it with a condition.
- **Custom roles are free**, capped at **5,000 per tenant**, need explicit `AssignableScopes` (one management group max, no `/`, no wildcards), and can't be deleted while assignments reference them. **Avoid wildcards in actions.**
- **Assign by role ID, not role name**, in any automation.
- **ABAC conditions** narrow a role assignment by attribute — GA for blob and queue data actions, with condition templates in the portal, environment attributes, and custom security attributes on the principal. They can only restrict, never expand, and never explicitly deny a resource.
- **Constrained delegation** = RBAC Administrator + a condition limiting which roles and which principals a delegate may touch. Key Vault Data Access Administrator ships with this built in.
- **Deny assignments beat everything** and are evaluated first — but you can't create them by hand. They come from deployment stacks, managed applications and (historically) Blueprints.
- **RBAC controls who can act; Azure Policy controls what may exist.** A subscription Owner can still be blocked by a Deny policy. `DeployIfNotExists` needs a managed identity with its own role assignments.
- **Azure Blueprints is retiring** — new definitions blocked from 31 July 2026, full retirement 31 January 2027, unexported content permanently deleted. Migrate to **Template Specs + Deployment Stacks**.
- **PIM (P2)** converts standing privilege into just-in-time, time-bound, approval-gated, MFA-enforced, justified, audited access. **Eligible assignments don't count towards the 4,000 limit.**
- **The hard numbers:** **4,000** role assignments per subscription (fixed), **500** per management group (fixed), **5,000** custom roles per tenant.
- **The ten-minute rule:** role assignment changes take up to 10 minutes; sign out and in to refresh your token. **Managed identity group membership caches for up to 24 hours** — assign directly to the identity when speed matters.
- **Classic administrator roles are fully retired as of May 2026.** RBAC is the only access model.
- **Role assignments don't follow moved resources**, can't be modified once created (delete and recreate), and need deterministic GUID names in Bicep/ARM via `guid(scope, principalId, roleDefinitionId)`.
- **Deleting a principal leaves an orphaned assignment** showing *Identity not found* — clean them up; they count towards your limit.
