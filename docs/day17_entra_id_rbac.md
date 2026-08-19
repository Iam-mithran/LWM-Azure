# Day 17 — Microsoft Entra ID & Azure RBAC

**Phase 5 — Identity, Security + Monitoring**

> For sixteen days there has been exactly one person allowed to touch anything you built: you. Every click you made in the portal was authorised because of one identity — the account you signed up with on Day 1 — and you never thought about it, because it just worked. Today we open that box. Because the moment a second person joins your project, or an application needs to read your database, "it just works" stops being good enough. You'll learn what happens in the half-second between typing your password and seeing the portal — **and what happens in the half-second after that**, when Azure decides whether you're actually allowed to do the thing you clicked. Two different questions, two different systems, and almost every confusing thing in Azure security comes from mixing them up.

---

## What You'll Learn

**Part One — Identity: who are you?**

- What Microsoft Entra ID is, and the **two planes** — the one idea that makes Azure permissions stop being confusing
- Tenant, directory, subscription — and why Entra ID isn't "Active Directory in the cloud"
- Users and groups, and the rule you'll follow for the rest of your career: **never assign permissions to a person**
- MFA, security defaults, and where Conditional Access fits
- How applications prove who they are — app registrations, service principals, and **managed identities**

**Part Two — Authorisation: what are you allowed to do?**

- Entra roles vs Azure roles — why a Global Administrator can't see a single virtual machine
- The three-part sentence every role assignment is made of: **principal + role + scope**
- What's actually inside a role definition, and why **Contributor is Owner minus one thing**
- **Control plane vs data plane** — the most surprising fact in Azure, proved live
- Custom roles, Azure Policy, and PIM
- An interview and exam keyword table for both halves

---

## Before We Begin

**Today costs nothing.** Every single hands-on step in this video is free tier. Identity and access control are where Microsoft gives away the fundamentals and charges only for enterprise-scale controls.

| Free — and we'll use all of it | Paid — explained today, not demonstrated |
|---|---|
| Users, groups, security defaults, MFA, SSPR | **Conditional Access** (Entra ID P1) |
| App registrations, service principals, B2B guests | **Dynamic group membership** (P1) |
| **Managed identities** — no cost, ever | **PIM** — just-in-time admin access (P2) |
| **Azure RBAC in its entirety** — every role assignment, every custom role | Identity Protection, access reviews (P2) |
| Azure Policy built-in definitions | |

That second column still matters — it comes up in interviews constantly, and you'll be expected to know what each one does and which licence it needs. So we cover the concepts properly. You just won't be asked to click anything you'd have to pay for.

!!! warning "The one thing that can genuinely lock you out"
    Identity is the only area of Azure where a mistake can lock **you** out of **your own tenant**, and there's no "delete the resource group" undo for that. Two rules:

    1. Before you enable MFA on your admin account, **have your phone in your hand** with Microsoft Authenticator installed.
    2. **Never remove your own Owner role assignment** at subscription scope to see what happens. Azure will usually stop you. Don't go looking for the edge case.

**Set this up first:**

- A resource group called `rg-day17-demo`.
- The **Microsoft Authenticator** app on your phone (free).
- A **private/incognito window**. We'll be signing in as a test user constantly, and you don't want to keep logging out of your admin session. Genuinely, do this — half the payoff moments of today happen in that window.

---

# PART ONE — IDENTITY

## Part 1 — What Entra ID Is, and the Two Planes

You type `portal.azure.com`, enter a password, approve a prompt, and the portal loads. In that half-second, two separate questions were answered:

1. **Who are you?** — **authentication**. Microsoft Entra ID answers this.
2. **What are you allowed to do?** — **authorisation**. Azure RBAC answers this.

Every security problem in every cloud reduces to those two questions. **Microsoft Entra ID is Microsoft's cloud identity service** — the directory holding your users, groups and application identities, and the thing that issues **tokens**: short-lived signed proofs of identity that Azure, Microsoft 365 and thousands of SaaS apps accept as evidence of who you are.

!!! note "The name"
    You'll see this called **Azure Active Directory**, **Azure AD** and **AAD** all over the internet. Microsoft renamed it **Microsoft Entra ID** in 2023 — same product, no migration, pure branding. When you read older content, mentally substitute.

### The Two Planes — The Idea That Explains Everything

This is the most important mental model today. Azure has **two separate planes**, managed in different places, by different services, with different permission systems:

| | **Identity plane** | **Resource plane** |
|---|---|---|
| **Service** | Microsoft Entra ID | Azure Resource Manager (ARM) |
| **Contains** | Users, groups, app registrations | VMs, storage accounts, VNets, databases |
| **Organised by** | Tenant → directory | Management group → subscription → resource group → resource |
| **Permissions via** | **Entra roles** (e.g. *User Administrator*) | **Azure RBAC roles** (e.g. *Contributor*) |
| **Managed in** | Entra ID → Roles & admins | Any resource → Access control (IAM) |
| **Lives in a region?** | **No** — global | **Yes** — every resource has a region |

Look at that last row. Everything you built for sixteen days had a region. **Entra ID has no region** — your tenant is a global service. You will never see a "location" field on a user.

And look at the permissions row, because it's the trap: **being a Global Administrator in Entra ID gives you access to zero Azure resources.** Not one VM. Different planes, different role systems. We prove it in Part 8.

### Tenant, Directory, Subscription

Three words, constantly confused:

- **Tenant** — your organisation's dedicated, isolated instance of Entra ID. You got one automatically on Day 1. It has a globally unique **Tenant ID** (a GUID) and a domain ending `.onmicrosoft.com`. A tenant is a hard security boundary.
- **Directory** — the store of objects inside the tenant. In practice "tenant" and "directory" are used interchangeably; don't lose sleep over it.
- **Subscription** — a billing and resource container that **trusts exactly one tenant** for authentication.

The rules that show up in interviews almost verbatim:

- **One subscription trusts exactly one tenant.** One tenant can hold many subscriptions.
- **A subscription can be moved to another tenant — and all its RBAC assignments are permanently deleted**, because the users they referenced don't exist there. This is one of the nastiest surprises in Azure.
- **A user cannot be moved between tenants.** You invite them as a guest instead.

### Entra ID Is Not Active Directory

If you've worked in corporate IT you've met **Active Directory Domain Services** — domain controllers, Group Policy, OUs. It's tempting to assume Entra ID is that, hosted by Microsoft. It isn't.

| | **AD DS** | **Microsoft Entra ID** |
|---|---|---|
| **Protocols** | Kerberos, NTLM, LDAP | OAuth 2.0, OpenID Connect, SAML |
| **Designed for** | Devices on a corporate LAN | Applications on the internet |
| **Structure** | Hierarchical — forests, domains, OUs | Flat — one directory of objects |
| **Group Policy** | Yes | No |

You cannot domain-join a machine to Entra ID in the AD DS sense, and there are no OUs. Most real organisations run **both** — AD DS on-premises, Entra ID in the cloud, and a sync tool joining them (Part 7).

### Two Portals, One Tenant

- **`portal.azure.com`** — the Azure portal you've used all course. Everything in Part Two lives here.
- **`entra.microsoft.com`** — the **Microsoft Entra admin center**, a dedicated identity console. Newer features land here first, and Microsoft's docs now write every set of steps for it.

Same tenant, two front doors. I'll use the Entra admin center for identity and the Azure portal for RBAC — exactly the split you'll use in real life.

### Hands-On: Find Your Tenant ✅

1. Go to **`entra.microsoft.com`** and sign in. In the left navigation, expand **Entra ID → Overview**. **✅**
2. Note three things: **✅**
   - **Tenant ID** — a GUID. **Copy it somewhere you'll find again.** It turns up in connection strings, app configuration, Terraform, `az login --tenant` — constantly.
   - **Primary domain** — `something.onmicrosoft.com`
   - **License** — almost certainly **Microsoft Entra ID Free**
3. Click the **Properties** tab and note two settings we'll come back to: **Access management for Azure resources** (Part 8) and **Manage security defaults** (Part 4). **✅**
4. Open **`portal.azure.com`** in another tab and search **Microsoft Entra ID**. Same tenant, same ID, different chrome. **✅**

---

## Part 2 — Users

A **user object** is a set of attributes plus credentials plus role assignments. It's the thing that can sign in.

Two attributes matter today:

- The **User Principal Name (UPN)** — looks like an email address, and it's the globally unique sign-in name. The domain part must be one your tenant owns.
- The **Object ID** — the user's GUID. **When you create a role assignment, Azure doesn't store "Priya Sharma." It stores that GUID.** Which is why deleting a user without cleaning up their role assignments leaves behind ghosts labelled *Identity not found*.

**Three sources of users:**

- **Cloud-only** — created directly in Entra ID. Everything we do today.
- **Synced** — copied up from on-premises Active Directory. **Most attributes are read-only in the portal** — you edit them on-premises. If you've wondered why a name field is greyed out, that's why.
- **Guest** — an identity that lives in another tenant, invited into yours (Part 7).

!!! tip "Deleting a user is reversible for 30 days"
    Deleted users go to a holding area for **30 days** and can be restored completely — **same object ID**, so their Azure role assignments come back working. Creating a fresh user with the same name would *not* restore access, because the GUID would be different. This is a favourite interview question.

### Hands-On: Create a User ✅

1. **Entra ID → Users → All users → + New user → Create new user**. **✅**
2. **Basics:** **✅**
   - **User principal name:** `priya` — pick your `.onmicrosoft.com` domain from the dropdown
   - **Display name:** `Priya Sharma`
   - **Password:** reveal and **copy it now** — this screen never comes back
3. **Properties tab:** set **Job title** `Financial Analyst`, **Department** `Finance`, and **Usage location** — that last one isn't optional if you ever assign licences, and it's the most common cause of "why won't this licence assign" tickets. **✅**
4. Skip **Assignments** — that's the entire second half of this video, and I want you to feel its absence first. **Review + create → Create**. **✅**
5. Open `Priya Sharma` and copy her **Object ID**. Look down the left menu — **Assigned roles** (Entra roles, Part 8), **Groups**, **Authentication methods** (the helpdesk blade), and **Azure role assignments**, which is empty and which we fill in Part 9. **✅**

### Hands-On: Sign In as Priya — and See Nothing ✅

This is the hinge the whole video turns on. Don't skip it.

1. Open a **private/incognito window**, go to `portal.azure.com`, and sign in as `priya@yourtenant.onmicrosoft.com`. **✅**
2. Change the password when prompted. If security defaults are on you'll be asked to register for MFA — complete it or skip if offered. **✅**
3. Go to **Subscriptions**. **✅**

**The list is empty.**

Priya is a fully valid, fully authenticated user. Entra ID confirmed exactly who she is. And she can see **nothing** — not one resource group, not one VM.

That's the two planes, live. **Authentication succeeded. Authorisation was never granted.** Both systems are working perfectly, and the result is a user who can log in and do absolutely nothing.

Hold onto that empty screen. In Part 9 we fix it in about forty seconds, and the contrast is the whole lesson.

4. While you're there, visit **`myaccount.microsoft.com`** as Priya — the self-service page every user gets, where they manage their own MFA methods and see their sign-in activity. Knowing this URL saves your helpdesk enormous time. **✅**
5. **Leave this window open.** We come back to it repeatedly.

---

## Part 3 — Groups

### Never Assign Permissions to a Person

That's the rule, and it's the most important operational habit in this video.

Assign permissions directly to users and after eighteen months you have four hundred individual assignments scattered across sixty resource groups, nobody knows who has access to what, and someone leaves with their access lingering in nine places.

Assign to **groups** and access becomes a membership question. New analyst joins? Add to the group. Leaves? Remove. Access appears and disappears everywhere, instantly, in one action.

There's a hard technical reason too: **a subscription supports a maximum of 4,000 role assignments, and that limit cannot be raised.** Per-user assignments burn through it. One group assignment can serve ten thousand people. Microsoft's own documented remedy for hitting the ceiling is literally "replace principal-based assignments with group-based assignments."

### Two Group Types, Two Membership Types

| | **Security group** | **Microsoft 365 group** |
|---|---|---|
| **Purpose** | Granting access — RBAC, app assignment | Collaboration — mailbox, SharePoint, Teams |
| **Use in Azure RBAC** | **Yes — always this one** | Not the normal path |

**Membership:**

- **Assigned** — you add and remove members by hand. **Free.**
- **Dynamic** — you write a rule and Entra ID maintains membership automatically. **Requires P1 — 💳 concept only today.**

A dynamic rule is a query over user attributes:

```text
(user.department -eq "Finance") and (user.country -eq "India")
```

The power is operational: HR changes someone's department, it syncs to Entra ID, and within minutes their group memberships — **and therefore their Azure permissions** — reshape automatically. Nobody files a ticket. That's HR data driving cloud authorisation with no human in the middle, and it's why P1 is worth the money in a real organisation.

**Group owners** can manage membership without being directory admins — proper delegation. And **Azure RBAC honours nested groups**: if you're in group A, and A is in B, and B has a role assignment, you have that role. Two levels is plenty; deeper becomes impossible to audit.

### Hands-On: Create a Security Group ✅

1. **Entra ID → Groups → All groups → + New group**. **✅**
2. **Group type:** *Security*. **Name:** `grp-finance-team`. **Membership type:** *Assigned* — open that dropdown first and notice *Dynamic User* is listed but blocked. **That greyed-out option is the P1 paywall, visible.** **✅**
3. Add yourself as **Owner** and `Priya Sharma` as a **Member**. **Create**. **✅**
4. Open the group and note its **Object ID** — **this is what the role assignment in Part 9 will reference.** Look at **Azure role assignments** in the left menu; empty for now. **✅**

---

## Part 4 — Authentication: MFA, Security Defaults and SSPR

### Why Passwords Lose

A password is a shared secret, and shared secrets leak — phished, reused, breached in bulk from some unrelated site, sprayed against your tenant. Microsoft's published figure is the one to remember: **MFA blocks over 99% of identity-based attacks.** Not because the second factor is unbreakable, but because it breaks the attacker's economics.

Real MFA needs factors from **different categories** — something you know (password), something you have (phone, security key), something you are (fingerprint, face). Password plus security question is *not* MFA. Password plus Authenticator push *is*.

### The Methods, Ranked

| Method | Strength | Notes |
|---|---|---|
| **Passkeys (FIDO2)** | **Phishing-resistant** | Cryptographically bound to the real domain — a fake login page can't use it |
| **Certificate-based auth** | Phishing-resistant | Smart cards; regulated industries |
| **Microsoft Authenticator (push + number matching)** | Strong | The free default for most organisations |
| **OATH TOTP codes** | Strong | 6-digit rotating codes, works offline |
| **SMS / voice** | **Weak** | SIM swapping. **Being retired.** |

**Number matching** deserves a mention. Older push notifications had one button: Approve. Attackers exploited that with "MFA fatigue" — spam prompts at 3am until someone taps Approve to make it stop. Now the sign-in screen shows a two-digit number you must type into the app. You can't approve a prompt you didn't initiate, because you can't see the number.

!!! info "Passkeys become the default from September 2026"
    From **1 September 2026** Microsoft begins rolling out **passkeys as the default authentication experience** in Entra ID, and **SMS and voice MFA are being retired**. From **1 February 2027**, users relying only on SMS or voice get a blocking passkey-registration prompt.

    Practical takeaway: **do not design around SMS.** If you're advising an organisation, this migration is a now project.

### Security Defaults — Free, Blunt, Better Than Nothing

**Security defaults** is one on/off switch applying a fixed bundle of protections to your whole tenant. It's free, needs no configuration, and if your tenant was created after October 2019 it's very likely **already on**.

What it enforces:

1. **All users must register for MFA**
2. **Administrators do MFA every time** they sign in
3. **Users do MFA when necessary**, based on risk signals
4. **Legacy authentication protocols are blocked** — IMAP, POP3, SMTP AUTH. Over 99% of password-spray attacks arrive over these, and they can't do MFA, so they're a permanent bypass. This is the highest-value item on the list.
5. **Privileged activity is protected** — MFA for the portal, Entra admin center, Azure PowerShell and CLI

Its limitation is that it's binary. On or off, everyone or nobody. No exclusions, no service accounts, no trusted locations. Fine for a small organisation, unworkable for a large one.

### Conditional Access — The Paid If/Then Engine 💳

**Conditional Access requires P1, so we won't build one — but you must know what it is, because it's asked in every identity interview.**

The model is a sentence: **IF** these conditions are true, **THEN** apply these controls.

- **IF** — which users and groups (and critically, who's *excluded*), which apps, and conditions: device platform, location, client app, device compliance state, sign-in risk (P2)
- **THEN** — block access, or grant it with requirements: require MFA, require a specific **authentication strength** (e.g. *phishing-resistant only* — this is how you mandate passkeys), require a compliant device, limit session length

Policies you'd find in essentially every mature tenant: require MFA for administrators, block legacy authentication, require MFA for Azure management, require a compliant device for corporate apps, block sign-ins from countries where you have no staff.

Two rules that prevent disaster, worth knowing even though we're not clicking:

1. **Report-only mode.** Every new policy starts in report-only, which logs what *would* have happened without enforcing it. Run it for days, confirm you haven't caught the backup service account, then enable.
2. **Break-glass accounts.** Two cloud-only Global Admin accounts, long random passwords, belonging to no individual, **excluded from every policy**, stored offline. For the day a policy misfires and the last admin's phone dies in another timezone.

!!! tip "Conditional Access vs RBAC — the distinction interviewers probe"
    **Conditional Access decides whether you get a token at all. RBAC decides what that token lets you do.** A CA policy cannot say "Priya may restart VMs" — it has no concept of a VM. An RBAC assignment cannot say "only from a corporate laptop" — it has no concept of a device. Real security uses both.

    If a question mentions *devices, locations, risk or MFA requirements* → **Conditional Access**. If it mentions *resources, actions or scope* → **RBAC**.

!!! note "Security defaults and Conditional Access are mutually exclusive"
    You cannot run both. Enabling a CA policy requires turning security defaults off. That's a reliable exam question — and if you ever do turn security defaults off, replace them the same hour.

### Mandatory MFA for Azure — Already Here

Independently of all of the above, Microsoft has made **MFA mandatory across all tenants** for Azure administration: first for the **Azure portal** and admin centers, and then at the **Azure Resource Manager layer** — meaning **Azure CLI, PowerShell, the REST APIs, Terraform and Bicep**.

That matters for the rest of this course. When we reach Bicep on Day 25 and Terraform on Day 26, those deployments authenticate through ARM, and **that authentication requires MFA**. This isn't opt-in. It's the floor now, for everybody.

### Self-Service Password Reset (SSPR)

Password resets are the single largest category of helpdesk tickets in most organisations. SSPR removes them — the user proves identity with registered methods and resets their own password at `aka.ms/sspr`. **Free for cloud-only users.** (Writing the new password back to on-premises AD needs P1.)

### Hands-On: Security Defaults, MFA and SSPR ✅

1. **Entra ID → Overview → Properties → Manage security defaults**. Read the current state and read the bullet list on the panel — this is the free security baseline for the entire Microsoft cloud. **Leave it Enabled.** It's doing real work. **✅**
2. In your normal browser, go to **`aka.ms/mfasetup`** → **+ Add sign-in method** → **Microsoft Authenticator** and follow the QR-code flow. **✅**
3. Complete the test prompt and note the **number matching** screen — the browser shows a two-digit number and you type it into the app. **✅**
4. If your phone or laptop supports it, also add a **Passkey**. That's the method Microsoft is standardising on. **✅**
5. **Entra ID → Password reset → Properties**: set SSPR to **Selected** and choose `grp-finance-team`. That's groups doing their job — a policy targeted at a group instead of a list of names. **✅**
6. Under **Authentication methods**, set methods required to **1** for the demo (**2** is the correct production answer). Under **Registration**, require users to register at next sign-in. **Save**. **✅**

---

## Part 5 — Applications: App Registrations and Service Principals

Everything so far has been about humans. But most authentication in a modern system isn't human — it's a web app calling an API, a script reading a storage account, a pipeline deploying infrastructure at 2am with nobody watching. Those need identities too.

And here's why this sits in the identity half but pays off in the authorisation half: **a service principal is a security principal.** Everything you learn about RBAC in Part Two applies to applications exactly as it applies to people. Azure genuinely does not care whether the thing holding a token has a face.

### Three Terms People Mix Up

- **App registration** — the **global definition** of your application: name, redirect URIs, permissions, credentials. There's exactly **one**, in the tenant where the app was created. The blueprint.
- **Service principal** — the **local instance** of that application in a specific tenant. **It's what actually gets role assignments.** Registering an app in your own tenant creates both automatically.
- **Enterprise application** — what the portal calls a service principal in the UI. Same object, different blade: **App registrations** is the developer view, **Enterprise applications** is the admin view.

One blueprint, many buildings — one per tenant. That's the analogy that sticks.

### Three Ways an App Proves Who It Is

| Credential | Reality |
|---|---|
| **Client secret** | A string. Expires. Ends up in config files, source control and Slack messages. Weakest. |
| **Certificate** | Private key — harder to leak by accident, but still something you store and rotate. |
| **Federated credential** | **No secret exists at all.** An external issuer (GitHub Actions, Kubernetes) presents its own token and Entra ID exchanges it. The modern answer for CI/CD — we'll use it on Day 22. |

### Delegated vs Application Permissions

When an app requests access to an API like Microsoft Graph, there are two fundamentally different kinds of permission, and the distinction gets asked about:

- **Delegated** — the app acts **on behalf of a signed-in user**. Effective access is the *intersection* of what the app was granted and what the user can do. Delegated `Files.Read` reads the files **that user** can read.
- **Application** — the app acts **as itself**, no user present. Application `Files.Read.All` reads **every file in the tenant**. There's no user to constrain it, so it always requires **admin consent**.

An over-permissioned application permission is one of the most common serious findings in a real Entra tenant.

!!! note "Graph permissions are not Azure RBAC"
    Same two-planes idea wearing a different hat. **Graph permissions** control access to *directory data* — users, groups, mail. **Azure RBAC roles** control access to *Azure resources*. An app can have `Directory.ReadWrite.All` and still not read a single blob.

### Hands-On: Register an Application ✅

1. **Entra ID → App registrations → + New registration**. **Name:** `app-lwm-demo`, **Supported account types:** *Single tenant*. Leave the redirect URI blank. **Register**. **✅**
2. On **Overview**, note the **Application (client) ID** and **Directory (tenant) ID**. Click **Managed application in local directory** — you land in **Enterprise applications**, looking at the service principal. Same app, other view. **✅**
3. **Certificates & secrets → + New client secret** → description `demo-secret`, 3 months → **Add**. **✅**
4. **Copy the Value immediately.** Not the Secret ID — the **Value**. It's shown exactly once. Everyone learns this the hard way. **✅**
5. Click the **Federated credentials** tab and read the scenario dropdown — *GitHub Actions deploying Azure resources*, *Kubernetes*, *Managed identity*. Bookmark this screen mentally for Day 22. **✅**

### Hands-On: Sign In as the Application — and Watch It Fail Like Priya ✅

A rare, justified use of the CLI, because there's no portal equivalent for "authenticate as an application."

1. Open **Cloud Shell** (the `>_` icon), Bash. **✅**
2. ```bash
   az login --service-principal \
     --username <application-client-id> \
     --password <the-secret-value> \
     --tenant <your-tenant-id>
   ```
   It succeeds — Entra ID just authenticated a non-human identity. **✅**
3. Now try to do something:
   ```bash
   az group list
   ```
   **The list is empty.** Exactly like Priya. **Authenticated, unauthorised.** The app has a valid identity and **zero** Azure RBAC. **✅**
4. Return to your own identity with `az login`. **✅**

!!! warning "You just handled a real credential"
    That secret is a password to your tenant, and it's now in your Cloud Shell history. In production it belongs in **Key Vault** (Day 18) — or better, it shouldn't exist at all, which is exactly what the next part is about. Delete this app registration at cleanup.

---

## Part 6 — Managed Identities

### The Problem, Stated Plainly

You just created a client secret. Now answer honestly: where does it live in production? Who could read it? What happens in three months when it expires — is there a calendar reminder, or does the app just break at 2am? Is it in your git history right now?

Credentials in configuration are the single most common cause of cloud compromise. **Managed identities delete the entire problem.**

### What It Is

A managed identity is a **service principal that Azure creates and manages for an Azure resource**, where the credentials are handled entirely by the platform, rotated automatically, and **never exposed to you or anyone else**.

There's no secret to copy, nothing to put in a config file, nothing to leak, nothing to rotate. And they cost **nothing**.

How it works, because it makes the idea concrete:

1. You enable a managed identity on a resource — VM, App Service, Function App, AKS, Data Factory.
2. Azure creates a service principal in your tenant, tied to that resource.
3. The platform injects the credential where **only that resource** can reach it — on a VM, that's the Instance Metadata Service at `169.254.169.254`.
4. Code inside the resource calls that local endpoint and receives an **Entra ID access token**.
5. It presents that token to the target service — Storage, SQL, Key Vault.
6. The target service validates the token and **checks the identity's RBAC permissions**.

No password crosses the wire. No password exists in your code.

Note step 6 carefully. **A managed identity solves authentication completely and authorisation not at all.** A brand-new managed identity, like Priya, can do nothing until you give it a role. We do that in Part 11.

### System-Assigned vs User-Assigned

| | **System-assigned** | **User-assigned** |
|---|---|---|
| **Created** | On the resource, as part of it | As a standalone Azure resource |
| **Lifecycle** | **Dies with the resource** | **Independent** — you delete it yourself |
| **Shared?** | No — exactly one resource | **Yes** — attach to many |
| **Best for** | One workload on one resource | Fleets, or permissions that must survive the compute |

The interview answer: if the identity must **survive VM recreation or be shared**, it's **user-assigned**. If it should be **cleaned up automatically with the resource**, it's **system-assigned**.

### Hands-On: Give an App Service an Identity ✅

1. **App Services → + Create → Web App**. **Resource group:** `rg-day17-demo`, **Name:** `app-lwm-day17-<yourname>`, **Publish:** Code, **Runtime:** any, **OS:** Linux, **Pricing plan: Free F1**. **Review + create → Create**. **✅**
2. Open it → **Settings → Identity → System assigned** → **Status: On** → **Save → Yes**. **✅**
3. An **Object (principal) ID** appears. Entra ID just created a service principal named exactly after your app. **✅**
4. Note what's **missing** from this blade: no credentials tab, no secret to copy, nothing to download. **That absence is the entire product.** **✅**
5. Click **Azure role assignments** — empty. Same position Priya was in. Part 11 fills it. **✅**

!!! tip "The rule to take away"
    **If both ends are Azure services, use a managed identity.** App Service to SQL, Function to Storage, VM to Key Vault, AKS to Container Registry — managed identity, every time. Reach for a client secret only when one end genuinely isn't an Azure resource.

---

## Part 7 — Guests and Hybrid Identity

Two short topics that come up constantly in interviews and almost never need clicking.

### B2B Guests

You hire a contractor. An auditor needs read access for six weeks. Do you create them an account in your tenant, with a password you now manage?

No. **B2B collaboration** lets you invite someone using **their own existing identity** — their work account in their own tenant, or a personal Microsoft account. They authenticate at home, under their own organisation's MFA policies. You just grant access.

- You never manage their password.
- Their employer disables their account when they leave, and **their access to your resources dies with it automatically.**
- A guest object appears in your tenant with a `#EXT#` in the UPN, and **guests can hold Azure RBAC role assignments exactly like members** — a guest can absolutely be a Reader on your subscription.

### Hybrid Identity

Most enterprises have twenty years of on-premises Active Directory and aren't throwing it away. Hybrid identity connects the two so a user has **one identity and one password** across both, using **Microsoft Entra Connect Sync** (the mature agent) or **Entra Cloud Sync** (lighter, cloud-configured, Microsoft's direction of travel).

The three authentication methods are a reliable exam and interview question:

| Method | Where the password is verified | Key characteristic |
|---|---|---|
| **Password Hash Sync (PHS)** | **In the cloud** | Simplest and most resilient. **If on-premises is offline, cloud sign-in still works.** Microsoft's recommended default. |
| **Pass-Through Auth (PTA)** | **On-premises**, via an agent | No password material in the cloud. If on-prem is down, sign-in fails. |
| **Federation (AD FS)** | **On-premises**, by a dedicated farm | Most control, most infrastructure. Microsoft encourages moving off it. |

The memory trick: **PHS = cloud verifies. PTA = on-prem verifies, no farm. Federation = on-prem verifies, with a farm.** When a question says *"must keep working if the on-premises environment is unavailable,"* the answer is **PHS**.

And note: a synced user is a normal security principal with a normal object ID. **You assign Azure roles to them identically.**

---

# PART TWO — AUTHORISATION

> **Halfway point.** Everything above answered *"who are you?"* Everything below answers *"and what exactly are you allowed to do?"* Right now you have a user who can sign in and see nothing, a group with no permissions, an application with a token and no access, and a managed identity that can't touch a single resource. Four identities, four empty permission sets. Let's fix them.

---

## Part 8 — Entra Roles vs Azure Roles

Before we build anything, we clear up the most misunderstood topic in Azure. There are **two completely separate role systems**, and people mix them up in interviews, in exams and in production incidents.

| | **Entra ID roles** | **Azure RBAC roles** |
|---|---|---|
| **Control** | The identity plane | The resource plane |
| **Examples** | Global Administrator, User Administrator, Application Administrator | Owner, Contributor, Reader, Storage Blob Data Reader |
| **Let you** | Create users, reset passwords, register apps | Create VMs, read blobs, delete resource groups |
| **Assigned at** | Tenant scope | Management group / subscription / resource group / resource |
| **Managed in** | Entra ID → Roles & admins | Any resource → Access control (IAM) |
| **Custom roles cost** | **P1 required** | **Free** |

**A Global Administrator has total control over identity and zero access to Azure resources.** They can create a hundred users, reset every password, delete the entire directory — and still not see a single virtual machine. Conversely, a subscription **Owner** can do anything to every resource and cannot create a user.

### The Bridge: The Elevate Access Toggle

There is exactly one connection between the two systems, and it's a switch: **Entra ID → Overview → Properties → Access management for Azure resources.**

Flip it to **Yes** and your Global Administrator identity is granted **User Access Administrator** at the **root management group** — across every subscription in the tenant.

This is the emergency door, for exactly one scenario: someone left, they were the only Owner on a subscription, and nobody can get in. A Global Admin elevates, grants themselves Owner, fixes it, and **switches the toggle back off**. Leaving it on permanently means every Global Admin silently holds access-admin rights over every subscription in your tenant.

### Hands-On: Prove the Split ✅

1. **Entra ID → Roles & admins → All roles**. Scan the list — over a hundred built-in Entra roles. **✅**
2. Open **Global Reader** and read the description: everything Global Admin can *see*, nothing it can *change*. **This is the single most underused role in Azure** — auditors and new team members should get Global Reader, not Global Administrator. **✅**
3. Open **User Administrator → + Add assignments** → select `Priya Sharma` → **Add**. **✅**
4. In the private window, sign Priya out and back in. Go to **Entra ID → Users**. **She can see the user list and create users.** Now go to **Subscriptions**. **✅**

   **Still empty.** She can create a hundred users and cannot see one resource group. That's the two planes demonstrated in a single browser session, and it's the clearest thirty seconds in this video.

5. Back as admin, open **Entra ID → Overview → Properties**, find **Access management for Azure resources**, read it, and **leave it set to No**. **✅**

!!! tip "The interview giveaway"
    Creating users, resetting passwords, registering apps, configuring Conditional Access → **Entra role**. Creating, reading or deleting Azure resources → **Azure RBAC role**. "Manage users *and* virtual machines" → **two separate assignments**.

---

## Part 9 — Azure RBAC: Who, What, Where

### The Whole Model, In One Line

Azure RBAC is the authorisation system built into Azure Resource Manager, and every access decision it makes comes from one construct: the **role assignment**. A role assignment is always, without exception, three things:

> **WHO** can do **WHAT**, **WHERE**.

- **WHO** — the **security principal**: a **user**, a **group**, a **service principal**, or a **managed identity**. Four types, complete list. Notice every one of them is something you created in Part One.
- **WHAT** — the **role definition**: a named collection of permissions. *Reader*, *Contributor*, *Storage Blob Data Reader*, or one you write.
- **WHERE** — the **scope**: management group, subscription, resource group, or an individual resource.

Bolt those three together and you've granted access. Delete the assignment and you've revoked it. There is no other mechanism. Every permission in Azure — yours included — exists because somewhere there's a row saying *this principal, this role, this scope*.

Say it out loud a few times. It's the answer to roughly a third of the RBAC questions you'll ever be asked: **security principal, role definition, scope.**

### Scope and Inheritance

```text
Root management group
   └── Management group        (e.g. "Production")
          └── Subscription     (e.g. "LWM-Prod")
                 └── Resource group  (e.g. "rg-day17-demo")
                        └── Resource (e.g. one storage account)
```

Assign a role at any level and it applies to that level **and everything beneath it**. **Nothing flows upward.**

- **Reader at management group scope** → read everything in every subscription underneath.
- **Contributor at resource group scope** → full control inside that one resource group, invisible everywhere else.

A **management group** is a container for subscriptions. It exists for exactly this: applying one role assignment or one policy across many subscriptions at once.

### Two Rules That Follow From This

**The golden rule:** *assign the least privileged role, at the narrowest scope, to a group rather than a person.* Microsoft adds a hard number — **a maximum of three subscription Owners**.

**RBAC is additive.** Your effective permissions are the **union** of every assignment that applies to you, from every scope, including ones inherited through group membership. There is no "deny" in a role assignment: if you have Reader at subscription scope and Contributor on one resource group, you're a Contributor there. **Adding a more restrictive role never takes anything away.**

!!! note "Why your new permission 'didn't work'"
    When you sign in, your token is issued **with your group memberships baked into it**. So adding someone to a group — or granting a new role — doesn't take effect until they get a **new token**. Sign out and back in. That single fact explains most "I granted access and nothing happened" complaints. Role assignment changes can also take **up to 10 minutes** to propagate.

### Hands-On: The Payoff — Give Priya Access ✅

Priya has been staring at an empty portal for an hour. Let's fix it.

1. **Resource groups → `rg-day17-demo` → Access control (IAM)**. Spend a moment here — this is the blade where you'll spend the rest of your Azure career: **✅**
   - **Check access** — "what can this specific person do here?"
   - **Role assignments** — everything granted at this scope, plus everything inherited
   - **Roles** — the full catalogue of role definitions
2. Look at **Role assignments** as it stands. Your own account is **Owner**, with scope shown as **(Inherited)** from the subscription. **That word is the hierarchy made visible.** **✅**
3. **+ Add → Add role assignment**, and walk the wizard tab by tab: **✅**
   - **Role:** search `Reader`. Note the two categories — **Job function roles** and **Privileged administrator roles** (Part 10). **Next**.
   - **Members:** select **`grp-finance-team`** — **the group, not Priya**. Golden rule, on camera.
   - **Review + assign** — read the summary aloud: *Reader · grp-finance-team · rg-day17-demo.* Who, what, where.
4. In the private window, **sign Priya out and back in** — she needs a fresh token. Go to **Resource groups**. **✅**

**`rg-day17-demo` is there.**

She can open it, see the App Service, read its configuration. That empty portal from Part 2 is populated — because of exactly one row in a table somewhere.

5. Now show the limits. As Priya: try to **stop** the App Service — greyed out or errors, because Reader reads. Check **Resource groups** again and confirm she sees **only** this one. That's scope. **✅**
6. Back as admin, open the **App Service → Access control (IAM) → Role assignments**. The Reader assignment appears here too, marked **(Inherited)**. **You never assigned anything on the App Service. It flowed down.** **✅**
7. Click **Check access**, search `Priya Sharma`, and select her. Azure shows exactly what she has and where it came from. **This is the blade to open when someone says "I can't access X" — it answers the question in five seconds.** **✅**

!!! tip "Say the sentence every time"
    Narrate every assignment: *"Reader, to grp-finance-team, at rg-day17-demo."* Do that consistently and you'll never write a sloppy one, because saying *"Owner, to Priya, at subscription"* out loud makes the problem audible.

---

## Part 10 — Inside a Role Definition, and the Built-in Roles

### A Role Is Just a List of Strings

A role definition looks impressive in the portal. Underneath it's JSON with five interesting properties:

```json
{
  "Name": "Reader",
  "Actions":         [ "*/read" ],
  "NotActions":      [],
  "DataActions":     [],
  "NotDataActions":  [],
  "AssignableScopes":[ "/" ]
}
```

**`Actions`** — control-plane operations, always `{Provider}/{resourceType}/{operation}`:

```text
Microsoft.Compute/virtualMachines/read           ← view a VM
Microsoft.Compute/virtualMachines/write          ← create or modify a VM
Microsoft.Compute/virtualMachines/start/action   ← start a VM
```

**`NotActions`** — a **subtraction, not a deny**. This is the most misread property in Azure. It removes items from the `Actions` list; it does **not** block anything granted by a different role assignment. **NotActions is arithmetic.**

**`DataActions` / `NotDataActions`** — the same idea for operations on the *data inside* a resource. That split is Part 11, and it's the most surprising thing in Azure RBAC.

**`AssignableScopes`** — where this definition is *available to be assigned*. Built-in roles use `/`, so they're assignable everywhere.

The arithmetic ARM performs is exactly:

```text
Actions      −  NotActions      =  effective management permissions
DataActions  −  NotDataActions  =  effective data permissions
```

### The Big Four

| Role | Read resources | Create/modify/delete | Assign roles to others |
|---|---|---|---|
| **Reader** | ✅ | ❌ | ❌ |
| **Contributor** | ✅ | ✅ | ❌ |
| **Owner** | ✅ | ✅ | ✅ |
| **User Access Administrator** | ✅ | ❌ | ✅ |

That last row surprises people: User Access Administrator can hand out Owner to anybody and cannot create a VM. It's a pure access-management role.

Azure sorts roles into **job function roles** (Virtual Machine Contributor, Storage Blob Data Reader, Website Contributor — 900+ of them, narrow and boring, and what you should be assigning almost all the time) and **privileged administrator roles** (Owner, Contributor, User Access Administrator, **Role Based Access Control Administrator**, Reservations Administrator). What puts a role in that second bucket isn't "can do a lot" — it's **"can grant permissions to others."** That's the master key.

!!! tip "Two habits worth forming"
    **Prefer a job function role.** Someone needs to restart VMs? *Virtual Machine Contributor*, not Contributor. Manage web apps? *Website Contributor*.

    **Prefer Role Based Access Control Administrator over User Access Administrator.** Same job — assigning roles — with strictly fewer permissions. Most tutorials haven't caught up with this one, and mentioning it in an interview lands well.

### Hands-On: Read a Built-in Role ✅

1. **Subscription → Access control (IAM) → Roles** tab. You're looking at the full catalogue. **✅**
2. Find **Contributor** → **…** → **View** → **JSON** tab. Read the `NotActions` array aloud: **✅**
   ```text
   Microsoft.Authorization/*/Delete
   Microsoft.Authorization/*/Write
   Microsoft.Authorization/elevateAccess/Action
   ```
   `Actions` is `["*"]`. **Everything.** Minus those subtractions. **Contributor is literally "Owner minus the ability to grant access."** Once you've seen the JSON you'll never forget the difference.
3. Open **Reader** → JSON. `Actions: ["*/read"]`. One line. **✅**
4. Open **Storage Blob Data Reader** → JSON and look at *where* the permission lives: **✅**
   ```json
   "dataActions": [
     "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
   ]
   ```
   **The interesting permission is in `dataActions`, not `actions`.** Remember that shape — the next part is built on it.
5. Search the Roles list for `Key Vault`, `SQL`, `Kubernetes`. There are **900+ built-in roles** and Microsoft adds more constantly. **Before you write a custom role, search properly. It probably already exists.** **✅**

---

## Part 11 — Control Plane vs Data Plane

### The Question Nobody Gets Right

Here's the question: **if I make you a Contributor on a storage account — full management rights, you can delete the entire account — can you read a file inside it?**

Everybody says yes. The answer is **no**.

Azure splits operations into two categories:

- **Control plane** — operations on the *resource itself*. Create the storage account, change replication, delete it. These go through **Azure Resource Manager**. Governed by **`Actions`**.
- **Data plane** — operations on the *data inside*. Read a blob, get a secret from Key Vault, query a Cosmos container. These go **straight to the service's own endpoint** — `yourstorage.blob.core.windows.net` — never touching ARM. Governed by **`DataActions`**.

**Contributor's `Actions` is `["*"]` and its `DataActions` is empty.** So a Contributor can perform every management operation on a storage account and has literally zero data permissions. **They can delete the account containing the file. They cannot open the file.**

That's not a quirk. It's deliberate, it's the foundation of separation of duties in Azure, and it's why separate roles like *Storage Blob Data Reader*, *Key Vault Secrets User* and *AcrPull* exist.

### Hands-On: Prove It ✅

**Set up:**

1. **Storage accounts → + Create**. **Resource group:** `rg-day17-demo`, **Name:** `stlwmday17<yourname>` (lowercase, unique), **Redundancy:** LRS. **Create**. **✅**
2. Open it → **Containers → + Container** → `reports` → **Create**. Open it → **Upload** → any small text file named `q3-results.txt`. **✅**

**Make Priya a Contributor on it:**

3. **Storage account → Access control (IAM) → + Add → Add role assignment**. **Role:** `Contributor`. **Members:** `grp-finance-team`. **Review + assign**. **✅**

**Watch it fail:**

4. In the private window, sign Priya out and back in, and navigate to the storage account. **✅**
5. Confirm she has real power: she can open **Configuration**, change the **access tier**, and the **Delete** button on the account is live. **She could destroy this storage account right now.** **✅**
6. Now click **Containers → reports**. **✅**

**She gets an error** — *"You do not have permissions to list the data using your user account with Microsoft Entra ID."*

Sit in that for a second. **She can delete the container. She cannot list what's in it.** Control plane, data plane.

**Fix it properly:**

7. Back as admin: **storage account → IAM → + Add → Add role assignment → `Storage Blob Data Reader`** → **Members:** `grp-finance-team` → **Review + assign**. **✅**
8. In the private window, refresh — give it a minute, propagation can take up to 10. Click into `reports`. **✅**

**The blob list appears.** She can open `q3-results.txt` and read it. She still can't upload or delete — that would need *Storage Blob Data Contributor*.

!!! warning "The back door worth knowing about"
    A Contributor *can* call `listKeys` — a **control-plane** action — and use the returned account key to reach the data anyway. That's why serious environments disable shared key access entirely and force all access through Entra ID and RBAC. Data-plane roles only give you real separation of duties once that back door is closed.

### Closing the Loop: Give the Managed Identity a Role ✅

Everything you just did for Priya applies identically to software. That App Service identity from Part 6 is a **security principal**. It can hold role assignments. Azure does not care that it has no face.

1. **Storage account → Access control (IAM) → + Add → Add role assignment → `Storage Blob Data Reader`**. **✅**
2. On the **Members** tab, change **Assign access to** from *User, group, or service principal* to **Managed identity**. **✅**
3. **+ Select members** → **Managed identity: App Service** → pick `app-lwm-day17-<yourname>`. **Review + assign**. **✅**

   Note what you did **not** do: you didn't paste a client ID, and no secret was involved anywhere.

4. Look at the storage account's **Role assignments** tab now: a group with Contributor, the same group with Storage Blob Data Reader, and an **App Service** with Storage Blob Data Reader. **Humans and software, same list, same model.** **✅**

Your App Service can now read that blob — with no username, no password, no key, no connection string anywhere on it. **Authentication with no credential, authorisation with no credential.** That's the whole point of Part 6 and Part 9 arriving in the same video.

!!! tip "The interview version of this part"
    *"A user has Contributor on a storage account but can't read blobs. What's the minimum change?"* → **Assign a data-plane role such as Storage Blob Data Reader.** Not Owner. Not Storage Account Contributor. The question is testing whether you know the two planes exist. Same pattern everywhere: **Key Vault Secrets User** vs Contributor on the vault (Day 18), **AcrPull** vs Contributor on a registry.

---

## Part 12 — Custom Roles

With 900+ built-in roles, your first move is always to search the catalogue. But sometimes nothing fits:

- Too broad — *Virtual Machine Contributor* lets someone delete VMs when you only want them to restart a hung one.
- Too narrow — you need permissions spanning two services and no single role covers both.

Custom roles are **free**, work with any principal type, and behave identically to built-in roles. (Reminder: custom **Entra** roles need P1. Custom **Azure RBAC** roles are free. Different systems.)

Three starting points: **clone a role** (usually right), **start from scratch**, or **start from JSON** (which is how you keep roles in source control — the grown-up answer).

### Hands-On: Build "VM Operator" ✅

The scenario: your team needs to restart misbehaving VMs at 3am. They must **not** be able to create, resize or delete them.

1. **Subscription → Access control (IAM) → + Add → Add custom role**. **✅**
2. **Basics:** **Name:** `LWM VM Operator`. **Baseline permissions:** *Clone a role* → **Virtual Machine Contributor**. **✅**
3. **Permissions tab** — cut it down: **✅**
   - Remove `Microsoft.Compute/virtualMachines/write` and `/delete`
   - Remove the disk and snapshot write/delete permissions
   - Confirm these survive:
     ```text
     Microsoft.Compute/virtualMachines/read
     Microsoft.Compute/virtualMachines/start/action
     Microsoft.Compute/virtualMachines/restart/action
     Microsoft.Compute/virtualMachines/powerOff/action
     ```
4. **Assignable scopes tab** — by default it lists your whole subscription. **Narrow it:** remove that and add `rg-day17-demo`. Now the role can only ever be assigned inside that resource group. **✅**
5. **JSON tab** — read the definition you just built. This is the payoff for Part 10. Copy it into a file if you want it in source control. **✅**
6. **Review + create → Create.** It takes a few minutes to propagate. **✅**
7. Assign it: **`rg-day17-demo` → IAM → + Add → Add role assignment**, use the **Type** filter → **CustomRole** to find it, and assign to `grp-finance-team`. **✅**

!!! warning "Don't reach for the wildcard"
    It's tempting to write `Microsoft.Compute/*` and be done. List actions explicitly instead. A wildcard silently grants every permission Microsoft adds to that provider **later** — that's how a "restart VMs" role quietly becomes something else eighteen months from now.

---

## Part 13 — Azure Policy vs RBAC

Students confuse these constantly, so one line separates them:

> **RBAC controls *who can perform actions*. Azure Policy controls *what resources are allowed to exist*.**

They're orthogonal and you need both. RBAC says "Priya may create VMs here." Policy says "and any VM created here must be in an approved region and carry a `CostCenter` tag." Priya has permission; the resource still gets rejected.

**Policy effects**, in rough order of aggression:

| Effect | What it does |
|---|---|
| **Audit** | Logs non-compliance, changes nothing. Always start here. |
| **Deny** | **Blocks the deployment outright** |
| **Modify** | Adds or updates tags/properties during creation |
| **DeployIfNotExists** | Auto-deploys a missing related resource — real remediation |

Policies group into **initiatives** (for example an entire ISO 27001 benchmark as one assignable unit) and are assigned at management group, subscription or resource group scope, inheriting downwards exactly like RBAC.

!!! note "Deny assignments — know the term, that's all"
    There's one thing that *can* override a role assignment: a **deny assignment**, which blocks actions for a principal and beats every role, even Owner. **You cannot create them by hand.** Azure creates them for managed applications and deployment stacks. The **Deny assignments** tab exists on every IAM blade and will be empty on your subscription. Know it exists; move on.

### Hands-On: A Policy That Blocks an Owner ✅

1. Search **Policy** → **Authoring → Definitions**. Find **`Require a tag on resources`** and open it. Read the JSON — an `if` block (the tag is missing) and a `then` block (`"effect": "deny"`). **✅**
2. **Assign** it: **Scope:** `rg-day17-demo`, **Parameters → Tag Name:** `CostCenter`. **Review + create → Create**. **✅**
3. Wait a few minutes — policy evaluation isn't instant. **✅**
4. Try to create any resource in `rg-day17-demo` **without** a `CostCenter` tag. **✅**

   **Rejected**, with `RequestDisallowedByPolicy` naming your assignment. You are the subscription **Owner**. You have every permission in Azure. **And it still failed** — because RBAC and Policy answer different questions.

5. Create the same resource **with** the tag. It succeeds. **✅**
6. **Delete the policy assignment** before moving on, or the cleanup steps will fight you. **✅**

!!! note "Where the two systems meet"
    `DeployIfNotExists` and `Modify` policies have to actually *do* something, so the policy assignment gets a **managed identity**, and that identity needs **role assignments** to perform the remediation. A governance rule, enforced by a managed identity, authorised by RBAC — every concept from today in one sentence.

---

## Part 14 — PIM, Troubleshooting and Interview Prep

### Privileged Identity Management 💳

**PIM requires Entra ID P2, so there's no demo — but the vocabulary comes up in interviews constantly, so know it cold.**

Think about who holds Owner or Global Administrator in a typical organisation, and ask how many hours a year they actually *need* it. Maybe twenty. They hold it for 8,760. Every hour of standing privilege is an hour a phished session can use.

**PIM shrinks that window.** Instead of holding the role, you're **eligible** for it, and you activate it when needed.

| Term | Meaning |
|---|---|
| **Eligible** | The assignment exists but is dormant — the user must **activate** it |
| **Active** | A normal, working assignment |
| **Time-bound** | Expires automatically on a date — eligible *or* active |
| **Activation** | Turning an eligible assignment on — can require MFA, a typed justification, and approval by a named person |

So the contractor's access expires by itself. The admin's Owner role lasts one hour, was approved by someone, and has a recorded reason. And everything lands in an audit trail.

Two bonus facts worth having: PIM covers **Entra roles, Azure resource roles, and group membership**; and **eligible assignments don't count towards the 4,000-role-assignment limit**, so PIM improves your security posture *and* your headroom.

The trigger words: *"just-in-time," "temporary elevation," "approval workflow," "time-bound admin access," "access must expire automatically"* → **PIM, and the licence is P2.**

### The Gotchas That Actually Waste Your Day

These four cause most real-world RBAC support tickets:

- **Propagation delay.** Role assignment changes take **up to 10 minutes**. And because group memberships are baked into the token, **sign out and back in** to see a new one. Start every "it didn't work" investigation here.
- **"Identity not found."** You deleted a user, group or service principal without removing its role assignments. The assignment survives pointing at a dead GUID — harmless but messy, and it counts towards your 4,000. Clean them up.
- **Role assignments don't move with resources.** Move a resource to another resource group and any assignment made *directly on it* is orphaned. Recreate it.
- **Subscription transfer wipes RBAC.** Move a subscription to a different tenant and **all** role assignments are permanently deleted. Worth saying twice, because people get bitten exactly once.

### Where the Evidence Lives

Two logs, two planes, both free:

- **Entra ID sign-in logs** — every authentication attempt, with the Conditional Access policies evaluated and the MFA method used. Four categories: interactive user, non-interactive user, **service principal**, and **managed identity** sign-ins. Your infrastructure signing in, logged like a person.
- **Azure Activity Log** — every control-plane operation on resources, **including role assignment writes and deletes**. This is where *"who gave Priya Contributor, and when?"* is answered.

**Retention is the catch:** free tenants keep Entra logs **7 days**, the Activity Log keeps 90. For anything real you export via **Diagnostic settings** to a Log Analytics workspace — that's **Day 19, Azure Monitor**, and it's the same pattern for every service in Azure.

### Hands-On: Audit What You Did Today ✅

1. **`rg-day17-demo` → Activity log** → filter **Operation: Create role assignment**. Every assignment you made today, with who made it and when. **This is the answer to "who granted that?"** **✅**
2. **Entra ID → Sign-in logs.** Find your own sign-in and work through the tabs. Find **Priya's**. Then switch the view to **Service principal sign-ins** and find `app-lwm-demo` from Part 5. **✅**
3. **Subscription → Access control (IAM) → Role assignments** and look at the count chart at the top of the page, showing where you stand against the 4,000 limit. **✅**

### Interview and Exam Quick Reference

| If you're asked… | The answer is… |
|---|---|
| "Create users / reset passwords / register apps" | **Entra ID role** (User Administrator) |
| "Permissions on Azure resources" | **Azure RBAC role** — not an Entra role |
| "Global Admin can't see any subscriptions" | **Elevate access** toggle — then turn it back off |
| "Membership must update automatically by department" | **Dynamic group** (P1) |
| "Require MFA only from outside the office" | **Conditional Access** with a named location (P1) |
| "Simplest way to require MFA for everyone, free" | **Security defaults** |
| "Must keep working if on-premises is unavailable" | **Password Hash Synchronisation** |
| "Give a partner access without creating an account" | **B2B guest invitation** |
| "Recover a user deleted last week" | **Deleted users** — 30-day restore, same object ID |
| "An Azure service must reach another with no credentials" | **Managed identity** + a role assignment |
| "Identity must survive VM recreation or be shared" | **User-assigned** managed identity |
| "A pipeline must deploy with no stored secret" | **Federated credential** (workload identity federation) |
| "Grant the minimum access to manage VMs" | **Virtual Machine Contributor** at the narrowest scope |
| "Manage everything but not grant access to others" | **Contributor** |
| "Grant access to others but not manage resources" | **User Access Administrator** — or better, **RBAC Administrator** |
| "One assignment across many subscriptions" | **Management group** scope |
| "Contributor but can't read blobs" | Assign a **data-plane role** — Storage Blob Data Reader |
| "Built-in roles don't fit" | **Custom role** (free) |
| "All resources must carry a CostCenter tag" | **Azure Policy**, **Deny** effect |
| "Just-in-time, time-bound admin access with approval" | **PIM** (P2) |
| "We've hit 4,000 role assignments" | **Assign to groups**, remove redundant assignments, make them eligible in PIM |

---

## Cleanup ✅

Cleanup has two halves, because you created things in two planes. **Deleting a resource group removes nothing from the directory** — users, groups and app registrations live in the tenant.

**Resource plane:**

1. **Delete the policy assignment** if you haven't: **Policy → Assignments → Require a tag on resources → Delete**. **✅**
2. **Delete the custom role:** `rg-day17-demo` → IAM → **Roles** tab → filter **CustomRole** → `LWM VM Operator` → **Delete**. **Remove its role assignment first**, or you'll get `RoleDefinitionHasAssignments`. **✅**
3. **Delete the resource group** `rg-day17-demo`. This removes the App Service, the plan, the storage account, and automatically every role assignment scoped inside it. **✅**
4. **Check the subscription IAM blade** for orphaned assignments showing *Identity not found* and remove them. **✅**

**Identity plane:**

5. **Delete the app registration:** **Entra ID → App registrations → app-lwm-demo → Delete**. This kills the client secret. Do it. **✅**
6. **Remove the Entra role** you gave Priya in Part 8 (**Roles & admins → User Administrator → Assignments**). **✅**
7. **Keep `Priya Sharma` and `grp-finance-team`.** They cost nothing, and Day 18 (Key Vault) uses exactly this pattern again — a group, a data-plane role, a managed identity. **✅**
8. **Keep `db-lwm-demo`** — it's on the free offer and Day 18 and Day 30 both use it. **✅**
9. **Keep the MFA you registered on your own account.** Genuinely. It's free, it's the highest-value security change you can make to your Azure account, and Microsoft mandates it for the portal and ARM anyway. **✅**

---

## Summary

Today you stopped asking "how do I build it?" and started asking two harder questions: **"who are you?"** and **"what exactly are you allowed to do?"**

**Microsoft Entra ID is the identity plane** — a global, region-less service holding users, groups and application identities, completely separate from the resource plane where your VMs and databases live. Different containers, different role systems, different blades. The moment that split clicks, Azure permissions stop being confusing: **a Global Administrator can't see a virtual machine, and a subscription Owner can't create a user.** We proved it twice — once with Priya, once with a service principal that had a valid token and no access to an empty resource group.

**Users and groups** are the raw material, and the rule is permanent: **never assign permissions to a person.** It isn't tidiness — it's the documented remedy for the hard 4,000-assignment ceiling.

**Authentication is where the free value is.** MFA blocks over 99% of identity attacks and costs nothing. Security defaults deliver real protection with one switch. Microsoft has made MFA mandatory for the portal and the ARM layer — which means CLI, PowerShell, Terraform and Bicep. And **passkeys become the default from September 2026**, with SMS being retired, so don't design around text messages.

**Applications get identities too**, and the best idea in Azure security is the **managed identity** — a credential you cannot read, cannot leak and never rotate, because it doesn't exist anywhere you can reach.

Then everything flipped. **Azure RBAC is one sentence: who can do what, where.** Security principal, role definition, scope. Every permission in Azure is that sentence written down somewhere, and every access problem you'll ever debug is one of those three words being wrong.

You watched scope inherit downward, and you saw the exact instant Priya's empty portal filled up. You read the JSON and found that **Contributor is literally Owner minus the ability to grant access**. You made a Contributor on a storage account **who could delete the account and could not read a file inside it** — the single most important surprise in Azure authorisation. You gave that same data role to a machine. You built a custom role and cut it down to three actions and one resource group. And you watched Azure Policy reject a deployment from a subscription Owner because a tag was missing.

Two questions. Two systems. One video, because in real life you never get to answer just one of them.

### What's Next

**Day 18 — Azure Key Vault** is where secrets go to live properly, and it's the perfect sequel. Key Vault carries *both* permission models side by side — the legacy **access policy** model and modern **Azure RBAC** — with data-plane roles like *Key Vault Secrets User* that behave exactly like the Storage Blob Data Reader you met today. And you'll wire a managed identity to a vault so an application reads a secret with no credential, using precisely the pattern you built here.

---

## Key Takeaways

**Identity**

- **Two planes.** Entra ID manages identities (global, no region). ARM manages resources (regional). Different role systems, different blades — and mixing them up causes most Azure permission confusion.
- **One subscription trusts exactly one tenant.** Moving a subscription to another tenant **permanently deletes every role assignment**.
- **Entra ID is not AD DS.** OAuth/OIDC/SAML, flat directory, no Group Policy, no OUs.
- **The object ID is what matters.** RBAC stores GUIDs, not names — which is why a 30-day user restore brings access back and a re-created user doesn't.
- **Never assign permissions to a person.** Groups make access a membership question and are the documented fix for the 4,000-assignment ceiling.
- **MFA is free and blocks 99%+ of identity attacks.** Security defaults give you a baseline in one click; **Conditional Access (P1)** adds the if/then logic. You can't run both.
- **Passkeys become the default from 1 September 2026**; SMS and voice are being retired. Don't build on SMS.
- **App registration = blueprint, service principal = the instance that holds role assignments.** Enterprise applications is just the admin view of the same object.
- **Delegated permissions act as the user; application permissions act as the app** and need admin consent.
- **Managed identities are free, have no readable credential, and are the correct answer whenever both ends are Azure services.** System-assigned dies with the resource; user-assigned survives and can be shared.
- **PHS = cloud verifies, PTA = on-prem verifies, Federation = on-prem with a farm.** "Must work when on-prem is down" → **PHS**.

**Authorisation**

- **Every role assignment is three things: security principal + role definition + scope.** Who, what, where. There is no other mechanism.
- **Four principal types** — user, group, service principal, managed identity. Software is a first-class citizen.
- **Four scope levels** — management group → subscription → resource group → resource. Permissions inherit **downwards only**.
- **RBAC is additive.** Your access is the union of every assignment. Adding a narrower role never removes anything.
- **Golden rule:** least privileged role, narrowest scope, assigned to a group. **Maximum three subscription Owners.**
- **`Actions − NotActions` = your effective permissions. NotActions is subtraction, not denial.**
- **Contributor = Owner minus the ability to grant access.** One `NotActions` list, nothing more.
- **Control plane ≠ data plane.** A Contributor on a storage account **cannot read a blob** — that needs Storage Blob Data Reader. The same split hits Key Vault, Cosmos DB and ACR. Watch for the `listKeys` back door.
- **Prefer job function roles over privileged administrator roles**, and prefer **RBAC Administrator** over User Access Administrator.
- **Custom roles are free.** Clone, cut down, narrow the assignable scopes — and never use wildcards.
- **RBAC controls who can act; Azure Policy controls what may exist.** A subscription Owner can still be blocked by a Deny policy.
- **PIM (P2)** turns standing privilege into just-in-time, time-bound, approved, audited access — and eligible assignments don't count towards the 4,000 limit.
- **The ten-minute rule:** assignment changes take up to 10 minutes, and group changes need a fresh token. Sign out and back in before you debug anything else.
