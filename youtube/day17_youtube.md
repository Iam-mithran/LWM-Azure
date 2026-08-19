# Day 17 — YouTube Metadata

---

## Video Title

Microsoft Entra ID & Azure RBAC Explained for Beginners: Users, MFA, Managed Identities & Roles (Hands-On) | Day 17 | LearnWithMithran

---

## Thumbnail

**Main text (large, bold):** `Signed In ≠ Allowed In`
**Sub text:** `Day 17 — Entra ID + Azure RBAC`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- Split down the middle with a bright divider — left half labelled "WHO ARE YOU?", right half "WHAT CAN YOU DO?"
- Left: a user avatar with a green tick and a phone showing an MFA number-match prompt
- Right: an empty Azure portal window with a red padlock over it
- Small corner badge: "100% Free Tier"
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** Proving who you are and being allowed to do something are two completely different systems in Azure — which is why your brand-new user signs in successfully and still sees an empty portal.

---

## Description

*Welcome back to Learn With Mithran! For sixteen days there has been exactly one person allowed to touch anything you've built: you. Every click you made in the portal was authorised because of one identity — the account you signed up with on Day 1 — and you never thought about it, because it just worked. Today we open that box. Because the moment a second person joins your project, or an application needs to read your database, "it just works" stops being good enough.*

You'll learn what happens in the half-second between typing your password and seeing the Azure portal — **and what happens in the half-second after that**, when Azure decides whether you're actually allowed to do the thing you clicked. Two different questions, two different systems, and almost every confusing thing in Azure security comes from mixing them up. In Part One we build the identity side: tenants and directories, users and groups, MFA and number matching, security defaults, app registrations versus service principals, and managed identities. We create a user called Priya, sign in as her, and watch her land in a completely empty portal. In Part Two we fix that properly. Every role assignment in Azure is one sentence — **security principal + role definition + scope** — and once you can say it, RBAC stops being mysterious. You'll read the role JSON and see with your own eyes that **Contributor is literally Owner minus one thing**. You'll hit the most surprising fact in Azure: a **Contributor on a storage account who cannot read a blob**, then fix it in thirty seconds. You'll give that same permission to a machine, build a custom role from scratch, and watch Azure Policy reject a deployment from a subscription Owner.

✅ **Every hands-on step in this video is free tier — there are no paid demos.** The paid features that come up in interviews (Conditional Access, dynamic groups, PIM) are explained properly as concepts, so you know what they do and which licence they need, without being asked to spend anything. 🔐

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

🔹 The two planes — identity (Entra ID) vs resources (ARM) — the one idea that makes Azure permissions stop being confusing
🔹 Tenant, directory, subscription — and why moving a subscription to another tenant wipes every role assignment
🔹 Why Microsoft Entra ID is NOT "Active Directory in the cloud"
🔹 Users, UPNs, and why the object ID (not the name) is what RBAC actually stores
🔹 The 30-day deleted-user restore — and why re-creating a user does NOT restore their access
🔹 Demo — create a user, sign in as her in a second browser, and discover she can see absolutely nothing
🔹 Groups — security vs Microsoft 365, and why you never assign permissions to a person
🔹 The hard 4,000 role assignment limit per subscription, and why groups are the fix
🔹 MFA explained properly — factor types, method strength ranking, and number matching
🔹 **Passkeys become the default sign-in method from September 2026** — SMS and voice MFA are being retired
🔹 Security defaults — the free security baseline — and what it actually enforces
🔹 Conditional Access explained (P1) — the if/then engine, report-only mode, and break-glass accounts
🔹 Conditional Access vs RBAC — CA decides if you get a token, RBAC decides what the token can do
🔹 Mandatory MFA for the Azure portal and the ARM layer — CLI, PowerShell, Terraform and Bicep
🔹 Self-service password reset, free for cloud users
🔹 App registration vs service principal vs enterprise application — one object, three views
🔹 Client secrets vs certificates vs federated credentials
🔹 Delegated vs application permissions — and why application permissions need admin consent
🔹 Demo — register an app, create a secret, sign in with the CLI: authenticated but completely unauthorised
🔹 Managed identities — no readable credential, nothing to rotate, and completely free
🔹 System-assigned vs user-assigned — the interview answer for which to pick
🔹 B2B guests, and hybrid identity: PHS vs PTA vs Federation
🔹 **Entra roles vs Azure roles** — why a Global Administrator can't see a single virtual machine
🔹 Demo — Priya becomes a User Administrator and STILL sees no subscriptions
🔹 The elevate access toggle — the break-glass door between the two systems
🔹 The three-part sentence — security principal + role definition + scope = role assignment
🔹 Scope and inheritance — management group → subscription → resource group → resource
🔹 Demo — one role assignment fills Priya's empty portal, and Check access explains why
🔹 Role definition JSON — Actions, NotActions, DataActions, AssignableScopes
🔹 The arithmetic that proves **Contributor = Owner minus the ability to grant access**
🔹 Reader / Contributor / Owner / User Access Administrator, properly distinguished
🔹 **Control plane vs data plane** — the demo that surprises everyone: Contributor cannot read a blob
🔹 Demo — fix it with Storage Blob Data Reader, then give the same role to a managed identity
🔹 The listKeys back door, and why production storage disables shared key access
🔹 Demo — build a custom role from scratch and narrow its assignable scope
🔹 RBAC vs Azure Policy — who can act, versus what is allowed to exist
🔹 Demo — a Deny policy rejects a deployment from the subscription Owner
🔹 PIM explained (P2) — eligible vs active, time-bound access, activation with approval
🔹 The four gotchas that waste your afternoon — including the ten-minute propagation rule
🔹 Sign-in logs, activity logs, and where "who granted that?" is answered
🔹 A full interview and exam keyword table for both halves

📌 *Who Is This Video For:*

💻 Beginners who've built resources and now need to let other people and applications use them safely
🧑‍🎓 Students preparing for AZ-104 or AZ-305 — identity and access is one of the heaviest scored areas on both
☁️ Anyone who has ever created a user in Azure and been confused about why they can't see anything
🔥 Developers who still have connection strings and API keys sitting in config files
🖥️ Admins coming from on-premises Active Directory who need to know what transfers and what doesn't
🚀 Anyone preparing for cloud interviews — nearly every question in this video gets asked in real ones

🔍 *Chapters:*
0:00 Intro — Signing In and Being Allowed In Are Two Different Things
3:00 What's Free Today (Everything) and What We Only Talk About
6:00 Part 1 — What Entra ID Is, and the Two Planes
13:00 Tenant, Directory, Subscription — And Why Transfers Wipe RBAC
17:00 Entra ID Is Not Active Directory + Demo: Find Your Tenant
21:00 Part 2 — Users, UPNs, and the Object ID That RBAC Stores
26:00 Demo — Create Priya, Sign In as Her, and See Absolutely Nothing
31:00 Part 3 — Groups, and Why You Never Assign to a Person
37:00 Part 4 — MFA, Number Matching, and the Move to Passkeys
43:00 Security Defaults, and Conditional Access Explained (P1)
50:00 Demo — Security Defaults, Register MFA, Configure SSPR
55:00 Part 5 — App Registrations vs Service Principals
61:00 Demo — Sign In as an App With the CLI: Authenticated, Unauthorised
65:00 Part 6 — Managed Identities: No Credential to Leak
71:00 Demo — Give an App Service an Identity
74:00 Part 7 — Guests, and Hybrid Identity: PHS vs PTA vs Federation
78:00 Part 8 — Entra Roles vs Azure Roles + Demo: Priya the User Admin
84:00 Part 9 — The Three-Part Sentence: Principal + Role + Scope
89:00 Demo — One Role Assignment Fills Priya's Empty Portal
95:00 Part 10 — Inside a Role Definition, and the Contributor Arithmetic
101:00 Part 11 — Control Plane vs Data Plane: The Demo That Surprises Everyone
108:00 Demo — Fix It, Then Give the Same Role to a Managed Identity
112:00 Part 12 — Building a Custom Role
117:00 Part 13 — Azure Policy vs RBAC + Demo: Blocking an Owner
122:00 Part 14 — PIM, Troubleshooting, and Interview Prep
128:00 Cleanup, Summary and Key Takeaways

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #MicrosoftEntraID #AzureAD #AzureRBAC #RBAC #ManagedIdentity #ConditionalAccess #AzureMFA #Passkeys #PIM #AzurePolicy #CustomRoles #AzureSecurity #AzureForBeginners #MicrosoftAzure #AZ104 #AZ305 #LearnAzure #AzurePortal #AzureTutorial #LearnWithMithran #CloudComputing #GreensTechnologies #AzureTraining #AzureCertification #IdentityAndAccessManagement #FreeTier

---

## Tags

microsoft entra id tutorial, azure active directory tutorial, azure rbac explained, azure role based access control, entra id vs active directory, entra roles vs azure roles, azure managed identity, system assigned vs user assigned managed identity, azure app registration vs service principal, azure conditional access explained, security defaults azure, azure mfa number matching, passkeys azure 2026, self service password reset azure, azure custom role, role definition json, actions notactions dataactions, control plane vs data plane azure, storage blob data reader, contributor cannot read blob, azure policy deny effect, rbac vs azure policy, privileged identity management, pim azure explained, phs vs pta vs federation, azure role assignment limit 4000, check access azure, az-104, az-305, azure interview questions, azure for beginners, azure free tier, azure portal, azure tutorial, learnwithmithran, greens technologies, azure training, azure certification, microsoft azure
