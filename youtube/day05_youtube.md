# Day 5 — YouTube Metadata

---

## Video Title

Availability, Disks, Snapshots, Custom Images & Azure Backup | Day 5 | LearnWithMithran

---

## Thumbnail

**Main text (large, bold):** `Azure Backup`
**Sub text:** `Day 5 — VM Management & Protection`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- Shield icon (backup/protection theme) on the right side
- Icons for: disk, snapshot, custom image stacked on the left
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** Everything you need to manage, protect, and recover a VM on Azure.

---

## Description

*Welcome back to Learn With Mithran! Today we go deep into VM management — the stuff that actually matters in production. By the end of this session you'll know how Azure keeps VMs alive through hardware failures, how to manage and initialise disks on both Windows and Linux, how to build and deploy a custom VM image, and how to protect everything with Azure Backup.*

In this session we cover Availability Sets and Zones (99.95% vs 99.99% SLA), VM Scale Sets, all four Azure disk types, attaching and initialising data disks on Windows and Linux, VM snapshots, building a custom Ubuntu + Nginx image and deploying a clone from it, and Azure Backup with a full breakdown of daily, weekly, and monthly tiered retention and all four restore options. 🚀

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

🔹 Availability Sets — fault domains, update domains, and 99.95% SLA explained from scratch
🔹 Fault Domain vs Update Domain — rack-level hardware failure vs planned maintenance reboots
🔹 Availability Zones — physically separate data centers, 99.99% SLA, production standard
🔹 Availability Sets vs Availability Zones — when to use each and the cost difference
🔹 VM Scale Sets (VMSS) — auto-scale identical instances based on CPU demand (full demo in Day 8)
🔹 Azure Disk Types — Standard HDD, Standard SSD, Premium SSD, and Ultra Disk compared
🔹 OS Disk vs Data Disk vs Temporary Disk — what each is for and why temp disk data is wiped on deallocation
🔹 Adding a data disk to a VM in the Azure Portal — no VM restart needed
🔹 Initialising a data disk on Windows — Disk Management, GPT, New Simple Volume, NTFS
🔹 Initialising a data disk on Ubuntu Linux — fdisk, mkfs.ext4, mount, /etc/fstab for persistence
🔹 VM Snapshots — point-in-time disk captures, when to use them, and why they aren't a backup replacement
🔹 Custom Images — what they are and how they power VM Scale Sets in production
🔹 Generalising a VM with waagent -deprovision — what it removes and why the source VM can't be reused
🔹 Capturing a custom Ubuntu + Nginx image in the Azure Portal
🔹 Deploying a clone VM from a custom image — Nginx already running, zero setup required
🔹 Azure Backup — Recovery Services Vault, backup policies, and how backup-as-a-service works
🔹 Daily, Weekly, and Monthly tiered retention — how one backup job feeds all three tiers simultaneously
🔹 How tag overlap works — why a Sunday snapshot isn't deleted when its daily tag expires
🔹 Instant recovery snapshot — the local fast-restore cache vs vault-tier recovery
🔹 Recovery points — application-consistent vs crash-consistent
🔹 All four restore options — Replace Existing VM, Create New VM, Restore Disks Only, File-Level Recovery
🔹 Stopping backup before deleting a VM — why orphaned recovery points keep charging

📌 *Who Is This Video For:*

💻 Complete beginners who want to understand how Azure keeps VMs reliable and recoverable
🧑‍🎓 Students preparing for AZ-900 or AZ-104 certification
☁️ Developers and IT pros moving workloads to Azure who need to set up real backup strategies
🖥️ Anyone who has created a VM and now needs to manage, protect, and scale it
🚀 Anyone building toward a production-grade Azure environment

🔍 *Chapters:*
0:00 Intro — What Happens When Your VM's Server Dies?
2:00 Availability Sets — Fault Domains and Update Domains
12:00 Demo — Create a VM with an Availability Set
18:00 Availability Zones — Full Data Center Protection
26:00 Demo — Create a VM with an Availability Zone
30:00 VM Scale Sets — Auto-scale Concept (Full Demo in Day 8)
36:00 Disk Types in Azure — HDD vs SSD vs Premium vs Ultra
44:00 OS Disk, Data Disk, and Temporary Disk Explained
48:00 Demo — Add a Data Disk in the Azure Portal
52:00 Demo — Initialise Data Disk on Windows (Disk Management)
62:00 Demo — Initialise Data Disk on Ubuntu Linux (fdisk + mount + fstab)
74:00 VM Snapshots — What They Are and When to Use Them
78:00 Demo — Create a VM Snapshot
82:00 Custom Images — Build Once, Deploy Anywhere
86:00 Demo — Install Nginx, Deploy Custom HTML Page
92:00 Demo — Generalise VM with waagent and Capture the Image
100:00 Demo — Deploy a Clone VM from Custom Image (Nginx Pre-installed)
106:00 Azure Backup — Recovery Services Vault Deep Dive
112:00 Daily, Weekly, and Monthly Tiered Retention Explained
122:00 Demo — Enable Backup for Your VM and Trigger On-Demand Backup
130:00 Demo — Browse Recovery Points and Explore All 4 Restore Options
136:00 Cleanup — Stop Backup Before Deleting
139:00 Summary & What's Next (Day 6 — Azure App Service)

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #AzureVM #AzureBackup #AvailabilitySets #AvailabilityZones #VMScaleSets #AzureDisk #CustomImage #VMSnapshot #RecoveryServicesVault #AzureBackupPolicy #TieredRetention #AzureForBeginners #MicrosoftAzure #AZ900 #AZ104 #LearnAzure #AzurePortal #AzureTutorial #UbuntuAzure #WindowsServerAzure #Nginx #fdisk #AzureCompute #LearnWithMithran #CloudComputing #AzureBeginner #GreenstTechnologies #AzureTraining #AzureCertification #VMManagement #AzureDisks #DataDisk #AzureRestore #FileRecovery

---

## Tags

azure vm management tutorial, azure backup tutorial, availability sets azure, availability zones azure, vm scale sets azure, azure disk types explained, standard hdd ssd premium ultra disk, add data disk azure vm, initialize data disk windows azure, fdisk linux azure, azure vm snapshot, custom image azure vm, waagent deprovision azure, capture azure vm image, deploy vm from custom image, nginx custom image azure, recovery services vault azure, azure backup policy, daily weekly monthly retention azure backup, tiered retention azure, azure backup restore options, file level recovery azure, replace existing vm azure backup, azure backup recovery point, application consistent backup azure, az-900 azure backup, az-104 azure backup, azure vm availability 99.99, azure vm availability 99.95, fault domain update domain azure, azure free tier backup, learnwithmithran, greens technologies, azure training perumbakkam, azure compute beginner, azure portal vm management
