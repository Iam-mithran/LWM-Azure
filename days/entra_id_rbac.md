# Day 17 — Microsoft Entra ID & Azure RBAC

> Phase 5 — Identity, Security + Monitoring
>
> **Merged day.** This was previously two videos (Entra ID, then Azure RBAC). They are now
> one, because authentication and authorisation only make sense taught together — the
> "I created the user, why can't they see anything?" question has its answer in the RBAC half.

## Goal

Understand Azure's two access questions and the two systems that answer them:
**who are you?** (Microsoft Entra ID) and **what are you allowed to do?** (Azure RBAC).

## Key Topics — Part One: Identity

- What is Microsoft Entra ID — Microsoft's cloud identity and access management service
- The two planes: identity plane (Entra ID) vs resource plane (Azure Resource Manager)
- Tenant, directory, subscription — and why subscription transfer deletes all RBAC
- Entra ID is not AD DS — OAuth/OIDC/SAML vs Kerberos/LDAP, flat vs hierarchical
- Editions: Free, P1, P2, ID Governance, Entra Suite — where the paywalls sit
- Users: cloud-only vs synced vs guest, member vs guest, UPNs, custom domains, 30-day restore
- Groups: security vs Microsoft 365, assigned vs dynamic (P1), owners, nesting/transitivity
- Authentication: MFA factors, method ranking, number matching, SSPR, Temporary Access Pass
- **Passkeys become the default from 1 Sept 2026**; SMS/voice MFA retiring; blocking prompt 1 Feb 2027
- Security defaults (six protections, free) vs Conditional Access (P1) — mutually exclusive
- Mandatory MFA for Azure: portal (Phase 1) and ARM/CLI/PowerShell/Terraform/Bicep (Phase 2)
- App registrations vs service principals vs enterprise applications
- Client secrets vs certificates vs federated credentials (workload identity federation)
- Delegated vs application permissions; Graph permissions are not Azure RBAC
- Managed identities: system-assigned vs user-assigned, IMDS at 169.254.169.254, free
- External identities / B2B guests; Azure AD B2C retired 15 March 2026 → Entra External ID
- Devices: registered / joined / hybrid joined
- Hybrid identity: Connect Sync vs Cloud Sync; PHS vs PTA vs Federation

## Key Topics — Part Two: Authorisation

- The three-part sentence: **security principal + role definition + scope = role assignment**
- Four principal types: user, group, service principal, managed identity
- Four scope levels: management group → subscription → resource group → resource; inherits downward
- Additive evaluation model; ARM's nine-step evaluation flow; deny checked first
- Role definition JSON: `Actions`, `NotActions`, `DataActions`, `NotDataActions`, `AssignableScopes`
- `Actions − NotActions` arithmetic; **Contributor = Owner minus authorization write/delete**
- 900+ built-in roles; job function roles vs the five privileged administrator roles
- **Role Based Access Control Administrator** — least-privilege alternative to User Access Administrator
- **Control plane vs data plane** — a Contributor on a storage account cannot read a blob
- RBAC for workload identities; managed identity group-membership cache (up to 24 h)
- Custom roles (free, 5,000/tenant, one management group in AssignableScopes, no wildcards in scopes)
- ABAC role assignment conditions (GA for blob/queue data actions); condition templates
- **Constrained delegation** — RBAC Administrator + condition limiting roles and principals
- Deny assignments — cannot be created by hand; come from deployment stacks / managed apps
- Azure Policy: effects (Audit, Deny, Modify, DeployIfNotExists); RBAC vs Policy distinction
- **Azure Blueprints retiring** — new definitions blocked 31 July 2026, full retirement 31 Jan 2027;
  migrate to Template Specs + Deployment Stacks
- PIM (P2): eligible vs active, time-bound, activation with MFA/justification/approval, access reviews
- Limits: 4,000 role assignments per subscription (fixed), 500 per management group, 5,000 custom roles
- Troubleshooting: 10-minute propagation, orphaned "Identity not found", PrincipalNotFound,
  role assignments don't follow moved resources, classic administrators fully retired May 2026

## Hands-On Demo

**Account Requirements:** Entra ID Free and **all of Azure RBAC** are free. Conditional Access
needs P1; PIM and dynamic groups need P2/P1. Everything else is free tier.

Identity:

- ✅ Find your tenant ID; compare portal.azure.com and entra.microsoft.com
- ✅ Create user `Priya Sharma`; sign in as her and discover she can see **nothing**
- ✅ Create security group `grp-finance-team`; add Priya
- 💳 Dynamic group with a rule on `department`
- ✅ Authentication methods policy; register MFA; configure SSPR
- ✅ Read security defaults; 💳 build a Conditional Access policy in report-only mode
- ✅ Register `app-lwm-demo`, create a secret, sign in with the CLI — authenticated, unauthorised
- ✅ User-assigned managed identity; **passwordless App Service → SQL** (`FROM EXTERNAL PROVIDER`)
- ✅ Invite a B2B guest; explore cross-tenant access settings

Authorisation:

- ✅ Assign **Reader** to `grp-finance-team` at `rg-day17-demo` — Priya's portal fills up
- ✅ Show inheritance and Check access on a child resource
- ✅ Read the Contributor / Reader / Storage Blob Data Reader JSON
- ✅ Right-size with **Website Contributor** at resource scope instead of Contributor
- ✅ **The surprise demo:** Contributor on a storage account cannot read a blob →
  add Storage Blob Data Reader → it works
- ✅ Assign a role to the App Service's managed identity
- ✅ VM with system-assigned identity: `az login --identity` + `--auth-mode login` reads a blob
  with no credential; remove the role → `AuthorizationPermissionMismatch`
- ✅ Build custom role `LWM VM Operator` (clone, cut down, narrow AssignableScopes, read the JSON)
- ✅ ABAC condition limiting blob reads to one container
- ✅ Constrained delegation: RBAC Administrator + condition — Priya's role dropdown shrinks
- ✅ Azure Policy requiring a `CostCenter` tag — deployment rejected even for the subscription Owner
- 💳 PIM: eligible time-bound Contributor, activation with MFA + justification + approval
- ✅ Activity log, sign-in logs (incl. managed identity sign-ins), Resource Graph role-assignment query

## Summary

Two questions, two systems. Entra ID proves who you are and issues the token; Azure RBAC decides
what that token can do. The bridge between the halves is Priya: authenticated in Part 3, still
seeing an empty portal, until one role assignment — principal, role, scope — fills it in Part 13.
The lesson that sticks hardest is control plane vs data plane: full Contributor rights on a storage
account, and you still cannot open the file inside it.
