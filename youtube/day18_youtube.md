# Day 18 — YouTube Metadata

---

## Video Title

Azure Key Vault for Beginners: Secrets, Keys, Certificates & Managed Identity (Hands-On) | Day 18

---

## Thumbnail

**Main text (large, bold):** `No Password. Still In.`
**Sub text:** `Day 18 — Azure Key Vault`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- Left: an `appsettings.json` file showing `Password=Winter2026` with a big red cross through it
- Centre: a steel vault door with the Key Vault icon, slightly open
- Right: an App Service icon with a green tick and the label "Key Vault Reference"
- Small corner badge: "Lab cost ≈ $0"
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** The app reads its database password from a vault without holding any credential at all. The secret moves out of your code and config and into one place.

---

## Description

*Welcome back to Learn With Mithran! Yesterday we created a client secret, and I said it was a password to your tenant that was now sitting in your Cloud Shell history. Today we answer the question that left open: where should a secret actually live?*

By the end of this video you'll have a running web app that reads a database password it cannot see, from a vault it has no credential for, without a single secret typed into a single configuration screen. Part One builds the vault: secrets vs keys vs certificates, every option on the create blade (including the two that are **permanent**), and the permission model that changed in February 2026. Part Two opens it up safely: Key Vault RBAC roles, the Contributor back door, managed identity + Key Vault references, networking, soft delete, purge protection, monitoring and Defender for Cloud.

💰 Key Vault isn't free tier, but storing objects costs nothing and operations are $0.03 per 10,000, so today's lab rounds to zero. Premium, Managed HSM and private endpoints are explained, not clicked. 🔐

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

☁️ AWS playlist: https://youtube.com/playlist?list=PLPLf8iqkntdMxtXT04-TG1WzDvBPUJ3qk
🛠️ DevOps playlist: https://youtube.com/playlist?list=PLPLf8iqkntdNaU9GbaZckoQalKPRJMvT6
🧠 Python playlist: https://youtube.com/playlist?list=PLPLf8iqkntdNefseVlDOaRQ7zersK79AI
🐧 Linux playlist: https://youtube.com/playlist?list=PLPLf8iqkntdMew0yP5Ad9pbaZki0Wf-2w

🎯 *Topics Covered*:

🔹 Why secrets in code, config files and app settings all fail the same way
🔹 Secrets vs keys vs certificates, and why a key can never be read back
🔹 The two create-blade settings you can never change
🔹 RBAC vs access policies: the February 2026 default and the 27 Feb 2027 API deadline
🔹 Secret versions, expiry dates and the disable switch
🔹 One certificate = three objects, and the access trap that creates
🔹 Standard vs Premium vs Managed HSM, and HTTP 429 throttling
🔹 Proof: Key Vault Contributor cannot read a single secret
🔹 The Contributor back door, and Key Vault Data Access Administrator (ABAC)
🔹 Demo: App Service + managed identity + Key Vault reference, zero credentials
🔹 Firewall, trusted services, private endpoints, and what breaks references
🔹 Soft delete and purge protection: delete, recover, purge
🔹 Audit logs with KQL, SecretNearExpiry events, and rotation
🔹 Defender for Cloud Secure Score and a full interview keyword table

📌 *Who Is This Video For:*

💻 Beginners who have passwords and API keys sitting in config files
🧑‍🎓 AZ-104, AZ-500 and AZ-305 candidates
🔥 Developers who want apps with no stored credentials
🚀 Anyone preparing for cloud interviews

🔍 *Chapters:*
0:00 Intro: Where Should a Secret Live?
4:00 What Today Costs, and the Two Permanent Settings
9:00 Part 1: Why Key Vault Exists
14:00 Secrets vs Keys vs Certificates
18:00 Part 2: Create Your First Vault
24:00 Part 3: RBAC vs Access Policies, and the 2027 Deadline
33:00 Part 4: Secrets, Versions and Expiry
40:00 Part 5: Keys, Cryptography as a Service
47:00 Part 6: Certificates, One Object Is Three
53:00 Part 7: Tiers, Limits and Throttling
57:00 Part 8: Key Vault RBAC Roles
61:00 Demo: Priya Reads a Secret, Contributor Can't
65:00 Part 9: The Contributor Back Door
71:00 Part 10: App Service + Managed Identity
76:00 Demo: The Key Vault Reference Green Tick
81:00 Part 11: Firewall and Private Endpoints
87:00 Part 12: Soft Delete and Purge Protection
93:00 Part 13: Audit Logs, Expiry and Rotation
99:00 Part 14: Defender for Cloud and Sentinel
105:00 Interview and Exam Quick Reference
111:00 Cleanup: Purge Your Vault
113:00 Summary and Key Takeaways

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #AzureKeyVault #KeyVault #ManagedIdentity #AzureRBAC #AzureSecurity #DefenderForCloud #AZ104 #AZ500 #AzureForBeginners #MicrosoftAzure #AzureTutorial #LearnWithMithran #GreensTechnologies

---

## Tags

azure key vault tutorial, azure key vault, key vault secrets, key vault rbac, access policy vs rbac, key vault reference, managed identity key vault, key vault secrets user, soft delete purge protection, key vault certificates, key vault firewall, azure secrets management, azure key vault interview questions, defender for cloud, az-104, az-500, az-305, azure for beginners, azure tutorial, microsoft azure, learnwithmithran
