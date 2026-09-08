# Day 18 — Azure Key Vault

**Phase 5 — Identity, Security + Monitoring**

> Yesterday you created a client secret. Remember what I said at the time? *"That secret is a password to your tenant, and it's now in your Cloud Shell history."* We deleted it at cleanup and moved on. But that question never actually went away, and today we answer it properly. Because every application you will ever build needs to know something it must not tell anyone — a database password, an API key, a certificate's private key — and it needs to know it at 3am on a Sunday with nobody watching. Where does that thing *live*? Not in your code. Not in your config file. Not in an environment variable someone can print. Today you build the answer, and by the end of this video you'll have a running web app that reads a database password it cannot see, from a vault it has no credential for, and you will not have typed a single secret into a single configuration screen.

---

## What You'll Learn

**Part One — The Vault itself**

- What Key Vault is, and the three genuinely different things it stores: **secrets, keys and certificates**
- Every option on the create blade — and what each one commits you to, some of them **permanently**
- **The permission model decision**, which changed in February 2026 and has a hard deadline in February 2027
- Storing secrets properly: versions, expiry dates, and the enable/disable switch nobody uses
- **Cryptography as a service** — the idea that a key can be used without ever being read
- Certificates, tiers, limits, and what this actually costs you

**Part Two — Access, integration and operations**

- The Key Vault RBAC roles, and the proof that **a subscription Owner cannot read a secret**
- The **Contributor back door** — the most important thing on this page for a real job
- **The payoff:** App Service + managed identity + a Key Vault reference, with zero credentials anywhere
- Networking: firewall, trusted services, and where private endpoints fit
- **Soft delete and purge protection** — the safety net, and the switch you can never turn off
- Monitoring, expiry and rotation
- Defender for Cloud's free Secure Score, and an interview keyword table

---

## Before We Begin

**What today costs.** Key Vault is the first service in this course where I can't just say "it's free." So let me be exact, because vague answers about money are how people get surprise bills.

| | Cost | Today |
|---|---|---|
| **Storing** secrets, keys, certificates | **No charge at all.** There is no base fee and no per-object fee. | ✅ |
| **Operations** (every get, set, list) | **$0.03 per 10,000 operations**, Standard tier | ✅ — our entire lab is a few hundred operations, so roughly **a third of one cent** |
| **Self-signed certificate** created in the vault | Charged as normal operations | ✅ |
| **Certificate renewal** through an integrated public CA | **$3 per renewal request** | 💳 — explained, not demonstrated |
| **Premium tier** (HSM-backed keys) | ~**$1/month per key**, plus operations | 💳 — explained, not demonstrated |
| **Managed HSM** (dedicated hardware pool) | **~$2,300+/month** | 💳 — vocabulary only. Do not click this. |
| **Private endpoint** | ~**$7.30/month** plus data processing | 💳 — instructor demo |

So: **the Standard-tier lab we're doing today rounds to zero**, but Key Vault is not on the free tier and it does bill from the first transaction. That's an honest distinction and it's worth internalising — "costs nothing meaningful" and "is free" are different sentences, and in a job interview you want to say the accurate one.

!!! danger "Two settings today are genuinely permanent"
    Most things in Azure are undoable. Two settings on the Key Vault create blade are not, and I want you to see them coming rather than discover them:

    1. **Soft-delete retention period** — once the vault is created, that number **cannot be changed for the life of the vault**.
    2. **Purge protection** — once enabled, **it cannot be disabled by you, by your Global Administrator, or by Microsoft support.** That is the entire point of it, and it is not a bug.

    We'll set both deliberately in Part 2. Read the warnings when we get there.

**Set this up first:**

- A resource group called `rg-day18-demo`.
- **`Priya Sharma` and `grp-finance-team` from Day 17.** We kept them alive on purpose — today they earn their keep. If you deleted them, recreate the user and the security group; it takes two minutes and Day 17 Parts 2 and 3 have the steps.
- **`db-lwm-demo`, your Azure SQL database from Day 16**, still alive on the free offer. Its connection string is the secret we're going to protect.
- Your **private/incognito window** again, for signing in as Priya.

!!! tip "One idea from yesterday does most of the work today"
    Day 17 Part 11 proved that **control plane and data plane are separate**: a Contributor on a storage account could delete the account and could not read a blob inside it. Key Vault is the *second* instance of exactly that pattern, and it's the one that shows up most in real jobs. If that demo made sense, today will feel familiar fast. If it didn't, go back and watch it — everything from Part 8 onwards is built on it.

---

## PART ONE — THE VAULT

### Part 1 — Why Key Vault Exists

#### The Problem, Honestly Stated

Your application needs a database password. Where do you put it?

Every beginner answers the same way, in the same order, and every answer is wrong:

- **"In the code."** Now it's in git. Git has history. Deleting the line doesn't delete the secret — it's still in the commit from four months ago, and anyone who clones the repo has it forever. This is the single most common way real secrets leak, and there are bots that do nothing but scan public repositories for exactly this.
- **"In a config file that's gitignored."** Better, until someone copies the file to a colleague on Slack, or the file gets baked into a container image, or a new developer doesn't have it and someone pastes it into a chat to unblock them.
- **"In an App Service application setting."** Genuinely better — App Service encrypts those at rest. But now **anyone with Contributor on that app can read every setting in the portal**, there's no audit trail of who read what, no expiry, no rotation, and if the same password is used by five apps you now have five copies to update when it changes.

The problem underneath all three is the same: **the secret is copied.** Once a secret exists in more than one place, you have lost control of it, because you can no longer answer the two questions that matter — *who can read this?* and *if I change it, what breaks?*

#### What Key Vault Actually Is

**Azure Key Vault is a managed service whose entire job is to hold sensitive values, hand them out only to identities you've authorised, and write down every single access in a log.**

Four properties make it different from a config file:

- **One copy.** The secret lives in exactly one place. Applications fetch it at runtime. Change it in the vault and every consumer picks up the new value — nothing to redeploy.
- **Access is an identity decision, not a file-permission decision.** You grant *this managed identity* the right to read *these secrets*, using the same RBAC model you learned yesterday.
- **Everything is logged.** Every read, every write, every failed attempt, with the identity that made it. When an auditor asks "who accessed the production database password in March?", there's an answer.
- **Objects have lifecycle.** Versions, activation dates, expiry dates, enable/disable — a secret is a managed object, not a string.

Underneath, Key Vault is multi-tenant, backed by hardware security modules, and **regional** — the vault lives in a region, and Azure automatically replicates its contents to the paired region for durability.

#### The Three Things a Vault Holds

This is the first thing people get wrong, and it's an easy interview mark. A vault holds three object types, and they are **not** three names for the same idea.

| | **Secret** | **Key** | **Certificate** |
|---|---|---|---|
| **What it is** | Any value up to 25 KB — a password, connection string, API token | An asymmetric crypto key — RSA or EC | An X.509 certificate |
| **Can you read the value back?** | **Yes.** That's the point. | **No. Never.** Not you, not an admin, not Microsoft. | Partly — see below |
| **What the vault does for you** | Stores it and hands it back to authorised callers | **Performs cryptographic operations on your behalf** — sign, verify, encrypt, decrypt, wrap, unwrap | Manages the whole certificate lifecycle, including renewal |
| **Typical use** | Database connection string, third-party API key | Encrypting data at rest, signing tokens, customer-managed keys | TLS/SSL for a website |

The row that matters is the third one. **A secret is something the vault gives you. A key is something the vault uses for you.** With a key, you never receive the key material — you send the vault your data and say "sign this," and it sends back a signature. The private key never crosses the network, never lands in your application's memory, and cannot be exfiltrated even by someone who fully compromises your app. That's a genuinely different security property, and it's the reason keys and secrets are separate object types instead of one.

Certificates are the interesting hybrid: **a certificate in Key Vault is actually three objects at once** — the certificate itself, plus an addressable *key* (the private key) and an addressable *secret* (the full PFX bundle). Which is why the role that lets someone read a certificate's private key is called **Key Vault Certificate User**, and why *Key Vault Secrets User* can also read the secret half of a certificate. Store that away; we come back to it in Part 8.

!!! tip "The one-sentence version for interviews"
    *"Secrets are values you get back. Keys never leave the vault — the vault does the cryptography for you. Certificates are managed X.509 objects with a key and a secret underneath."*

---

### Part 2 — Creating Your First Vault

Every option on this blade matters, and two of them are permanent, so we're going to walk it slowly rather than clicking Next four times.

#### Hands-On: Create the Vault ✅

1. Search **Key vaults** → **+ Create**. **✅**

2. **Basics tab:** **✅**

   | Setting | Value | Why |
   |---|---|---|
   | **Resource group** | `rg-day18-demo` | |
   | **Key vault name** | `kv-lwm-day18-<yourname>` | **Globally unique** — it becomes a DNS name, `https://<name>.vault.azure.net`. Alphanumerics and hyphens, 3–24 characters. |
   | **Region** | Same region you've been using | The vault is regional. Put it near the apps that will read from it — every secret fetch is a network call. |
   | **Pricing tier** | **Standard** | Premium's only difference is HSM-backed keys. More on this in Part 7. |

3. **Days to retain deleted vaults:** leave at **90**. **✅**

   Read that field carefully, because the portal is being slightly coy about what it controls. This is the **soft-delete retention period** — how long a deleted vault, *and every deleted object inside it*, stays recoverable before it can be permanently purged. Legal range is **7 to 90 days**.

    !!! danger "This number can never be changed"
        Once the vault is created, the retention period is **fixed for the life of the vault**. There is no setting to edit, no support ticket that changes it. If you need 7 days and you created it with 90, your only option is to create a new vault and migrate.

        Why would anyone want 7? Because during the retention window, **the vault's name is reserved and cannot be reused.** Delete `kv-prod-01` with 90-day retention and you cannot create a new vault called `kv-prod-01` for ninety days. That has ruined more than one redeployment. For learning, and for production, **90 is the right answer** — you want the long safety net. Just know why the option exists.

4. **Purge protection:** for this demo, leave it **disabled**. **✅**

    !!! danger "The one switch in Azure you can never turn off"
        Purge protection means that during the retention period, a deleted vault or object **cannot be permanently purged by anyone**. Not you. Not the subscription Owner. Not a Global Administrator. Not Microsoft support. The deletion simply waits out the full retention window.

        And **enabling it is irreversible.** There is no path back.

        That sounds alarming and it's actually the correct setting for production. It's the protection against a specific, real attack: an attacker who gains admin access deletes *and purges* your vault, destroying the encryption keys that protect your data — a ransomware pattern with no ransom. Purge protection makes that impossible, and it is **required** if the vault holds customer-managed keys for Azure Storage or SQL encryption.

        We're leaving it off today for exactly one reason: **so that cleanup at the end of this video actually works.** Turn it on in production. Never turn it on casually on a test vault, because you'll be living with that vault name until the retention period expires.

5. **Access configuration tab.** Confirm the **Permission model** is set to **Azure role-based access control**. It should already be selected. **✅**

   Don't change it — but do notice the other option sitting there, *Vault access policy*, and notice which one the portal picked for you. That default is new, it's the whole subject of Part 3, and it has a deadline attached.

6. **Networking tab:** leave **Public access** enabled, **All networks**. **✅** We'll come back and lock this down properly in Part 11.

7. **Review + create → Create.** **✅**

8. When it deploys, open it and look at the **Overview** blade. Copy the **Vault URI** — `https://kv-lwm-day18-<yourname>.vault.azure.net/`. **That hostname is the data plane.** Remember from Day 17 that data-plane traffic goes straight to the service's own endpoint and never touches Azure Resource Manager. This is that endpoint. **✅**

---

### Part 3 — The Permission Model, and the Deadline

This is the one genuinely new idea in today's video, it changed a few months ago, and it has a hard cutoff in February 2027. If you learned Key Vault from a tutorial recorded before 2026, this part is the bit you need to unlearn.

#### Two Models, One Recommendation

Key Vault has always had two ways to answer "who may read this secret?"

**Vault access policies (legacy).** A list attached to the vault itself. Each entry says: *this principal gets these operations on secrets, these on keys, these on certificates.* It predates Azure RBAC for data planes and it's entirely self-contained — the permissions live on the vault, nowhere else.

**Azure RBAC.** Exactly what you learned yesterday. Security principal + role definition + scope. The same *Add role assignment* wizard, the same **Check access** blade, the same inheritance, the same everything.

Access policies have four real problems, and they're worth being able to list:

- **They're per-vault.** Fifty vaults means fifty separate permission lists, with no single place to ask "what can this identity reach?" RBAC answers that from the identity's own **Azure role assignments** blade, across the whole subscription.
- **They don't inherit.** You cannot say "the platform team reads secrets in every vault in this resource group." RBAC does that with one assignment at resource-group scope.
- **They're coarse.** Access policy permissions are per-*operation-type* across the whole vault. You cannot express "read only the secrets whose names start with `prod-`." RBAC can, with conditions.
- **They're outside the tooling.** PIM, access reviews, Check access, custom roles, Azure Policy on role assignments — all of it understands RBAC and none of it understands access policies.

#### What Changed in February 2026

**Key Vault API version `2026-02-01` made Azure RBAC the default access control model for newly created vaults.** In the underlying resource, the property is `enableRbacAuthorization`, and it now defaults to `true`.

Three consequences that are worth stating precisely, because the details get garbled in blog posts:

- **New vaults default to RBAC.** That's why step 5 above found the setting already correct. The change brought the API into line with what the portal was already doing.
- **Existing vaults are untouched.** A vault created years ago with access policies keeps them. Updating it with the new API version does **not** silently flip it. Nothing breaks by itself.
- **Access policies are still fully supported.** They are *not* deprecated as a feature. If you genuinely need them on a new vault, you set `enableRbacAuthorization` to `false` explicitly at creation time.

!!! warning "The date that is actually a deadline: 27 February 2027"
    Here's the part that will land on someone's sprint board, so know it.

    **All Key Vault control-plane API versions before `2026-02-01` retire on 27 February 2027.** After that date your vaults keep existing and keep working, but they are only manageable through API version `2026-02-01` or later.

    That means every **ARM template, Bicep file, Terraform provider, SDK package and REST call** that manages a Key Vault has to be updated before that date, regardless of which permission model you use. Data-plane APIs — the actual reading and writing of secrets — are **not** affected, so your running applications are fine. It's the infrastructure code that needs the attention.

    One footnote with a sting: **Azure Cloud Shell always uses the latest API version.** So a `az keyvault create` script that worked last year and didn't specify a permission model will now produce an **RBAC** vault instead of an access-policy vault. If you inherit a script like that and the vault suddenly seems to have "lost" its permissions, that's why.

#### Hands-On: Look at Both Models ✅

1. **Key vault → Settings → Access configuration.** **✅**
2. Confirm **Azure role-based access control** is selected. Click **Vault access policy** to see what the alternative looks like — a permission grid with checkbox columns for Key, Secret and Certificate permissions. Read it, understand its shape, then **click away without saving.** **✅**

    !!! danger "Switching models invalidates the old one instantly"
        Flipping a live vault from access policies to RBAC **immediately invalidates every access policy on it**. If you haven't already assigned equivalent RBAC roles, every application using that vault loses access at that moment. This is a documented, well-known way to cause a production outage.

        The correct migration order is always: **assign the RBAC roles first, verify they work, and only then switch the model.** Never the other way round.

3. Now click **Access control (IAM)** on the vault. **This is the blade that matters now** — the same one you used on a resource group and a storage account yesterday. Key Vault permissions are just Azure role assignments. **✅**

!!! tip "The interview answer"
    *"Which permission model should I use for Key Vault?"* → **Azure RBAC.** It's the default for new vaults as of API version 2026-02-01, it's centrally manageable and inheritable, and it works with the rest of Azure's access tooling. Access policies remain supported but are the legacy path. And separately, **all control-plane API versions before 2026-02-01 retire on 27 February 2027**, so IaC templates need updating either way.

---

### Part 4 — Secrets

A secret is a name, a value up to 25 KB, and a surprising amount of metadata. Let's use the metadata, because almost nobody does and it's where the operational maturity lives.

#### Versions

**Every time you set a value, Key Vault creates a new version** and keeps the old one. Versions are immutable — you never edit a secret's value, you add a version. Each has its own GUID and its own URI:

```text
https://myvault.vault.azure.net/secrets/DbPassword              ← "current version", whatever that is now
https://myvault.vault.azure.net/secrets/DbPassword/a1b2c3d4...  ← this exact version, forever
```

That distinction is a design decision you will make repeatedly:

- **Reference without a version** and your app automatically picks up rotations. Convenient, and the right default.
- **Reference with a version** and your app is pinned. Nothing changes under it — which is exactly what you want when a value must stay in lockstep with a deployment.

#### The Lifecycle Fields

Three fields, all optional, all under-used:

- **Enabled / Disabled** — a disabled secret still exists but cannot be read. It's the "break glass" switch: you suspect a credential leaked, you disable it, everything using it fails immediately and loudly, and you haven't destroyed anything.
- **Activation date** — the secret cannot be read *before* this time. Useful for staging a value ahead of a cutover.
- **Expiration date** — the secret cannot be read *after* this time.

!!! warning "Expiry does not delete anything, and it does not stop anyone setting it"
    Two things people get wrong about expiration:

    1. An expired secret is **not deleted**. It sits there, unreadable, until someone updates or removes it.
    2. Key Vault **does not warn you by default.** It will happily let a production secret expire at 2am and take your app down. Alerting on expiry is something *you* configure, and we'll do it in Part 13.

    So: set expiry dates, because they're the trigger for rotation and every compliance framework wants them — but set up the alerting at the same time, or you've just scheduled an outage.

#### Hands-On: Store a Real Secret ✅

We're going to store the connection string for `db-lwm-demo`, the SQL database from Day 16. A real value, for a real resource.

1. Get the connection string: **SQL databases → `db-lwm-demo` → Settings → Connection strings → ADO.NET** tab. Copy it. **✅**
2. Notice it contains `Password={your_password}` as a placeholder. Replace that with your actual SQL admin password. **Look at what you're now holding in your clipboard** — that is exactly the string that, in a less careful project, would be sitting in an `appsettings.json` in a git repository. **✅**
3. **Key vault → Objects → Secrets → + Generate/Import**. **✅**
4. Fill it in: **✅**

   | Field | Value |
   |---|---|
   | **Upload options** | Manual |
   | **Name** | `DbConnectionString` — alphanumerics and hyphens only; **no underscores**, which catches people out |
   | **Secret value** | Paste the connection string |
   | **Set expiration date** | Tick it, choose a date about 90 days out |
   | **Enabled** | Yes |

5. **Create**. Open the secret and click into the version that appears. Note the **Secret Identifier** — the full versioned URI. **Copy it; Part 10 needs it.** **✅**
6. Click **Show Secret Value** and confirm it round-trips. **✅**
7. Now create a **second version**: back on the secret, **+ New Version**, change the value slightly (add a space), **Create**. You now have two versions listed, the newer one current. **Click the older version and set it to Disabled** — you've just retired a credential without destroying your ability to audit it. **✅**

!!! tip "Naming matters more than it looks"
    Secret names are visible to anyone with even *Key Vault Reader*, so **never put anything sensitive in the name**. `prod-sql-admin-password` is a fine name. `sql-password-Winter2026` is a catastrophe — you've leaked the secret to everyone who can list the vault.

    Pick a convention and hold it: `<environment>-<system>-<purpose>`. Your future self, reading a list of two hundred secrets, will be grateful.

---

### Part 5 — Keys: Cryptography as a Service

This is the part students skim and interviewers ask about, so let's do it properly.

#### The Idea

With a secret, the vault stores a value and gives it back. **With a key, the vault stores the key and never gives it back — instead, it does the work for you.**

Your application sends the vault some data and an instruction: *sign this*, *unwrap this*. The vault performs the operation using a private key that has never left it and returns the result. Your application receives a signature or a plaintext — never the key.

Think about what that removes. If an attacker fully compromises your application server — root access, memory dumps, everything — **they still do not have your private key.** They can ask the vault to perform operations while they hold that access, which is bad, but the moment you cut their access the key is untouched. There is no key to steal, revoke and reissue. That is a fundamentally different blast radius from a stolen key file, and it's why keys are their own object type.

#### What Keys Are Used For

- **Encryption at rest with customer-managed keys (CMK).** Azure Storage, SQL, Disks and Cosmos all encrypt your data by default with Microsoft-managed keys. Point them at a Key Vault key instead and *you* hold the root of that encryption — including the ability to revoke access to your own data by disabling the key. This is the single most common reason enterprises use Key Vault keys, and it's why **purge protection is mandatory** for it.
- **Digital signing** — tokens, documents, code.
- **Key wrapping** — encrypting other keys, the building block of envelope encryption.

#### Key Types and Where They Live

| Type | Meaning |
|---|---|
| **RSA** | Software-protected RSA key (2048 / 3072 / 4096) — Standard tier |
| **RSA-HSM** | The same, but generated and held inside a **hardware security module** — **Premium tier only** 💳 |
| **EC** | Elliptic curve (P-256, P-384, P-521) — smaller and faster than RSA |
| **EC-HSM** | Elliptic curve in an HSM — **Premium tier only** 💳 |

The `-HSM` suffix is the entire difference between Standard and Premium. In both cases the key material is protected and never exported; in Premium it's protected by validated dedicated hardware, which is what regulated industries are required to be able to prove.

#### Rotation Policies

Keys support **automatic rotation** — you set a policy ("rotate 30 days before expiry") and Key Vault generates a new version on schedule, with no human involved. Azure services using the key pick up the new version automatically.

This is worth contrasting sharply with secrets: **keys can rotate themselves; secrets cannot.** Key Vault has no idea what your database password *is* — rotating it means changing it in SQL Server too, which is application-specific work. That's why secret rotation needs an Azure Function and keys don't, and it's a nice distinction to be able to draw in an interview.

#### Hands-On: Create and Use a Key ✅

1. **Key vault → Objects → Keys → + Generate/Import**. **✅**
2. **Options:** Generate. **Name:** `key-lwm-demo`. **Key type:** RSA. **RSA key size:** 2048. **✅**
3. Expand **Set key rotation policy**: switch **Enabled** on, set **Rotation time** to something like 90 days, and note the **Notification time** field — an Event Grid event fires that far ahead. Read it, then set it back to disabled for this demo. **✅**
4. **Create**. Open the key. **✅**
5. **Look for a "show value" button.** There isn't one. There is no way — through the portal, the CLI, the API, or a support ticket — to extract that private key. **The absence of that button is the entire feature.** **✅**
6. Click the current version and note the **Permitted operations** list: encrypt, decrypt, sign, verify, wrapKey, unwrapKey. That's the menu of things the vault will do *for* you. **✅**

!!! note "Which one do I actually use?"
    Rough rule: **if you need the value back, it's a secret. If you need an operation performed, it's a key.**

    A database password is a secret — your app has to send the actual password to SQL Server. A customer-managed encryption key is a key — nothing outside the vault ever needs the raw bytes.

---

### Part 6 — Certificates

A certificate in Key Vault is a managed object with a lifecycle, not a file you uploaded.

#### What You Get

- **Creation or import** — generate a new certificate, or import an existing PFX/PEM.
- **A policy** attached to the certificate describing subject name, key type, validity period, and **what to do when it's about to expire**.
- **Automatic renewal** — for certificates issued by an integrated public CA (DigiCert, GlobalSign), Key Vault will request the renewal itself, before expiry, with nobody involved.
- **Automatic distribution** — App Service, Application Gateway and Front Door can all reference a certificate directly from Key Vault, so a renewal propagates without a redeploy.

That last pair is the real value. **Expired TLS certificates are one of the most common causes of self-inflicted outages in the industry**, and the cause is always the same: renewal was a human's calendar reminder, and the human left the company. Key Vault turns it into infrastructure.

!!! note "Key Vault does not sell you certificates"
    Worth being precise, because it's a common misconception: Key Vault **integrates with** public CAs to automate enrolment and renewal. It does not issue or resell public certificates. You still have an account with the CA. Key Vault automates the paperwork.

#### Hands-On: Create a Self-Signed Certificate ✅

Self-signed means no CA, no cost beyond normal operations, and no browser will trust it — perfect for learning the mechanics.

1. **Key vault → Objects → Certificates → + Generate/Import**. **✅**
2. Fill it in: **✅**

   | Field | Value |
   |---|---|
   | **Method of Certificate Creation** | Generate |
   | **Certificate Name** | `cert-lwm-demo` |
   | **Type of Certificate Authority** | **Self-signed certificate** |
   | **Subject** | `CN=lwm-demo.local` |
   | **Validity Period** | 12 months |
   | **Content Type** | PKCS #12 |
   | **Lifetime Action Type** | *Automatically renew at a given percentage lifetime* — 80% |

3. **Create.** It takes a few seconds to issue. **✅**
4. Open the certificate → the current version. Note the **Certificate Identifier**, and **Download in CER format** — that's the public half, safe to share. **✅**
5. Now the thing that surprises people. Go to **Objects → Keys**. **✅**

   **`cert-lwm-demo` is in the keys list.**

   Go to **Objects → Secrets**. **It's there too.** One certificate, three addressable objects: the certificate, its private key, and the full PFX bundle as a secret.

6. Click **Issuance Policy** on the certificate and read it. Everything about how this certificate renews lives here. Note the **Advanced Policy Configuration** — including the option to select an integrated CA instead of self-signed. **💳 That path costs $3 per renewal request, so we're reading it, not clicking it.** **✅**

!!! warning "That three-objects thing is a real access-control trap"
    Because a certificate's private key is addressable as a secret, **anyone with *Key Vault Secrets User* on the vault can retrieve the full PFX — private key included — for every certificate in it.**

    This catches teams out constantly. They grant an app "just secrets access," reasonably assuming that's a narrow permission, and hand it every private key in the vault. The fix is the boring one that keeps being the answer: **separate vaults for separate concerns.** Application secrets in one vault, certificates in another.

---

### Part 7 — Tiers, Limits, and What This Costs

Short part, mostly so nothing surprises you later.

#### The Three Product Tiers

| | **Standard** | **Premium** 💳 | **Managed HSM** 💳 |
|---|---|---|---|
| **Key protection** | Software, FIPS 140-2 Level 1 | **HSM**, FIPS 140-2 Level 3 | Dedicated single-tenant HSM pool, FIPS 140-3 Level 3 |
| **Shared with other customers?** | Yes (multi-tenant service) | Yes | **No — hardware reserved for you** |
| **Secrets & certificates** | ✅ | ✅ | ❌ **Keys only** |
| **Cost** | No base fee; $0.03/10k ops | Same, **plus ~$1/month per HSM key** | **~$2,300+/month, always on** |
| **You'd choose it when** | Almost always | A regulation names FIPS 140-2 Level 3 | Regulation demands single-tenant hardware and full key sovereignty |

Two things to remember from that table. **Managed HSM is a completely separate resource type, not a tier of a vault, and it does not hold secrets or certificates — keys only.** And **you can upgrade Standard to Premium later, but you cannot downgrade.**

#### Limits Worth Knowing

Key Vault is **throttled per vault**, and this is the operational gotcha that produces the most confusing production incidents.

Roughly: **2,000 secret/certificate transactions per 10 seconds per vault**, with key operations having their own separate and often lower limits depending on key type. Exceed it and you get **HTTP 429 — Too Many Requests**.

Where this bites: an app that fetches its connection string from Key Vault **on every single request**. That works beautifully in testing with one user and falls over the moment real traffic arrives.

!!! tip "The mitigation is one word: cache"
    **Fetch secrets at startup, cache them in memory, and refresh on an interval.** Key Vault is a *secret store*, not a high-throughput configuration database, and it is not designed to sit in your per-request hot path.

    App Service's Key Vault references — which we build in Part 10 — do this for you automatically, caching values and refetching every 24 hours. That's not a limitation of the feature; it's the correct pattern, implemented for you.

    If one vault genuinely can't take your load, the answer is **more vaults** — which lines up neatly with the recommended practice anyway: **one vault per application, per environment.**

---

## PART TWO — ACCESS, INTEGRATION AND OPERATIONS

> **Halfway point.** You have a vault holding a real database connection string, an RSA key that cannot be read, and a certificate. Right now exactly one person can reach any of it: you. Everything from here is about opening that up safely — to a colleague, and then to an application — and then about running the thing properly once it's real.

---

### Part 8 — Key Vault RBAC Roles, and Why Owner Can't Read a Secret

#### The Roles

Key Vault's RBAC roles split along two lines: **which object type** (secrets, keys, certificates), and **read versus manage**. Learn the shape and you can predict the names.

| Role | What it grants | Plane |
|---|---|---|
| **Key Vault Contributor** | Manage the **vault** — create, delete, configure networking. **No data access whatsoever.** | Control |
| **Key Vault Reader** | Read vault metadata and **list** objects — names, versions, expiry dates. **Cannot read any value.** | Control-ish |
| **Key Vault Administrator** | **All data operations on everything** — secrets, keys, certificates. Cannot manage the vault resource or role assignments. | Data |
| **Key Vault Secrets User** | **Read secret values.** The workhorse role for applications. | Data |
| **Key Vault Secrets Officer** | Everything on secrets — create, update, delete, recover, purge | Data |
| **Key Vault Crypto User** | **Use** keys for cryptographic operations | Data |
| **Key Vault Crypto Officer** | Manage keys — create, rotate, delete | Data |
| **Key Vault Certificate User** | Read a certificate **including its private key** | Data |
| **Key Vault Certificates Officer** | Manage certificates | Data |
| **Key Vault Purge Operator** | Permanently delete soft-deleted vaults | Control |

Three patterns to lock in:

- **"User" reads and uses. "Officer" manages.** Once you've got that, you don't need to memorise the list.
- **Every data-plane role only works on vaults using the RBAC permission model.** Assign *Key Vault Secrets User* on an access-policy vault and it does nothing at all — silently. That's a genuinely nasty debugging session if you don't know it.
- **`Key Vault Contributor` is control plane only.** The docs state it flatly: it does not allow access to keys, secrets and certificates.

#### The Proof

Yesterday: a Contributor on a storage account could delete the account and could not read a blob in it. Today, the same demonstration, with higher stakes — and it's the version that shows up in real jobs, because vaults are exactly where the separation matters.

#### Hands-On: Prove Owner ≠ Secret Reader ✅

1. As admin, **Key vault → Access control (IAM) → Role assignments**. Find your own account — **Owner**, inherited from the subscription. **✅**
2. Go to **Objects → Secrets**. You can see `DbConnectionString` and read its value. **✅**

   Now here's the question, and answer it before you click: **is that because you're the Owner?**

3. **Access control (IAM) → + Add → Add role assignment → `Key Vault Secrets User` → Members: `grp-finance-team` → Review + assign.** **✅**
4. In the private window, sign Priya out and back in for a fresh token, and go to the vault. **✅**
5. **Objects → Secrets → `DbConnectionString` → current version → Show Secret Value.** **✅**

   **She reads it.** Priya has no Owner role, no Contributor role, nothing at subscription level. One data-plane role assignment, scoped to one vault, and she can read exactly the secrets in it — and nothing else in your entire Azure estate.

6. Have her click **Objects → Keys**. **Denied.** Secrets User is secrets only. **✅**
7. Now the reveal. Back as admin: **IAM → Role assignments**, find **your own** Owner assignment and read what it grants. Then go to **Objects → Secrets** and try to read the value again. **✅**

   **It still works — but not for the reason you think.** Owner includes `Microsoft.Authorization/*`, which means Owner can *grant itself* any Key Vault data role at any moment. The portal leans on that. Assign someone **Contributor** instead — which has no `roleAssignments/write` — and watch the difference: they can delete the entire vault, reconfigure its networking, change its permission model, and **the secret list comes back as an access error.**

    !!! warning "State this one precisely in an interview"
        The clean, correct sentence is: **"Control-plane roles like Contributor and Key Vault Contributor grant no data access. A subscription Owner can read secrets, but only because Owner can assign itself the data role — not because Owner includes data permissions."**

        That nuance is exactly what separates someone who has read a docs page from someone who has actually debugged a vault.

---

### Part 9 — Delegating Access, and the Contributor Back Door

#### The Back Door

I just alluded to it; now let's be blunt about it, because **this is the most important operational fact in today's video.**

**Anyone with Contributor on a key vault can give themselves full access to its data.**

Two routes, both trivial:

1. On an **access-policy** vault, Contributor includes the permission to write access policies. They add themselves. Done.
2. On an **RBAC** vault, Contributor includes `Microsoft.KeyVault/vaults/write` — so they can flip the permission model back to access policies and add themselves. (This is slightly harder now: the portal requires `roleAssignments/write` to change the model, specifically to stop people locking themselves out. But at the API layer, control-plane write on a vault is a very powerful permission.)

Microsoft's own documentation says it plainly: *"If a user has Contributor permissions to a key vault control plane, the user can grant themselves access to the data plane... You should tightly control who has Contributor role access to your key vaults."*

The practical consequences:

- **Never hand out Contributor at subscription or resource-group scope casually** if any vault lives underneath. That's not a vault permission in anyone's mental model, and it is one.
- Give vault operators **Key Vault Contributor** if they must manage vault settings — still powerful, but narrower than full Contributor.
- Give applications and people **data-plane roles at vault scope**, and nothing broader.
- Put production vaults in **their own resource group**, so resource-group-scoped Contributor assignments elsewhere can't reach them. This is the single most effective structural mitigation, and it costs nothing.

#### Delegating Without Handing Over the Keys

There's a purpose-built role for "let this person manage who can read secrets, but don't let them read secrets themselves": **Key Vault Data Access Administrator**.

It can add and remove role assignments — but **only the Key Vault ones** (Administrator, Secrets Officer/User, Crypto Officer/User, Certificates Officer, Reader). It cannot grant Owner. It cannot grant Contributor. It is constrained by a built-in **ABAC condition** — an attribute-based rule attached to the role definition that restricts *which* roles it may assign.

That's the pattern you want for a platform team: they onboard applications to vaults all day without ever being able to read a production secret or escalate themselves.

!!! note "ABAC in one paragraph, because it's new to this course"
    Everything in Day 17 was **RBAC** — principal, role, scope. **ABAC** adds a fourth element: a **condition**, evaluated at access time against attributes of the request. On Key Vault it currently applies to **secret data actions**, and lets you write rules like *"this principal may read secrets whose name starts with `dev-`."*

    You don't need to author these to pass an interview. You need to know the word, know it's a *condition attached to a role assignment*, and know **Key Vault Data Access Administrator** is the built-in role that ships with one.

#### Hands-On: Read the Constrained Role ✅

1. **Key vault → Access control (IAM) → Roles** tab. Filter for `Key Vault`. Count them — there are more than you'd guess. **✅**
2. Open **Key Vault Data Access Administrator** → **View** → **JSON**. Look at `Actions`: it has `roleAssignments/write`, but read the `conditions` — that's the ABAC rule limiting it to Key Vault roles only. **✅**
3. Open **Key Vault Contributor** → **JSON**. `Actions` covers `Microsoft.KeyVault/*`. Now look at **`DataActions`**. **✅**

   **It's empty.** The whole of Part 8 in one screenshot.

---

### Part 10 — The Payoff: An App That Reads a Secret It Cannot See

This is what the whole video has been building to, and it's the pattern you'll use in every real Azure project for the rest of your career.

#### What We're Building

```text
App Service                    Microsoft Entra ID              Key Vault
    │                                 │                            │
    │ 1. "I'm this managed identity,  │                            │
    │     give me a token for KV"     │                            │
    ├────────────────────────────────►│                            │
    │◄────────────────────────────────┤                            │
    │        access token             │                            │
    │                                                              │
    │ 2. GET /secrets/DbConnectionString   (Bearer <token>)         │
    ├─────────────────────────────────────────────────────────────►│
    │                                          3. Is this identity  │
    │                                             Key Vault Secrets │
    │                                             User on me? Yes.  │
    │◄─────────────────────────────────────────────────────────────┤
    │        the secret value                                       │
```

Count the credentials in that diagram. **Zero.** No password, no key, no connection string in any configuration file. The App Service proves who it is with an identity the platform manages, and the vault checks a role assignment. That's Day 17's managed identity and Day 17's RBAC, doing the job Day 18 built the vault for.

#### Hands-On, Step 1: Create the App and Give It an Identity ✅

We deleted last video's App Service at cleanup, so we need a fresh one.

1. **App Services → + Create → Web App.** **Resource group:** `rg-day18-demo`. **Name:** `app-lwm-day18-<yourname>`. **Publish:** Code. **Runtime:** any (.NET or Node is fine). **OS:** Linux. **Pricing plan: Free F1.** **Review + create → Create.** **✅**
2. Open it → **Settings → Identity → System assigned → Status: On → Save → Yes.** **✅**
3. Copy the **Object (principal) ID** that appears. Entra ID has just created a service principal for this app. **✅**

#### Hands-On, Step 2: Grant It Exactly One Role ✅

4. **Key vault → Access control (IAM) → + Add → Add role assignment.** **✅**
5. **Role:** `Key Vault Secrets User`. **Next.** **✅**
6. **Members:** change **Assign access to** from *User, group, or service principal* to **Managed identity** → **+ Select members** → **Managed identity: App Service** → pick `app-lwm-day18-<yourname>`. **✅**
7. **Review + assign.** **✅**

   Note what you did not do: you did not create a credential, copy a client ID, or store a password. **The role assignment *is* the integration.**

    !!! tip "Least privilege, on camera"
        We picked *Key Vault Secrets User*, not *Key Vault Administrator* and definitely not *Contributor*. The app reads one secret; it gets exactly the permission to read secrets, at exactly the scope of one vault. If this app is ever compromised, that's the entire blast radius.

#### Hands-On, Step 3: The Key Vault Reference ✅

Here's the elegant part. App Service understands a special syntax in app settings: put a **Key Vault reference** in as the value, and the platform resolves it at runtime, using the app's managed identity, before your code ever sees it. **Your application code reads a normal environment variable and has no idea Key Vault is involved.** No SDK, no library, no code change.

8. **App Service → Settings → Environment variables → App settings → + Add.** **✅**
9. **Name:** `DbConnection`. **Value** — one of these two forms: **✅**

   ```text
   @Microsoft.KeyVault(SecretUri=https://kv-lwm-day18-<yourname>.vault.azure.net/secrets/DbConnectionString)
   ```

   or the equivalent, which many people find easier to read:

   ```text
   @Microsoft.KeyVault(VaultName=kv-lwm-day18-<yourname>;SecretName=DbConnectionString)
   ```

   **Leave the version off**, so the app follows the current version and picks up rotations automatically.

10. **Apply → Apply** to save. The app restarts. **✅**
11. Wait a moment, then look at the app settings list. Next to `DbConnection` you should see a **green tick and the source shown as "Key Vault Reference"**. **✅**

    **That green tick is the whole video.** It means App Service authenticated as the managed identity, called the vault, was authorised by your role assignment, and got the secret. There is no credential anywhere in this configuration — click **Edit** on the setting and the only thing stored is the reference string itself.

#### When It Doesn't Work

It will fail for someone watching this, so here's the debugging path in the order you should try it:

| Symptom | Cause | Fix |
|---|---|---|
| Red X, "Key Vault reference not resolved" | The managed identity has no role on the vault, or it hasn't propagated | Check the vault's IAM → Role assignments. Wait 10 minutes. |
| No status icon at all, value shows literally | **Syntax error** — the portal didn't even recognise it as a reference | Check the `@Microsoft.KeyVault(...)` spelling exactly, and that there are no spaces |
| Worked yesterday, broken today | Secret expired, disabled, or deleted | Check the secret's status in the vault |
| Works locally, fails in the app | Vault firewall is on and the app isn't allowed through | Part 11 |

Two portal tools worth knowing: **Edit the setting** and the dialog shows the live resolution status including the error, and **Diagnose and solve problems → Availability and Performance → Web app down → Key Vault Application Settings Diagnostics** runs a purpose-built detector.

!!! note "How rotation actually behaves"
    App Service **caches** resolved references and refetches them roughly **every 24 hours**. So rotating a secret in the vault does not instantly change what your running app sees.

    If you need it immediately, **any configuration change to the app forces a restart and an immediate refetch** — that's the practical trick. There's also a documented REST endpoint to force resolution without a restart.

    This caching is a feature, not a limitation. It's exactly the caching Part 7 said you should be doing, implemented for you.

!!! tip "The same pattern, everywhere"
    Key Vault references work identically in **Azure Functions** and **Logic Apps (Standard)**. **AKS** does the equivalent with the Secrets Store CSI driver. **Azure DevOps** pipelines do it with a variable group linked to a vault — that's Day 22. **Data Factory, App Configuration, API Management** — all the same shape: managed identity, data-plane role, reference instead of value.

    Learn it once here and you've learned it for the whole platform.

---

### Part 11 — Networking: Firewall, Trusted Services and Private Endpoints

Right now your vault accepts connections from **anywhere on the internet**. Authentication and RBAC still protect it — an attacker with no valid token gets nothing — but the front door is publicly reachable, which means it's reachable by password sprays, by misconfigured scripts, and by anyone who finds the hostname.

#### Three Levels of Network Lockdown

**Level 1 — Public, all networks.** The default. Fine for learning.

**Level 2 — Firewall with selected networks.** The vault rejects traffic except from IP ranges you list and virtual network subnets you allow (via **service endpoints**). Traffic still goes over the Azure backbone to a public endpoint, but the vault refuses everyone else. **Free.** ✅

**Level 3 — Private endpoint.** The vault gets a **private IP address inside your VNet**. Traffic never touches the public internet at all, and you can disable public access entirely. This is the production answer for anything sensitive. 💳 — about **$7.30/month** plus data processing, so it's an instructor demo.

You met all three concepts on Day 11 with storage accounts. **Key Vault behaves identically**, which is the point — Azure's networking model is consistent across services, so learning it once transfers.

#### The Trap Everyone Falls Into

Turn on the firewall and a long list of things breaks at once, because plenty of Azure services need to reach your vault and they don't come from an IP address you can list.

That's what **"Allow trusted Microsoft services to bypass this firewall"** is for. It lets specifically-listed first-party services through — App Service certificate deployment, Azure Backup, Disk Encryption, Event Grid, Azure SQL for customer-managed keys.

!!! warning "Trusted services does NOT cover Key Vault references"
    This is the one to remember, because it wastes entire afternoons.

    **App Service Key Vault references are not covered by the trusted-services bypass.** If you enable the vault firewall, the app we just built in Part 10 stops resolving its secret — and the error message will not tell you it's a networking problem.

    The supported fix is **VNet integration on the app plus a service endpoint or private endpoint on the vault** — which requires a Standard or higher App Service plan, not Free F1. That's why we are **not** enabling the firewall on our vault today: it would break the demo we just built, on a plan tier that can't fix it.

    Know the constraint. It's a very common real-world design conversation.

#### Hands-On: Look, Don't Touch ✅

1. **Key vault → Settings → Networking → Firewalls and virtual networks.** **✅**
2. Click **Selected networks** to reveal the options — the **+ Add existing virtual network** control, the **Firewall / IP address ranges** box with a helpful *Add your client IP address* link, and the **Exception: Allow trusted Microsoft services** checkbox. **✅**
3. **Set it back to All networks and don't save any change**, for the reason in the warning above. **✅**
4. Click the **Private endpoint connections** tab and read it. 💳 In production this is where you'd create the private endpoint, and it's how a serious environment runs a vault. **✅**

!!! tip "The exam-style answer"
    *"Key Vault must not be reachable from the public internet"* → **private endpoint**, and set public network access to **Disabled**.

    *"Only our corporate office and our app subnet may reach the vault"* → **firewall with IP rules and a service endpoint**.

    *"Azure Backup can't reach the vault after we enabled the firewall"* → **allow trusted Microsoft services**.

---

### Part 12 — Soft Delete, Purge Protection and Recovery

#### Soft Delete Is Not Optional Any More

**Soft delete is mandatory on all key vaults and cannot be turned off.** It was optional years ago; it isn't now. Microsoft enabled it across all vaults, and every vault you create today has it.

What it does: when you delete a vault or an object inside it, the thing enters a **deleted state** rather than vanishing. During the retention window (your 7–90 days from Part 2) it is invisible in normal listings but fully recoverable — and critically, **recovery restores the same object, with the same identifiers**, so applications pointing at it start working again.

Two things it costs you, both of which surprise people:

- **The name is reserved.** You cannot create a new vault with a deleted vault's name until it's purged or the window expires. Global uniqueness applies to deleted vaults too.
- **You may still be billed for the storage of a soft-deleted vault's contents** during the retention window.

#### Purge Protection: The Switch With No Off

Covered in Part 2, but now you've seen the mechanics, the point should land harder. Without purge protection, an admin can delete *and immediately purge* a vault — genuinely gone, keys and all. With purge protection, the deletion always waits out the full retention window, and **no permission level can shortcut it**.

The attack it defends against is specific: an attacker with admin access destroys the customer-managed keys that encrypt your storage and databases. The data is still there and permanently unreadable. Purge protection means the keys are recoverable for up to 90 days, which is enough time to notice and respond.

**Turn it on in production. Especially if the vault holds encryption keys — where it is a hard requirement, not a suggestion.**

#### Hands-On: Delete and Recover ✅

1. **Key vault → Objects → Secrets.** Create a throwaway: **+ Generate/Import**, name `TempSecret`, any value, **Create**. **✅**
2. Select it → **Delete**. Confirm. It disappears from the list. **✅**
3. Click **Manage deleted secrets** at the top of the Secrets blade. **✅**

   **There it is** — with the deletion date and the scheduled purge date.

4. Select it → **Recover**. Go back to the Secrets list. **It's back, same name, same versions, same identifiers.** **✅**
5. Delete it again, go back into **Manage deleted secrets**, and this time select it and click **Purge**. **Now it is genuinely gone.** **✅**

   Notice that Purge was available — because we left purge protection **off**. With it on, that button would be there and would refuse. That's the difference in one screen.

6. Bonus: search **Key vaults** in the portal and click **Manage deleted vaults** on the toolbar. Any vault you've soft-deleted in this subscription is listed there. **This is the blade to remember when Azure tells you a vault name is taken and you're certain it isn't.** **✅**

!!! tip "The interview question, and it's asked a lot"
    *"You deleted a key vault by accident. How do you get it back?"* → **Soft delete. It's on by default and mandatory, retention is 7–90 days, recover it from Manage deleted vaults, and it comes back with the same URI and contents.**

    Follow-up: *"What if purge protection was enabled?"* → **Then it couldn't have been permanently purged in the first place — it's guaranteed recoverable for the full retention period.**

---

### Part 13 — Monitoring, Expiry and Rotation

A vault that nobody watches is a vault that fails silently, and Key Vault fails in two specific ways: **secrets expire**, and **access is denied**.

#### Logging

**Key Vault does not log data-plane operations by default.** You turn it on with a **diagnostic setting**, exactly as you would for any other Azure service, sending `AuditEvent` logs to a **Log Analytics workspace**.

Once that's on, every operation is recorded: who, what object, what operation, from which IP, and whether it succeeded. That's the audit trail that answers *"who read the production connection string last Tuesday?"* — and it's a question you cannot answer retroactively, so this is a thing to switch on **before** you need it, not after.

That's Day 19's territory, and Key Vault is the perfect motivation for it.

#### Expiry: The Silent Outage

Part 4 warned about this; here's the fix. Key Vault raises **Event Grid events** for object lifecycle:

- **`SecretNearExpiry`** — fires **30 days before** the expiration date by default
- **`SecretExpired`** — fires when it lapses
- **`SecretNewVersionCreated`** — the hook for rotation automation
- Equivalent events exist for keys and certificates

Two prerequisites nobody mentions and both matter: **the object must have an expiration date set** (no date, no event, ever), and **Key Vault evaluates near-expiry on a periodic sweep**, so the first event can take up to 24 hours to appear.

#### Rotation

Recall the distinction from Part 5, because it's the crisp answer to a common question:

- **Keys rotate themselves.** Set a rotation policy; Key Vault generates new versions on schedule. No code.
- **Secrets cannot rotate themselves.** Key Vault has no idea your secret is a SQL password — changing it means changing it in SQL Server too. That's application-specific, so the standard pattern is: `SecretNearExpiry` → Event Grid → **Azure Function** → change the credential at the source → write the new version back to the vault. Your apps, referencing without a version, pick it up automatically.

Which brings the day full circle: **because the secret lives in exactly one place, rotating it is one operation instead of a hunt through five repositories.** That's the payoff for all of today's discipline.

#### Hands-On: Turn On the Audit Trail ✅

1. **Key vault → Monitoring → Diagnostic settings → + Add diagnostic setting.** **✅**
2. **Name:** `kv-audit`. **Logs:** tick **Audit Logs** (`AuditEvent`). **Destination:** *Send to Log Analytics workspace* — create one in `rg-day18-demo` if you don't have one. **Save**. **✅**

    !!! note "This one has a real, if small, cost"
        Log Analytics ingestion is billed per GB with a free monthly allowance, and Key Vault audit logs are tiny. For this lab it's effectively free — but it is **not** zero, and Day 19 covers the cost model properly. If you want to be strict about spending, delete this diagnostic setting at cleanup.

3. Go and do a few operations — read a secret, list the keys. Then **Monitoring → Logs** and run: **✅**

   ```kusto
   AzureDiagnostics
   | where ResourceType == "VAULTS"
   | project TimeGenerated, OperationName, identity_claim_upn_s, ResultSignature
   | order by TimeGenerated desc
   ```

   Logs can take a few minutes to arrive. When they do, **there's your audit trail** — every access to the vault, with the identity that made it. Find the App Service's managed identity fetching `DbConnectionString`. That's your Part 10 demo, recorded.

4. Open **Monitoring → Alerts** and note you could alert on `ServiceApiResult` — the standard alert for *someone is being denied access to the vault*, which is either a misconfiguration or an intrusion, and you want to know either way. **✅**

---

### Part 14 — Defender for Cloud, Sentinel and Interview Prep

We finish with the layer above everything: the tooling that watches your whole subscription and tells you what you got wrong — including, as you'll see, a few things from today.

#### Microsoft Defender for Cloud

**Defender for Cloud** is Azure's security posture and threat-protection service. It has two halves, and the licensing question is the one interviewers ask.

**Foundational CSPM — free.** *Cloud Security Posture Management*. It continuously assesses your resources against the **Microsoft Cloud Security Benchmark** and gives you:

- **Secure Score** — a single percentage for how well your environment follows security best practice
- **Security recommendations** — a prioritised, actionable list, each weighted by how much it would raise your score
- **Asset inventory** — every resource, with its security state
- **Regulatory compliance** views against standards like ISO 27001, PCI-DSS and CIS

**Defender plans — paid.** 💳 Per-resource-type threat protection: Defender for Servers, for Storage, for SQL, for Containers, for Key Vault. These do actual **threat detection** — alerting on anomalous access to a vault, malware uploaded to a storage account, suspicious SQL activity. Plus **Defender CSPM**, the paid posture tier, which adds attack path analysis and agentless vulnerability scanning.

The distinction to hold: **free CSPM tells you what's misconfigured. Paid Defender plans tell you you're being attacked.**

!!! warning "This changes on 27 October 2026 — worth knowing right now"
    **From 27 October 2026, Foundational CSPM moves to an opt-in model and is no longer enabled by default on new Azure subscriptions.**

    It stays free and you can enable it at any time — but the default changes. So a subscription created after that date will show you *nothing* until someone switches it on, and "we have no security recommendations" will mean "we aren't looking," not "we're fine."

    If you're watching this after that date and the next demo shows an empty dashboard, that's why. Enable the free plan and come back in a few hours.

#### Microsoft Sentinel — Vocabulary Only

**Sentinel is Azure's SIEM and SOAR** — Security Information and Event Management, and Security Orchestration, Automation and Response. It sits on top of a Log Analytics workspace, ingests logs from across your estate (including the Key Vault audit logs you just enabled), correlates them, hunts for threats, and can run automated playbooks in response.

It is **consumption-priced on data ingestion** and gets expensive quickly, so this is genuinely vocabulary today. The one distinction to be able to draw:

**Defender for Cloud protects your Azure resources. Sentinel is a SIEM for your whole organisation** — Azure, on-premises, other clouds, Microsoft 365 — and it's where a security operations centre actually works.

#### Hands-On: Read Your Secure Score ✅

1. Search **Microsoft Defender for Cloud** → **Overview**. **✅**
2. Look at your **Secure Score**. It won't be great, and that's the point. **✅**
3. **Recommendations** → filter for `Key Vault`. You will very likely see several of today's decisions come back at you: **✅**
   - *Key vaults should have deletion protection enabled* — that's purge protection, which we deliberately left off
   - *Key vaults should have soft delete enabled* — this one you'll pass, because it's mandatory now
   - *Azure Key Vault should use RBAC permission model* — pass, because it's the default
   - *Private endpoint should be configured for Key Vault* — fail, and now you know exactly what it costs to fix
4. Open one recommendation and read its structure: description, affected resources, remediation steps, and often a **Fix** button that applies the change directly. **✅**
5. Click **Regulatory compliance** and look at your subscription against a benchmark. **✅** 💳 *Some standards need a paid Defender plan; the default benchmark view is free.*

    !!! tip "Secure Score is a genuinely good learning tool"
        Work down that recommendations list on your own subscription and you'll learn more practical Azure security than from any course, including this one — because it's specific to things you actually built, and each item explains itself.

        In interviews, *"how would you assess the security posture of an Azure environment you've just inherited?"* has a very strong opening: **"Start with Defender for Cloud's Secure Score and work the recommendations by weight."**

#### Interview and Exam Quick Reference

| If you're asked… | The answer is… |
|---|---|
| "Where do application secrets belong?" | **Key Vault** — never in code, config, or app settings |
| "Secret vs key vs certificate?" | **Secrets come back to you; keys never leave the vault (it does the crypto); certificates are managed X.509 objects** |
| "Which permission model for a new vault?" | **Azure RBAC** — the default since API version 2026-02-01 |
| "Deadline for Key Vault control-plane APIs" | **27 February 2027** — everything before 2026-02-01 retires |
| "App must read a secret with no stored credential" | **Managed identity + Key Vault Secrets User + Key Vault reference** |
| "Minimum role for an app to read secrets" | **Key Vault Secrets User** — not Administrator, not Contributor |
| "User has Contributor on the vault but can't read secrets" | **Correct — control plane only.** Assign a data-plane role |
| "…but they granted themselves access anyway" | **The Contributor back door.** Contributor can write vault config / access policies. Restrict it |
| "Manage who can read secrets, without being able to read them" | **Key Vault Data Access Administrator** (constrained by an ABAC condition) |
| "Recover an accidentally deleted vault" | **Soft delete** — mandatory, 7–90 days, *Manage deleted vaults* |
| "Guarantee nobody can permanently destroy the vault" | **Purge protection** — irreversible once enabled |
| "Vault name is taken but I can't see the vault" | It's **soft-deleted**. Purge it or wait out retention |
| "Required for customer-managed encryption keys" | **Soft delete + purge protection** |
| "Regulation demands FIPS 140-2 Level 3" | **Premium tier** (HSM-backed keys) |
| "Single-tenant dedicated HSM, keys only" | **Managed HSM** — separate resource, ~$2,300+/month |
| "Vault must not be reachable from the internet" | **Private endpoint** + public access disabled |
| "Azure Backup broke after enabling the vault firewall" | **Allow trusted Microsoft services** |
| "App Service Key Vault reference broke after enabling the firewall" | Trusted services **doesn't** cover it — needs **VNet integration + service/private endpoint** |
| "HTTP 429 from Key Vault under load" | **Throttling.** Cache secrets; don't fetch per request; split vaults |
| "Be warned before a secret expires" | **Event Grid `SecretNearExpiry`** — 30 days by default, and the object must have an expiry date |
| "Who read this secret last month?" | **Diagnostic settings → AuditEvent → Log Analytics.** Not on by default |
| "Rotate a database password automatically" | **Event Grid → Azure Function → update source → new version.** Keys self-rotate; secrets don't |
| "Assess security posture of an unfamiliar subscription" | **Defender for Cloud Secure Score + recommendations** (free Foundational CSPM) |
| "Free vs paid Defender for Cloud" | **Free = posture (Secure Score, recommendations). Paid = threat detection** |
| "SIEM across Azure, on-prem and M365" | **Microsoft Sentinel** |

---

## Cleanup ✅

1. **Delete the diagnostic setting** if you'd rather not ingest any Log Analytics data: **Key vault → Diagnostic settings → `kv-audit` → Delete**. **✅**
2. **Remove the App Service's role assignment** — good hygiene, and it stops an orphaned *Identity not found* entry appearing later: **Key vault → IAM → Role assignments → `app-lwm-day18-…` → Remove**. **✅**
3. **Delete the resource group `rg-day18-demo`.** This removes the vault, the App Service, the plan and the Log Analytics workspace in one action. **✅**
4. **The vault is now soft-deleted, not gone.** Search **Key vaults → Manage deleted vaults**, find it, and **Purge** it — otherwise its name stays reserved for 90 days and it may keep billing you for stored content. **✅**

    !!! tip "This step is the practical lesson of Part 12"
        Deleting the resource group felt like cleanup, and it left something behind that you can only see from a different blade. That is exactly how people end up unable to reuse a vault name six weeks later. **Always purge test vaults deliberately.**

5. **Keep `Priya Sharma` and `grp-finance-team`.** Still free, and the Capstone on Day 30 uses them again. **✅**
6. **Keep `db-lwm-demo`** — Day 30 needs it. **✅**

---

## Summary

Today started with a question we ducked yesterday: *where does the secret actually live?*

**Key Vault is the answer, and its value is that there's exactly one copy.** Not a copy in a repo, a copy in a config file and a copy in someone's Slack history — one object, in one place, with an identity-based access decision in front of it and an audit log behind it. Everything else today followed from that.

**Three object types, and the difference is real.** Secrets come back to you. **Keys never leave** — you send the vault data and it returns a signature, so even a fully compromised application server never holds your private key. Certificates are managed X.509 objects that renew themselves, and quietly exist as three addressable objects at once, which is an access-control trap worth remembering.

**The permission model is the decision that changed.** As of API version **2026-02-01**, Azure RBAC is the default for new vaults, and every control-plane API version before it **retires on 27 February 2027**. Access policies still work, but they're per-vault, don't inherit, and none of Azure's access tooling understands them. Choose RBAC — and if you're migrating a live vault, **assign the roles before you flip the switch**, because flipping it invalidates every access policy instantly.

Then yesterday's lesson came back with higher stakes. **Key Vault Contributor manages the vault and cannot read one secret** — the same control-plane/data-plane split you proved on a storage account, in the place it matters most. And its dangerous corollary: **Contributor on a vault can grant itself data access**, which is why production vaults belong in their own resource group and why *Key Vault Data Access Administrator* exists.

And then the payoff, which is the pattern you'll reuse constantly: **an App Service with a managed identity, one *Key Vault Secrets User* role assignment, and a `@Microsoft.KeyVault(...)` reference in an app setting.** A running application reading a database password from a vault, with **no credential anywhere in its configuration** and no code that knows Key Vault exists. Day 17 gave the app an identity; Day 18 gave it something worth protecting.

Finally the operational reality: **soft delete is mandatory and saves you**, **purge protection is permanent and protects you**, the retention period **can never be changed**, a deleted vault **keeps its name reserved**, the vault **won't tell you a secret is about to expire** unless you wire up the events, and **it doesn't log anything** until you turn on diagnostics. Every one of those is a lesson somebody learned the expensive way.

### What's Next

**Day 19 — Azure Monitor & Alerts.** We hit its edges three times today: the Log Analytics workspace behind those audit logs, the alert on denied vault access, and the Event Grid notification for an expiring secret. Every one of those is a monitoring question we answered in one sentence and moved past.

Day 19 does it properly — metrics, logs, KQL, alert rules, action groups, Application Insights and workbooks — and by the end you'll know not just how to build things in Azure, but how to find out when they break. Which, on the day it matters, is the more valuable skill.

---

## Key Takeaways

**The vault and its contents**

- **Never put secrets in code, config files, or plain app settings.** Key Vault gives you one copy, identity-based access, and a full audit trail.
- **Secrets return values. Keys never leave the vault — it performs the cryptography for you. Certificates are managed X.509 objects with automatic renewal.**
- A certificate is **three addressable objects** — certificate, key and secret. *Key Vault Secrets User* can therefore read certificate private keys. **Separate your vaults.**
- **Storing costs nothing; operations cost $0.03 per 10,000.** Certificate renewals through an integrated CA are **$3 each**. Managed HSM is **~$2,300+/month** and holds keys only.
- **One vault per application, per environment.** It's the recommended practice and it's also your defence against per-vault throttling.
- **Cache secrets — never fetch per request.** HTTP 429 under load is the classic beginner architecture failure.

**Access**

- **Azure RBAC is the default permission model** for vaults created with API version **2026-02-01** and later. All earlier control-plane API versions **retire on 27 February 2027**.
- **Switching a live vault to RBAC invalidates every access policy immediately.** Assign roles first, then switch.
- **"User" roles read and use; "Officer" roles manage.** *Key Vault Secrets User* is the right role for an application.
- **Key Vault Contributor has no data access at all.** A subscription Owner can read secrets only because Owner can assign itself the data role.
- **The Contributor back door is real:** control-plane write on a vault can be turned into data access. Restrict Contributor, and isolate production vaults in their own resource group.
- **Key Vault Data Access Administrator** delegates vault access management without granting data access — constrained by a built-in **ABAC** condition.

**Integration and operations**

- **Managed identity + data-plane role + Key Vault reference** is the pattern. `@Microsoft.KeyVault(SecretUri=…)` or `@Microsoft.KeyVault(VaultName=…;SecretName=…)`, no version if you want automatic rotation pickup.
- App Service **caches references for ~24 hours**; any config change forces an immediate refetch.
- **Trusted Microsoft services does not cover App Service Key Vault references** — that combination needs VNet integration plus a service or private endpoint.
- **Soft delete is mandatory and cannot be disabled.** Retention is **7–90 days and can never be changed after creation**. A soft-deleted vault **keeps its name reserved**.
- **Purge protection cannot be undone by anyone, including Microsoft.** Required for customer-managed encryption keys. Enable it in production; think twice on a test vault.
- **Key Vault logs nothing until you add a diagnostic setting**, and **warns you about nothing** until you wire up Event Grid `SecretNearExpiry` — which needs an expiry date to exist in the first place.
- **Keys can rotate themselves; secrets need an Azure Function** to change the credential at its source.
- **Defender for Cloud's Foundational CSPM is free** (Secure Score, recommendations) and **becomes opt-in for new subscriptions from 27 October 2026**. Paid Defender plans add threat detection. **Sentinel is the SIEM.**
