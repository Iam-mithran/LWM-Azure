# Day 13 — YouTube Metadata

---

## Video Title

Azure DNS Explained: Public Zones, Private Zones & DNS Private Resolver (Hands-On) | Day 13 | LearnWithMithran

---

## Thumbnail

**Main text (large, bold):** `Name Everything`
**Sub text:** `Day 13 — Azure DNS Public & Private Zones`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- A globe icon (public zone) on one side and a VNet box with a small house icon (private zone) on the other, connected by a "DNS" label in the middle
- A small padlock icon labeled "DNSSEC" tucked in a corner
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** Host your domain in Azure, give your VNet a self-updating internal phonebook, and understand the managed hybrid DNS bridge — all hands-on.

---

## Description

*Welcome back to Learn With Mithran! Every day of this course you've typed a hostname — the portal, a storage account, a VM. Every single one of those had to be translated into an IP address before anything could actually talk. Today we open up that translation layer ourselves. You'll host a domain's records in Azure, give your own VNet an internal phonebook that updates itself automatically, and understand the modern, fully-managed way Azure bridges DNS between the cloud and on-premises networks.*

In this hands-on session, you'll create an Azure DNS Public Zone and add A, CNAME, MX, and TXT records — then query Azure's own name servers directly to prove resolution works, no real domain or registrar required. We'll cover Alias records, Azure's enhancement that keeps a record locked to a resource instead of a static IP that can go stale, and DNSSEC, which lets you cryptographically sign a zone so resolvers can trust your answers haven't been tampered with. Then we flip to the internal side: you'll build a fresh VNet and VM, create an Azure DNS Private Zone, link it with autoregistration enabled, and prove three resolution paths side by side from inside the VM — an auto-created record, a manual record, and normal public DNS — all served by the same built-in resolver with zero configuration. We close with a portal tour of Azure DNS Private Resolver, GA since June 2025, the fully managed replacement for hand-built DNS forwarder VMs in hybrid networking scenarios. Every resource is built from scratch and cleaned up at the end. 🚀

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

🔹 What DNS actually does — translating hostnames to IP addresses so nothing ever hard-codes an address that can change
🔹 DNS record types explained — A, AAAA, CNAME, MX, TXT, NS, SOA — and what each one is really for
🔹 TTL (Time to Live) — the cache vs. propagation-speed trade-off, and when to lower it before a planned change
🔹 Azure DNS Public Zones explained — four Azure-assigned name servers, and how domain delegation actually works
🔹 Demo — create a Public DNS Zone and add A, CNAME, MX, and TXT records
🔹 Demo — query Azure's name servers directly with nslookup to prove resolution, no registrar delegation required
🔹 Alias records explained — keeping a record locked to an Azure resource (Public IP, Traffic Manager, Front Door, CDN) instead of a static value that can go stale
🔹 DNSSEC explained — signing a zone so resolvers can cryptographically verify your DNS answers are authentic
🔹 Azure DNS Private Zones explained — internal-only DNS that resolves inside a VNet, never touching the public internet
🔹 VNet links and autoregistration — every VM in a linked VNet gets a hostname automatically, zero manual record-keeping
🔹 Demo — build a fresh VNet and VM, create a Private Zone, link it, and watch the VM's A record appear automatically
🔹 Demo — add a manual record and prove autoregistered, manual, and public DNS all resolve correctly from inside the VM
🔹 Azure's built-in DNS resolver (168.63.129.16) — what every VM uses by default and exactly what it can resolve
🔹 Custom DNS on a VNet — when and why you'd point at your own DNS servers for hybrid scenarios
🔹 Azure DNS Private Resolver explained — the fully managed hybrid DNS bridge, GA since June 2025
🔹 Inbound vs outbound endpoints and DNS forwarding rulesets — how the Private Resolver is actually built
🔹 Portal tour — walking the Private Resolver creation blade without leaving it running
🔹 Where this connects to VPN Gateway and ExpressRoute, coming in a future dedicated session
🔹 AZ-104 and AZ-305 exam framing for Azure DNS
🔹 Cleanup — delete every resource so nothing keeps billing

📌 *Who Is This Video For:*

💻 Beginners who've built VNets and VMs and are ready to give them proper names instead of raw IPs
🧑‍🎓 Students preparing for AZ-104 (Administrator) or AZ-305 (Solutions Architect) certification
☁️ Anyone who needs to host a real domain's DNS in Azure and understand delegation
🔥 Learners who want to *see* autoregistration happen in real time, not just hear about it
🖥️ IT professionals planning a hybrid DNS bridge between Azure and an on-premises network
🚀 Anyone designing production-grade architectures that need Alias records, DNSSEC, or Private Resolver

🔍 *Chapters:*
0:00 Intro — Every Hostname You've Typed So Far
3:00 Part 1 — DNS Fundamentals and Record Types
10:00 TTL Explained — Cache vs. Propagation Trade-Off
14:00 Part 2 — Azure DNS Public Zones and Delegation
20:00 Alias Records Explained
25:00 DNSSEC Explained
30:00 Demo — Create a Public DNS Zone and Add Records
40:00 Demo — Query Azure's Name Servers Directly
46:00 Part 3 — Azure DNS Private Zones Explained
52:00 VNet Links and Autoregistration
58:00 Demo — Build a Fresh VNet, VM, and Private Zone
68:00 Demo — Link With Autoregistration and Watch the Record Appear
75:00 Demo — Prove Three Resolution Paths From Inside the VM
82:00 Azure's Built-In Resolver and Custom DNS
88:00 Part 4 — Azure DNS Private Resolver Explained
96:00 Inbound/Outbound Endpoints and Forwarding Rulesets
102:00 Portal Tour — Private Resolver Creation Blade
108:00 Summary, Exam Tips, and Cleanup

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #AzureDNS #DNSPrivateResolver #AzurePrivateDNS #DNSSEC #AliasRecords #AzureNetworking #HybridDNS #AzureForBeginners #MicrosoftAzure #AZ104 #AZ305 #LearnAzure #AzurePortal #AzureTutorial #LearnWithMithran #CloudComputing #AzureBeginner #GreensTechnologies #AzureTraining #AzureCertification #AzureVNet #AzureDNSDemo #PrivateDNSZone #PublicDNSZone

---

## Tags

azure dns tutorial, azure dns public zone, azure dns private zone, azure dns private resolver, dnssec azure, azure alias records, azure vnet dns autoregistration, hybrid dns azure, az-104, az-305, azure for beginners, azure portal, azure tutorial, learnwithmithran, greens technologies, azure training, azure certification, microsoft azure, azure beginner, azure networking fundamentals, azure private dns zone demo, azure dns record types, azure dns delegation, azure dns resolver 168.63.129.16
