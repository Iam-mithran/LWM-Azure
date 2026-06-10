# Day 8 — YouTube Metadata

---

## Video Title

Azure Files, Queue Storage, Table Storage & Storage Explorer Explained | Day 8 | LearnWithMithran

---

## Thumbnail

**Main text (large, bold):** `Files, Queues & Tables`
**Sub text:** `Day 8 — Complete the Storage Picture`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- Azure Storage Account icon (top left)
- Four service icons (Blob, Files, Queue, Table) — Files, Queue, and Table highlighted, Blob greyed out to signal "today = the rest of storage"
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** One storage account. A file share you can mount, messages that decouple your apps, and a NoSQL table for instant lookups.

---

## Description

*Welcome back to Learn With Mithran! Last session we went deep on Blob Storage — today we complete the Azure Storage picture with Azure Files, Queue Storage, and Table Storage, plus the tools that tie it all together.*

In this session we cover Azure Files — a fully managed cloud file share you can mount like a network drive over SMB or NFS — including file share tiers, snapshots for point-in-time recovery, and soft delete for accidental deletion protection. We then look at Azure File Sync, which extends an on-premises Windows file server into the cloud with cloud tiering. Next up is Queue Storage — how producers and consumers talk to each other through messages, what visibility timeout means, and a hands-on Python SDK demo sending and peeking messages. Then Table Storage — Azure's schemaless NoSQL key-value store, and how PartitionKey and RowKey design your data for fast queries. We close with three essentials for AZ-104 and real production environments: Storage Firewall for network restrictions, AzCopy for bulk data migration, and Immutable Storage (WORM policies) for compliance. Finally, we tie it all together with Azure Storage Explorer — the free desktop tool that manages every storage service in one place. Coming up next, we move into Azure Virtual Networking. 🚀

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

🔹 Azure Files — fully managed cloud file shares accessible via SMB, NFS, and HTTPS
🔹 Azure Files vs Blob Storage — directory structure, mounting as a drive, and when to use each
🔹 File Share Tiers — Transaction Optimized, Hot, Cool, and Premium — and why the tier choice is permanent
🔹 Demo — Create an Azure File Share and upload files into nested folders
🔹 Demo — View mount instructions for Windows (SMB) and Linux (NFS), and the port 445 firewall gotcha
🔹 File Share Snapshots — storage-efficient, point-in-time recovery copies
🔹 Demo — Take a snapshot, simulate a destructive change, and restore a file
🔹 Soft Delete for File Shares — recover an entire deleted share within a grace period
🔹 Demo — Enable soft delete, delete a share, and undelete it
🔹 What is Queue Storage — decoupling application components with messages
🔹 Visibility timeout explained — how Queue Storage prevents double-processing
🔹 Demo — Create a queue and add, peek, and dequeue messages in the portal
🔹 Demo — Python SDK: send and peek queue messages programmatically
🔹 What is Table Storage — Azure's schemaless NoSQL key-value store
🔹 PartitionKey and RowKey — the composite primary key and how it drives query performance
🔹 Demo — Create a table, add entities with different properties, and query by PartitionKey
🔹 Azure File Sync — sync on-premises Windows file servers with Azure Files
🔹 Cloud Tiering — keep hot files local, automatically tier cold files to Azure
🔹 Storage Firewall — restrict storage account access to specific networks and IP ranges
🔹 AzCopy — the command-line tool for bulk data migration (an AZ-104 exam staple)
🔹 Immutable Storage (WORM) — time-based retention and legal hold for compliance
🔹 Demo — Enable a time-based immutability policy and confirm deletion is blocked
🔹 Azure Storage Explorer — manage Blob, Files, Queue, and Table from one desktop app
🔹 Demo — Connect Storage Explorer and browse all four storage services in one place
🔹 What's coming next — Azure Virtual Networking, NSGs, DNS, and Azure Bastion

📌 *Who Is This Video For:*

💻 Complete beginners building on the Storage Account fundamentals from Day 7
🧑‍🎓 Students preparing for AZ-900 or AZ-104 certification
☁️ Developers who need file shares, message queues, or fast NoSQL lookups in their apps
🖥️ IT professionals planning to migrate on-premises file servers to the cloud
🔄 Architects designing decoupled, event-driven application components
🚀 Anyone building toward a production-ready Azure application

🔍 *Chapters:*
0:00 Intro — Completing the Azure Storage Picture
2:00 What Is Azure Files — SMB, NFS & HTTPS Explained
8:00 Azure Files vs Blob Storage — Key Differences
12:00 File Share Tiers — Transaction Optimized, Hot, Cool & Premium
18:00 Demo — Create an Azure File Share and Upload Files
26:00 Demo — Mount the File Share on Windows and Linux
34:00 File Share Snapshots — Point-in-Time Recovery
40:00 Demo — Create, Browse, and Restore a Snapshot
46:00 Soft Delete for File Shares
50:00 Demo — Enable Soft Delete and Recover a Deleted Share
56:00 What Is Queue Storage — Decoupling Application Components
64:00 Visibility Timeout Explained
70:00 Demo — Create a Queue and Send/Peek/Dequeue Messages
78:00 Demo — Python SDK: Send and Peek Queue Messages
88:00 What Is Table Storage — Schemaless NoSQL Explained
94:00 PartitionKey and RowKey — Designing for Fast Queries
100:00 Demo — Create a Table, Add Entities, and Query by PartitionKey
108:00 Azure File Sync and Cloud Tiering
116:00 Storage Firewall — Restricting Network Access
120:00 AzCopy — Bulk Data Migration for AZ-104
126:00 Immutable Storage (WORM) — Time-Based Retention & Legal Hold
132:00 Demo — Enable an Immutability Policy
138:00 Azure Storage Explorer — One Tool for All Storage Services
144:00 Demo — Connect and Browse Storage Explorer
150:00 Cleanup — Delete the Resource Group
152:00 Summary and What's Next (Virtual Networking)

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #AzureStorage #AzureFiles #AzureFileShare #AzureFileSync #CloudTiering #QueueStorage #AzureQueueStorage #TableStorage #AzureTableStorage #PartitionKeyRowKey #AzCopy #ImmutableStorage #WORM #StorageFirewall #AzureStorageExplorer #AzureSDK #PythonAzure #AzureForBeginners #MicrosoftAzure #AZ900 #AZ104 #LearnAzure #AzurePortal #AzureTutorial #LearnWithMithran #CloudComputing #AzureBeginner #GreensTechnologies #AzureTraining #AzureCertification #AzureFreeTier #AzureNoSQL

---

## Tags

azure files, azure file share, azure file sync, cloud tiering, queue storage azure, azure queue storage, table storage azure, azure table storage, partitionkey rowkey, azure nosql, azcopy, azure storage explorer, immutable storage azure, worm policy azure, storage firewall azure, azure smb nfs, az-900, az-104, azure for beginners, azure sdk python, azure storage tutorial, azure storage account, learnwithmithran, greens technologies, azure training, azure portal, microsoft azure, azure beginner, azure cloud storage, azure free tier storage
