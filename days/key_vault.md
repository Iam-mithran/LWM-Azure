# Day 18 — Azure Key Vault

> Phase 5 — Identity, Security + Monitoring
>
> **Scope note.** This file previously read "Key Vault & Security Center" and gave Defender for Cloud
> and Sentinel equal billing with Key Vault. `course_outline.md` — which is the authority — lists
> Day 18 as **Key Vault**. Resolved in favour of the outline: Key Vault is the hands-on core
> (Parts 1–13), and Defender for Cloud + Sentinel get **one concept-level closing part** with the
> free Secure Score as the only lab step. Rationale below.

## Scope Rules for This Day

Same standard as the reworked Day 17 — student-friendly depth, not a reference manual:

- **14 parts.** Roughly 12,000 words. Around 90–100 minutes of video.
- **Every hands-on step is free or rounds to zero.** Key Vault Standard has no base fee; our whole
  lab is a few hundred operations at $0.03/10,000 — about a third of one cent.
- **Paid features are theory only** — Premium/HSM, Managed HSM, integrated-CA certificate renewal,
  private endpoints, paid Defender plans, Sentinel.
- **Interview and real-job relevance is the filter**, exactly as on Day 17.
- **Cut deliberately:** Managed HSM administration, BYOK/key import ceremonies, cross-region vault
  replication internals, Key Vault backup/restore APIs, certificate issuer/contact administration,
  the full ABAC condition language, Sentinel beyond a definition. Several survive as one-line
  mentions so the vocabulary is familiar; none get a demo.

### Why Defender and Sentinel are one part, not half the video

Three reasons. Defender for Cloud is **posture tooling for the whole subscription**, not a Key Vault
topic — it belongs conceptually next to Day 19's monitoring. Sentinel is consumption-priced and gets
expensive fast, so it can never be a student lab. And giving them equal weight would push this day
past the 2-hour cap that Day 17 established as the ceiling.

What they *do* earn is a closing part, because Secure Score is genuinely the best free learning tool
in Azure and it hands back several of this very day's decisions as recommendations — purge protection
off, no private endpoint. That closes the loop better than a standalone treatment would.

## Portal Currency — Verified September 2026

Researched against current Microsoft documentation before writing. Five findings shaped the script:

| Finding | Detail | Where it lands |
|---|---|---|
| **RBAC is now the default** | Key Vault API version **2026-02-01** makes `enableRbacAuthorization = true` the default for new vaults, matching what the portal already did. Existing vaults are untouched. | Part 3 — the headline of the day |
| **A real deadline** | **All control-plane API versions before 2026-02-01 retire 27 February 2027.** ARM/Bicep/Terraform/SDK calls must be updated. Data plane unaffected. Cloud Shell always uses the latest API version, so old `az keyvault create` scripts now produce RBAC vaults. | Part 3 |
| **Soft delete is mandatory** | No longer optional on any vault. Retention 7–90 days and **can never be changed after creation**. A soft-deleted vault **keeps its name reserved**. | Parts 2 and 12 |
| **Trusted services gap** | The trusted-Microsoft-services bypass **does not cover App Service Key Vault references**. Enabling the vault firewall breaks the Part 10 demo, and the fix needs VNet integration — impossible on Free F1. | Part 11 — why the firewall demo is look-don't-touch |
| **Defender free tier changes** | **From 27 October 2026, Foundational CSPM becomes opt-in** and is no longer on by default for new subscriptions. Still free, but a new subscription shows nothing until enabled. | Part 14 |

Also confirmed: Key Vault reference syntax (`SecretUri=` and `VaultName=;SecretName=`), the ~24-hour
App Service reference cache, `SecretNearExpiry` firing 30 days out and requiring an expiry date to
exist, the current data-plane role list, and that **Key Vault Contributor's `DataActions` is empty**.

## Goal

Answer the question Day 17 deliberately left open: **where does the secret actually live?** Then prove
it by running an application that reads a database password it has no credential for.

---

## Part One — The Vault (Parts 1–7)

**1. Why Key Vault Exists**
The three wrong answers (code, config file, app settings) and why they fail for the same reason — the
secret gets copied. What a vault is. **The three object types and why they're genuinely different:
secrets come back, keys never leave, certificates are managed X.509 objects.**

**2. Creating Your First Vault**
Every option on the blade, slowly. **The two permanent settings** — retention period (never
changeable) and purge protection (never disableable).
*Demo: create `kv-lwm-day18-<name>`, Standard, 90-day retention, purge protection off so cleanup works.*

**3. The Permission Model, and the Deadline**
RBAC vs access policies, the four failings of access policies, the **2026-02-01 default flip** and
the **27 Feb 2027 API retirement**. Switching a live vault invalidates access policies instantly.
*Demo: read both models on the Access configuration blade, change nothing.*

**4. Secrets**
Versions and the versioned-vs-current URI decision. Enabled/activation/expiry. Expiry does not delete
and does not warn.
*Demo: store the real `db-lwm-demo` connection string; create a second version; disable the old one.*

**5. Keys — Cryptography as a Service**
The idea that carries the part: **you send data, the vault returns a result, the key never moves.**
CMK, signing, wrapping. RSA/EC and the `-HSM` Premium split. Keys self-rotate.
*Demo: create an RSA key; look for the "show value" button and find there isn't one.*

**6. Certificates**
Policy, auto-renewal, auto-distribution. Key Vault doesn't sell certificates.
*Demo: self-signed cert, then find it listed under **both** Keys and Secrets — and the access trap
that follows from that.*

**7. Tiers, Limits and Cost**
Standard / Premium / Managed HSM table. **Managed HSM is a separate resource and holds keys only.**
Throttling at ~2,000 transactions/10s and the "cache, don't fetch per request" rule. **No demo.**

---

## Part Two — Access, Integration, Operations (Parts 8–14)

**8. Key Vault RBAC Roles, and Why Owner Can't Read a Secret**
The role table. **"User" reads and uses; "Officer" manages.** Data-plane roles are inert on
access-policy vaults. The precise sentence: *Owner reads secrets only because Owner can assign itself
the data role.*
*Demo: give `grp-finance-team` **Key Vault Secrets User** — Priya reads the secret and is denied on
Keys. Then read `Key Vault Contributor`'s empty `DataActions`.*

**9. Delegating Access, and the Contributor Back Door**
**The most important operational fact of the day:** Contributor on a vault can grant itself data
access. Mitigations — isolate production vaults in their own resource group. **Key Vault Data Access
Administrator** and its built-in ABAC condition. ABAC introduced here for the first time in the course.
*Demo: read the constrained role's JSON.*

**10. The Payoff — An App That Reads a Secret It Cannot See**
The flagship. Token-flow diagram with **zero credentials in it**.
*Demo: new Free F1 App Service → system-assigned identity → **Key Vault Secrets User** → app setting
`@Microsoft.KeyVault(SecretUri=…)` → the green "Key Vault Reference" tick.* Includes a
symptom→cause→fix troubleshooting table and the ~24-hour cache behaviour.

**11. Networking**
Three levels: public / firewall+service endpoints (free) / private endpoint (💳). Trusted Microsoft
services — **and the documented gap that it doesn't cover Key Vault references.**
*Demo: look, don't touch — enabling the firewall would break Part 10 on Free F1.*

**12. Soft Delete, Purge Protection and Recovery**
Mandatory now. Name reservation. Purge protection as ransomware defence, and why it's irreversible.
*Demo: delete a throwaway secret → Manage deleted secrets → Recover → delete again → Purge. Then find
**Manage deleted vaults** in the Key vaults toolbar.*

**13. Monitoring, Expiry and Rotation**
Key Vault logs nothing by default and warns about nothing by default. `SecretNearExpiry` at 30 days,
needs an expiry date to exist. **Keys rotate themselves; secrets need a Function.**
*Demo: diagnostic setting → Log Analytics → a KQL query showing the App Service's managed identity
fetching the secret.*

**14. Defender for Cloud, Sentinel and Interview Prep**
**Free Foundational CSPM = posture. Paid Defender plans = threat detection. Sentinel = SIEM.** The
**27 Oct 2026 opt-in change**. Combined interview/exam keyword table.
*Demo: read Secure Score and filter recommendations for Key Vault — several of today's own decisions
come back as findings.*

**Cleanup** — including **purging the soft-deleted vault**, which the resource-group delete leaves
behind. That step is the practical lesson of Part 12.

---

## Hands-On Demo Summary

**Account requirements: everything below is free or rounds to zero.** Key Vault Standard has no base
fee; storing objects is free; our operation count costs a fraction of a cent. The App Service is Free
F1. The only genuine (tiny) cost is Log Analytics ingestion in Part 13, and cleanup removes it.

- ✅ Create `kv-lwm-day18-<name>` — Standard, 90-day retention, purge protection off
- ✅ Read both permission models on Access configuration, change nothing
- ✅ Store `db-lwm-demo`'s connection string as `DbConnectionString` with an expiry date
- ✅ Create a second version; disable the old one
- ✅ Create RSA key `key-lwm-demo`; confirm there is no way to read it
- ✅ Create self-signed `cert-lwm-demo`; find it under Keys **and** Secrets
- ✅ Assign **Key Vault Secrets User** to `grp-finance-team` — Priya reads the secret, denied on Keys
- ✅ Read `Key Vault Contributor` JSON — empty `DataActions`
- ✅ Read `Key Vault Data Access Administrator` JSON — the ABAC condition
- ✅ New Free F1 App Service + system-assigned managed identity
- ✅ **Key Vault Secrets User** to that managed identity
- ✅ `@Microsoft.KeyVault(SecretUri=…)` app setting → **green "Key Vault Reference" tick**
- ✅ Networking blade — read the firewall and private endpoint options, save nothing
- ✅ Delete → recover → delete → purge a throwaway secret
- ✅ Diagnostic setting → Log Analytics → KQL audit query
- ✅ Defender for Cloud Secure Score + Key Vault recommendations
- ✅ Cleanup — **including purging the soft-deleted vault**

**Explained but not demonstrated (paid):** Premium/HSM keys, Managed HSM, integrated-CA certificate
renewal ($3/renewal), private endpoints (~$7.30/month), paid Defender plans, Microsoft Sentinel.

## Cleanup

Delete the diagnostic setting, remove the App Service role assignment, delete `rg-day18-demo`, then
**purge the soft-deleted vault from Key vaults → Manage deleted vaults** — otherwise the name stays
reserved for 90 days. **Keep** `Priya Sharma`, `grp-finance-team` and `db-lwm-demo` for Day 30.

## Summary

Day 17 asked *who are you* and *what may you do*. Day 18 answers the question they left hanging:
*where does the secret actually live?* One copy, one identity-based access decision, one audit log.

The spine of the day is the same control-plane/data-plane split proved on Day 17, now in the place it
matters most — **Key Vault Contributor manages the vault and cannot read one secret** — plus its
dangerous corollary, that Contributor can grant itself data access anyway. The payoff is an App
Service reading a database password through a managed identity and a `@Microsoft.KeyVault(…)`
reference, with **no credential anywhere in its configuration**. Day 17 gave the app an identity;
Day 18 gave it something worth protecting.
