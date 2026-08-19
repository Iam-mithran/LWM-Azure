# Day 17 — Microsoft Entra ID & Azure RBAC

> Phase 5 — Identity, Security + Monitoring
>
> **Merged day.** Previously two videos (Entra ID, then Azure RBAC). They are one now, because
> authentication and authorisation only make sense taught together — the "I created the user, why
> can't they see anything?" question has its answer in the RBAC half.

## Scope Rules for This Day

Deliberately **not** an exhaustive reference. This day is reworked to be student-friendly:

- **14 parts, not 24.** Roughly 9,000 words. Around 90 minutes of video.
- **Every hands-on step is free tier.** There are no paid demos at all.
- **Paid features are theory only** — Conditional Access, dynamic groups and PIM are explained in a
  few paragraphs each because they come up in every interview, but nobody is asked to click them.
- **Interview and real-job relevance is the filter.** If it isn't asked in interviews or used in a
  normal month of Azure work, it's cut.
- **Cut deliberately:** ABAC conditions, constrained delegation, deny-assignment internals, Azure
  Blueprints, the nine-step ARM evaluation flow, the full limits table, Resource Graph KQL auditing,
  devices/Intune, workload identity federation detail, external-tenant settings, custom domains.
  Several survive as one-line mentions so the vocabulary is familiar; none get a demo.

## Goal

Answer Azure's two access questions and know which system answers which:
**who are you?** (Microsoft Entra ID) and **what are you allowed to do?** (Azure RBAC).

---

## Part One — Identity (Parts 1–7)

**1. What Entra ID Is, and the Two Planes**
Identity plane (Entra ID) vs resource plane (ARM) — the mental model the whole day hangs on.
Tenant / directory / subscription. Entra ID is not AD DS (short table). Two portals.
*Demo: find your tenant ID.*

**2. Users**
User object, UPN, object ID and why RBAC stores the GUID. Cloud-only vs synced vs guest. Member vs
guest. 30-day restore.
*Demo: create `Priya Sharma`, sign in as her in a private window, discover she can see **nothing**.*
This is the hinge of the entire video.

**3. Groups**
Security vs Microsoft 365 groups. Assigned vs dynamic (dynamic = P1, **theory only**). Why you never
assign permissions to a person — including the hard 4,000-assignment ceiling.
*Demo: create `grp-finance-team`, add Priya.*

**4. Authentication — MFA, Security Defaults, SSPR**
Factor types, why MFA blocks ~99% of identity attacks, number matching, method ranking, passkeys
becoming default (1 Sept 2026) and SMS/voice retiring. Security defaults (free, six protections).
Mandatory MFA for the portal and ARM. Conditional Access explained as the paid if/then engine —
**theory only**, plus the CA-vs-RBAC distinction.
*Demo: read security defaults; register MFA on your own account; enable SSPR.*

**5. Applications — App Registrations & Service Principals**
Why software needs identity. App registration vs service principal vs enterprise application.
Client secret vs certificate vs federated credential. Delegated vs application permissions.
*Demo: register an app, create a secret, sign in with the CLI — authenticated, unauthorised.*

**6. Managed Identities**
The credential problem, stated plainly. What a managed identity is and how the token flow works.
System-assigned vs user-assigned. Free.
*Demo: enable a system-assigned identity on an App Service. (Its role assignment lands in Part 11.)*

**7. Guests and Hybrid Identity (short)**
B2B guests in four sentences. PHS vs PTA vs Federation as one table — a reliable interview question.
**Theory only.**

---

## Part Two — Authorisation (Parts 8–14)

**8. Entra Roles vs Azure Roles**
The single most misunderstood thing in Azure. Global Administrator ≠ access to any resource.
The elevate-access toggle as a break-glass door.
*Demo: give Priya User Administrator — she can create users and still sees no resources.*

**9. Azure RBAC — Who, What, Where**
Security principal + role definition + scope = role assignment. Four principal types, four scope
levels, inheritance downward, additive evaluation. The golden rule. Token refresh after group changes.
*Demo: assign **Reader** to the group at resource-group scope — Priya's empty portal fills. Then
inheritance on a child resource, and **Check access**.*

**10. Inside a Role Definition, and the Built-in Roles**
`Actions`, `NotActions`, `DataActions`, `NotDataActions`, `AssignableScopes`. The subtraction that
proves **Contributor = Owner minus the ability to grant access**. Reader / Contributor / Owner /
User Access Administrator table. Prefer job function roles; prefer RBAC Administrator over UAA.
*Demo: read the Contributor, Reader and Storage Blob Data Reader JSON.*

**11. Control Plane vs Data Plane**
The demo nobody predicts correctly, and the highest-value five minutes of the day. Includes the
`listKeys` back door in one sentence, and closes the managed-identity loop from Part 6.
*Demo: Contributor on a storage account cannot read a blob → add Storage Blob Data Reader → it works.
Then assign that same data role to the App Service's managed identity.*

**12. Custom Roles**
When built-in doesn't fit, the three starting points, why not to use wildcards. Free.
*Demo: clone Virtual Machine Contributor into `LWM VM Operator`, cut it down, narrow AssignableScopes.*

**13. Azure Policy vs RBAC**
"RBAC controls who can act; Policy controls what may exist." Effects: Audit, Deny, Modify,
DeployIfNotExists. Deny assignments in one line.
*Demo: require a `CostCenter` tag — the deployment is rejected even for the subscription Owner.*

**14. PIM, Troubleshooting and Interview Prep**
PIM vocabulary — eligible vs active, time-bound, activation with MFA/justification/approval
(**P2 — theory only**). The gotchas that actually waste your day: 10-minute propagation, orphaned
"Identity not found", role assignments don't follow moved resources, subscription transfer wipes RBAC.
Combined interview/exam keyword table for both halves.

**Cleanup** — resource plane and identity plane separately, because deleting a resource group removes
nothing from the directory.

---

## Hands-On Demo Summary

**Account requirements: everything below is free tier. There are no paid steps.** Entra ID Free
covers users, groups, MFA, security defaults, SSPR, app registrations and guests. **Azure RBAC is
free in its entirety** — every role assignment and every custom role. Managed identities are free.
Azure Policy built-ins are free.

- ✅ Find your tenant ID
- ✅ Create `Priya Sharma`; sign in as her and see nothing
- ✅ Create `grp-finance-team` and add her
- ✅ Read security defaults; register MFA; enable SSPR
- ✅ Register `app-lwm-demo`, create a secret, `az login --service-principal` → `az group list` is empty
- ✅ Enable a system-assigned managed identity on an App Service (Free F1)
- ✅ Give Priya User Administrator — directory power, zero resource access
- ✅ Assign **Reader** to the group at `rg-day17-demo` — her portal fills up
- ✅ Inheritance on a child resource, and **Check access**
- ✅ Read the Contributor / Reader / Storage Blob Data Reader JSON
- ✅ Contributor on a storage account **cannot read a blob** → add Storage Blob Data Reader → it works
- ✅ Assign the data role to the App Service's managed identity
- ✅ Build the custom role `LWM VM Operator`
- ✅ Azure Policy requiring a `CostCenter` tag blocks a subscription Owner
- ✅ Cleanup — both planes

**Explained but not demonstrated (paid):** Conditional Access (P1), dynamic groups (P1), PIM (P2).

## Cleanup

Delete `rg-day17-demo`, the policy assignment, the custom role and the app registration. **Keep**
`Priya Sharma`, `grp-finance-team` and `db-lwm-demo` — Day 18 (Key Vault) and Day 30 (Capstone)
reuse them.

## Summary

Two questions, two systems. Entra ID proves who you are and issues the token; Azure RBAC decides what
that token can do. The bridge is Priya: authenticated in Part 2, staring at an empty portal, until one
role assignment — principal, role, scope — fills it in Part 9. The lesson that sticks hardest is
control plane vs data plane: full Contributor rights on a storage account, and you still cannot open
the file inside it.
