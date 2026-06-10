# Day 7 — YouTube Metadata

---

## Video Title

Azure Blob Storage Deep Dive: Access Tiers, SAS Tokens, Static Website Hosting & Versioning | Day 7 | LearnWithMithran

---

## Thumbnail

**Main text (large, bold):** `Blob Storage`
**Sub text:** `Day 7 — Store Anything in Azure`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- Azure Storage Account icon (top left)
- Four service icons (Blob, Files, Queue, Table) — Blob highlighted, others greyed out to signal "today = Blob"
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** One storage account. Store any file. Control who sees it and for how long.

---

## Description

*Welcome back to Learn With Mithran! Every application needs somewhere to put its data — images, backups, log files, documents. In Azure, the answer is a Storage Account. Today we go deep on the most-used part: Blob Storage.*

In this session we cover what an Azure Storage Account is, how replication options (LRS, ZRS, GRS, GZRS) protect your data at different levels, and everything inside Blob Storage — containers, the three blob types (Block, Append, Page), the four access tiers (Hot, Cool, Cold, Archive), Shared Access Signatures for time-limited secure sharing, lifecycle management policies that automatically move data as it ages, Static Website Hosting to serve an entire frontend with no server, and Blob Versioning to automatically save and restore previous copies of any file. Azure Files, Queue Storage, Table Storage, and Storage Explorer are covered in the next video. 🚀

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

🔹 What is an Azure Storage Account — one account, four storage services, one bill
🔹 The unified namespace — why the account name must be globally unique
🔹 Performance tiers — Standard (HDD) vs Premium (SSD) and when to use each
🔹 Replication options — LRS, ZRS, GRS, GZRS and what failure each protects against
🔹 Demo — Create a Storage Account with LRS redundancy
🔹 What is Blob Storage — Binary Large Objects, object storage for any unstructured data
🔹 Containers and blobs — the two-level structure and how blob URLs are formed
🔹 Public access levels — Private, Blob, and Container — what each allows
🔹 Three blob types — Block Blob (general files), Append Blob (logs), Page Blob (VM disks)
🔹 Demo — Create a container and upload a file
🔹 Why a private blob URL returns an error in the browser — and why that's correct
🔹 Access tiers — Hot, Cool, Cold, Archive — how storage cost and access cost trade off
🔹 Archive tier rehydration — why it takes up to 15 hours and when never to archive data
🔹 Demo — Change a blob's access tier from Hot to Cool in the portal
🔹 Lifecycle Management Policies — JSON rules that automatically transition or delete blobs based on age
🔹 Demo — Create a lifecycle rule: Cool at 30 days → Archive at 90 days → Delete at 365 days
🔹 What is a Shared Access Signature (SAS) — time-limited, permission-scoped URLs
🔹 SAS parameters — expiry, resource type, permissions, cryptographic signature
🔹 What happens when you tamper with a SAS URL — AuthenticationFailed explained
🔹 Demo — Generate a read-only SAS URL for a blob and test it in the browser
🔹 Static Website Hosting — serve HTML, CSS, and JavaScript directly from a blob container with no server
🔹 The $web container — how Azure creates and serves the public endpoint automatically
🔹 When to use static hosting vs App Service — the right tool for each scenario
🔹 Demo — Enable Static Website Hosting and publish a live HTML page from Blob Storage
🔹 Blob Versioning — how Azure automatically saves a copy every time a blob is overwritten
🔹 Version IDs — how each version is identified and how to browse full file history
🔹 Soft Delete vs Versioning — what each protects against and why you should enable both
🔹 Demo — Enable Versioning, overwrite a file, and restore the previous version in one click
🔹 What's coming next — Azure Files, Queue Storage, Table Storage, and Storage Explorer

📌 *Who Is This Video For:*

💻 Complete beginners who want to understand where Azure applications store their data
🧑‍🎓 Students preparing for AZ-900 or AZ-104 certification
☁️ Developers who need to store files, images, or backups from an Azure application
🌐 Frontend developers who want to host a static site or React app with no server cost
🖥️ IT professionals moving file storage workloads to the cloud
🚀 Anyone building toward a production-ready Azure application

🔍 *Chapters:*
0:00 Intro — Why Every Application Needs Cloud Storage
2:00 What Is an Azure Storage Account — The Unified Namespace
8:00 Performance Tiers — Standard vs Premium
12:00 Replication Options — LRS, ZRS, GRS, GZRS Explained
20:00 Demo — Create a Storage Account in the Portal
28:00 What Is Blob Storage — Containers, Blobs, and URLs
34:00 Three Blob Types — Block, Append, and Page
40:00 Demo — Create a Container and Upload a File
46:00 Access Tiers — Hot, Cool, Cold, Archive — Cost vs Speed Trade-Off
54:00 Demo — Change a Blob's Access Tier
58:00 Lifecycle Management — Automate Tier Transitions and Deletion
64:00 Demo — Create a Lifecycle Policy Rule
70:00 Shared Access Signatures — Time-Limited Secure File Sharing
78:00 Demo — Generate a SAS URL and Test Tamper Protection
84:00 Static Website Hosting — Serve a Site Directly From Blob Storage
92:00 Demo — Enable Static Website Hosting and Publish a Live Page
100:00 Blob Versioning — Protect Every File Against Accidental Overwrites
106:00 Demo — Enable Versioning and Restore a Previous Version
114:00 Cleanup — Delete the Resource Group
116:00 Summary and What's Next (Files, Queue, Tables & Storage Explorer)

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #AzureStorage #BlobStorage #AzureStorageAccount #SASToken #SharedAccessSignature #AzureLifecycleManagement #AzureAccessTiers #HotCoolArchive #AzureStaticWebsite #BlobVersioning #AzureDataProtection #BlockBlob #AppendBlob #AzureLRS #AzureGRS #AzureZRS #AzureReplication #AzureForBeginners #MicrosoftAzure #AZ900 #AZ104 #LearnAzure #AzurePortal #AzureTutorial #LearnWithMithran #CloudComputing #AzureBeginner #GreenstTechnologies #AzureTraining #AzureCertification #AzureFreeTier #AzureObjectStorage

---

## Tags

azure storage account, blob storage azure, azure sas token, shared access signature azure, azure access tiers, azure lifecycle management, azure static website hosting, blob versioning azure, azure data protection, block blob azure, azure hot cool archive, azure lrs zrs grs, azure replication, az-900, az-104, azure for beginners, azure object storage, azure containers blobs, azure free tier storage, learnwithmithran, greens technologies, azure training, azure portal, microsoft azure, azure beginner, azure cloud storage, azure blob url, azure blob upload, azure storage tutorial
