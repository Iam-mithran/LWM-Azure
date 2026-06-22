# Day 10 — YouTube Metadata

---

## Video Title

Azure Virtual Network Explained: Subnets, NSGs & NAT Gateway (Hands-On) | Day 10 | LearnWithMithran

---

## Thumbnail

**Main text (large, bold):** `Build Your First VNet`
**Sub text:** `Day 10 — Subnets, NSGs & NAT Gateway`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- A VNet box splitting into two subnets (public/private), one with a lock icon
- A small "NAT" badge near an outbound arrow
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** Go from zero to a fully working, secured Azure network — built and proven with real VMs.

---

## Description

*Welcome back to Learn With Mithran! In Day 9 we learned the language every network speaks — IP addresses, binary, and CIDR. Today we put that language to work and build your first real Azure Virtual Network from scratch.*

In this hands-on session, you'll design a VNet's address space, carve it into public and private subnets, lock them down with Network Security Group rules, and then prove the whole design works by deploying real Windows and Linux VMs — watching first-hand why a VM with a public IP is reachable from the internet and a VM without one simply isn't, no matter what your firewall rules say. We close out with a NAT Gateway demo, giving your private-subnet VMs a stable, scalable outbound path to the internet without ever exposing them to inbound traffic. By the end, you'll have a fully working, secured two-tier network architecture built entirely with your own hands. 🚀

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

🔹 What a Virtual Network (VNet) is and why every Azure resource lives inside one
🔹 Choosing a VNet address space with CIDR — why `10.0.0.0/16` is the go-to default
🔹 The 5 addresses Azure reserves in every subnet (and why a `/24` only gives you 251 usable IPs)
🔹 Subnets explained — segmenting a VNet for security and organisation
🔹 Demo — creating a VNet with a `public-subnet` and `private-subnet`
🔹 Network Security Groups (NSGs) — priority, rules, and Azure's default allow/deny behaviour
🔹 Subnet-level vs NIC-level NSGs — where to attach rules and why
🔹 Demo — creating an NSG, adding SSH/HTTP/RDP rules, and attaching it to a subnet
🔹 Demo — deploying real Windows and Linux VMs into public and private subnets
🔹 Proving a public VM is directly reachable from the internet via SSH and RDP
🔹 Proving a private VM is completely unreachable from the internet — even with no NSG blocking it
🔹 The jump box / bastion host pattern — reaching a private VM through a public one
🔹 Why Azure's default outbound access isn't production-grade
🔹 NAT Gateway explained — stable, scalable, outbound-only internet access
🔹 Demo — deploying a NAT Gateway and verifying a private VM's outbound IP
🔹 What's coming next — VNet Peering, Private/Service Endpoints, and Azure Bastion

📌 *Who Is This Video For:*

💻 Complete beginners who've never built a network before, in Azure or anywhere else
🧑‍🎓 Students preparing for AZ-900 or AZ-104 certification
☁️ Anyone designing a secure, multi-tier architecture in Azure for the first time
🔥 Learners who want to actually *see* public vs private reachability proven, not just explained
🖥️ IT professionals who know VNets exist but have never built one hands-on
🚀 Anyone preparing to connect multiple VNets or use Azure Bastion down the line

🔍 *Chapters:*
0:00 Intro — From CIDR Theory to a Real Azure Network
2:00 Part 1 — What Is a Virtual Network?
8:00 Part 2 — Choosing Your VNet's Address Space
14:00 What Azure Reserves in Every Subnet
20:00 Part 3 — Subnets, Segmenting Your Network
26:00 Demo — Create a VNet With Two Subnets
34:00 Part 4 — Network Security Groups Explained
44:00 Demo — Create an NSG and Add Inbound Rules
54:00 Demo — Attach the NSG to the Public Subnet
60:00 Part 5 — Deploying Real Public & Private VMs
74:00 Demo — Confirm the Public VM Is Reachable
82:00 Demo — Confirm the Private VM Is NOT Reachable
90:00 Demo — Jump Box Pattern, Public VM to Private VM
100:00 Part 6 — NAT Gateway Explained
108:00 Demo — Deploy and Verify a NAT Gateway
116:00 Summary and What's Next (VNet Peering & Azure Bastion)

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #AzureVNet #AzureNetworking #NetworkSecurityGroup #NATGateway #AzureSubnets #AzureForBeginners #MicrosoftAzure #AZ900 #AZ104 #LearnAzure #AzurePortal #AzureTutorial #LearnWithMithran #CloudComputing #AzureBeginner #GreensTechnologies #AzureTraining #AzureCertification #AzureFreeTier #JumpBox #BastionHost #AzureVM #PrivateVsPublicIP

---

## Tags

azure virtual network, azure vnet tutorial, azure subnets explained, network security group azure, nsg rules, azure nat gateway, public vs private subnet, jump box azure, bastion host pattern, az-900, az-104, azure for beginners, azure portal, azure tutorial, learnwithmithran, greens technologies, azure training, azure certification, microsoft azure, azure beginner, azure free tier, azure networking fundamentals, azure cloud networking, azure vm deployment, ssh rdp azure
