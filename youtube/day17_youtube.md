# Day 17 — YouTube Metadata

---

## Video Title

Microsoft Entra ID & Azure RBAC Explained: Users, MFA, Managed Identities, Custom Roles & PIM (Hands-On) | Day 17 | LearnWithMithran

---

## Thumbnail

**Main text (large, bold):** `Signed In ≠ Allowed In`
**Sub text:** `Day 17 — Entra ID + Azure RBAC`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- Split down the middle with a bright divider line — left half labelled "WHO ARE YOU?", right half labelled "WHAT CAN YOU DO?"
- Left: a user avatar with a green tick and a phone showing an MFA number-match prompt
- Right: an empty Azure portal window with a red padlock over it
- A small key icon with a red strike-through, labelled "No password anywhere"
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** Proving who you are and being allowed to do something are two completely different systems in Azure — and mixing them up is why your new user signs in successfully and still sees an empty portal.

---

## Description

*Welcome back to Learn With Mithran! For sixteen days there has been exactly one person allowed to touch anything you've built: you. Every click you made in the portal was authorised because of one identity — the account you signed up with on Day 1 — and you never thought about it, because it just worked. Today we open that box, and we open it all the way. Because the moment a second person joins your project, or a second application needs to read your database, or a laptop gets stolen in an airport, "it just works" stops being good enough.*

This is the longest video in the course, and it's deliberately one video instead of two. You'll learn what happens in the half-second between typing your password and seeing the Azure portal — **and then what happens in the half-second after that**, when Azure decides whether you're actually allowed to do the thing you just clicked. Those are two different questions answered by two different systems, and almost every confusing thing in Azure security comes from mixing them up. In Part One we build the identity side: tenants and directories, users and groups, MFA and number matching, the **September 2026 shift to passkeys by default**, security defaults versus Conditional Access, app registrations versus service principals, and managed identities — finishing with a live **App Service connecting to a SQL database with no password existing anywhere in the world**. We create a user called Priya, sign in as her, and watch her land in a completely empty portal. In Part Two we fix that, properly. Every role assignment in Azure is one sentence — **security principal + role definition + scope** — and once you can say it, RBAC stops being mysterious. You'll read role definition JSON and see with your own eyes that **Contributor is literally Owner minus two permissions**. You'll hit the single most surprising thing in Azure: a **Contributor on a storage account who cannot read a blob**, and then fix it in thirty seconds. You'll watch a virtual machine read a file from storage with **no credential on the box at all**, then remove one role assignment and watch it break. You'll build a custom role from scratch, use an **ABAC condition** to lock a role down to a single container, set up **constrained delegation** so an admin can hand out exactly one role to exactly one group and nothing else, block a subscription Owner with Azure Policy, and hand out just-in-time admin rights with **PIM**. Almost all of it is genuinely free — Azure RBAC in its entirety, including custom roles and conditions, costs nothing. 🔐

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

🔹 The identity plane vs the resource plane — the distinction that explains almost every confusing thing in Azure
🔹 Tenant, directory, subscription — how the three fit together, and why moving a subscription wipes every role assignment
🔹 Why Microsoft Entra ID is NOT "Active Directory in the cloud" — different protocols, flat vs hierarchical, no OUs or Group Policy
🔹 Entra ID editions — Free, P1, P2, ID Governance, Entra Suite — and exactly where the paywalls sit
🔹 Users — member vs guest, cloud-only vs synced, UPNs, custom domains, and the 30-day undo button
🔹 Demo — create a user, sign in as her in a second browser, and discover she can see absolutely nothing
🔹 Groups — security vs Microsoft 365, assigned vs dynamic membership, nesting and transitivity
🔹 Why you should never assign permissions to a person, only to a group
🔹 MFA explained properly — factor types, method strength ranking, and number matching
🔹 The Authentication methods policy, Temporary Access Pass, and SSPR
🔹 **Passkeys become the default sign-in method from September 2026** — and SMS/voice MFA is on the way out
🔹 Mandatory MFA for Azure — Phase 1 (portal) and Phase 2 (ARM, CLI, PowerShell, Terraform, Bicep)
🔹 Security defaults — six free protections — vs Conditional Access, and why you can't have both
🔹 Demo — build a Conditional Access policy in report-only mode without locking yourself out
🔹 App registrations vs service principals vs enterprise applications — one object, three views
🔹 Client secrets vs certificates vs federated credentials — and why workload identity federation wins
🔹 Demo — register an app, create a secret, sign in with the CLI: authenticated but completely unauthorised
🔹 Managed identities — system-assigned vs user-assigned, and the IMDS endpoint at 169.254.169.254
🔹 Demo — App Service → Azure SQL with a managed identity and NO password anywhere in the world
🔹 B2B guests, cross-tenant access settings, and the retirement of Azure AD B2C
🔹 Devices — registered, joined, hybrid joined — and hybrid identity: PHS vs PTA vs Federation
🔹 **The bridge:** Entra roles vs Azure roles — why a Global Administrator can't touch a single resource
🔹 The three-part sentence — security principal + role definition + scope = role assignment
🔹 Scope and inheritance — management group → subscription → resource group → resource
🔹 Demo — one role assignment fills Priya's empty portal
🔹 Role definition JSON — Actions, NotActions, DataActions, NotDataActions, AssignableScopes
🔹 The arithmetic that proves **Contributor = Owner minus two permissions**
🔹 Job function roles vs the five privileged administrator roles — and the RBAC Administrator role you should use instead of User Access Administrator
🔹 **Control plane vs data plane** — the demo that surprises everyone: Contributor on a storage account cannot read a blob
🔹 Demo — a VM reads a blob with no credential on the machine, then loses access when one role is removed
🔹 Custom roles — clone, cut down, narrow the AssignableScopes, and read the JSON
🔹 ABAC conditions — restrict a role assignment to a single container with an attribute
🔹 Constrained delegation — let an admin hand out exactly one role to exactly one group, and nothing else
🔹 Deny assignments, Azure Policy, and why a subscription Owner can still be blocked
🔹 **Azure Blueprints is retiring** — the dates, and what replaces it
🔹 Demo — PIM: eligible instead of active, time-bound activation with MFA, justification and approval
🔹 Auditing with sign-in logs, activity log, and a Resource Graph query for every role assignment
🔹 The hard limits — 4,000, 500, 5,000 — and the ten-minute propagation rule that wastes everyone's afternoon
🔹 AZ-104 and AZ-305 exam framing, with the giveaway keywords for both halves

📌 *Who Is This Video For:*

💻 Beginners who've built resources and now need to let other people and applications use them safely
🧑‍🎓 Students preparing for AZ-104 (Administrator) or AZ-305 (Solutions Architect) — identity and access is one of the heaviest scored areas on both
☁️ Anyone who has ever created a user in Azure and been confused about why they can't see anything
🔥 Developers who still have connection strings and API keys sitting in config files and want them gone
🖥️ Admins coming from on-premises Active Directory who need to know what actually transfers and what doesn't
🚀 Anyone responsible for who has access to a production subscription

🔍 *Chapters:*
0:00 Intro — Signing In and Being Allowed In Are Two Different Things
4:00 Part 1 — What Microsoft Entra ID Actually Is
10:00 The Identity Plane vs the Resource Plane
15:00 Tenant, Directory, Subscription — And Why Transfers Wipe RBAC
20:00 Why Entra ID Is Not Active Directory in the Cloud
24:00 Part 2 — Editions: Free, P1, P2, and Where the Paywalls Sit
29:00 Part 3 — Users, UPNs, Custom Domains
34:00 Demo — Create Priya, Sign In as Her, and See Absolutely Nothing
39:00 Part 4 — Groups, and Why You Never Assign to a Person
46:00 Part 5 — MFA, Number Matching, and the Move to Passkeys
54:00 Demo — Authentication Methods Policy, Register MFA, Configure SSPR
60:00 Part 6 — Security Defaults vs Conditional Access
68:00 Demo — A Conditional Access Policy in Report-Only Mode
73:00 Part 7 — App Registrations, Service Principals, Enterprise Apps
80:00 Demo — Register an App and Sign In With the CLI: Authenticated, Unauthorised
85:00 Part 8 — Managed Identities: The Best Idea in Azure Security
92:00 Demo — App Service to SQL With No Password Anywhere
99:00 Parts 9–11 — Guests, Devices, and Hybrid Identity
105:00 Part 12 — The Bridge: Entra Roles vs Azure Roles
111:00 Part 13 — The Three-Part Sentence: Principal + Role + Scope
116:00 Demo — One Role Assignment Fills Priya's Empty Portal
121:00 Part 14 — Role Definitions Under the Hood, and the Contributor Arithmetic
127:00 Part 15 — Job Function Roles vs Privileged Administrator Roles
132:00 Part 16 — Control Plane vs Data Plane: The Demo That Surprises Everyone
139:00 Part 17 — A VM That Reads a Blob With No Credential
146:00 Part 18 — Building a Custom Role From Scratch
152:00 Part 19 — ABAC Conditions and Constrained Delegation
159:00 Part 20 — Deny Assignments, Azure Policy, and the End of Blueprints
165:00 Part 21 — PIM: Just-in-Time Admin Access
171:00 Part 22 — Auditing, Troubleshooting, and the Hard Limits
177:00 Part 23 — Exam Framing for AZ-104 and AZ-305
181:00 Summary, Key Takeaways, and Cleanup

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #MicrosoftEntraID #AzureAD #AzureRBAC #RBAC #ManagedIdentity #ConditionalAccess #AzureMFA #Passkeys #PIM #PrivilegedIdentityManagement #AzurePolicy #CustomRoles #ABAC #AzureSecurity #AzureForBeginners #MicrosoftAzure #AZ104 #AZ305 #LearnAzure #AzurePortal #AzureTutorial #LearnWithMithran #CloudComputing #GreensTechnologies #AzureTraining #AzureCertification #IdentityAndAccessManagement

---

## Tags

microsoft entra id tutorial, azure active directory tutorial, azure rbac explained, azure role based access control, entra id vs active directory, azure managed identity, system assigned managed identity, user assigned managed identity, app service to sql managed identity, azure app registration vs service principal, azure conditional access, security defaults azure, azure mfa number matching, passkeys azure 2026, self service password reset azure, azure custom role, role definition json, actions notactions dataactions, control plane vs data plane azure, storage blob data reader, azure abac conditions, constrained delegation azure, rbac administrator role, privileged identity management, pim azure, azure policy deny effect, azure blueprints retirement, deny assignments azure, azure role assignment limits, az-104, az-305, azure for beginners, azure portal, azure tutorial, learnwithmithran, greens technologies, azure training, azure certification, microsoft azure
