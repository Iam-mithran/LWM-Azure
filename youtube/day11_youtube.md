# Day 11 — YouTube Metadata

---

## Video Title

Azure VNet Peering, Private Endpoints & Bastion Explained (Hands-On) | Day 11 | LearnWithMithran

---

## Thumbnail

**Main text (large, bold):** `Connect & Secure Your VNet`
**Sub text:** `Day 11 — Peering, Private Endpoints & Bastion`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- Two VNet boxes joined by a peering line, with a small lock icon over a "Private Endpoint" connection
- A browser window icon labeled "Bastion" replacing a public IP with a red X over it
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** Connect VNets, reach Azure services privately, and access VMs with zero public IPs — all hands-on.

---

## Description

*Welcome back to Learn With Mithran! In Day 10 you built your first VNet with public and private subnets, locked down with NSGs. But real Azure environments rarely live in just one VNet, and they almost never talk to services like Storage over the public internet if they can help it. Today we connect the dots — and lock things down further.*

In this hands-on session, you'll connect two separate VNets privately with VNet Peering, then secure access to an Azure Storage account two different ways — first with a Service Endpoint, then with a Private Endpoint that gives the storage account its own private IP inside your VNet. We prove both restrictions are real by hitting the same SAS-authenticated URL from an allowed VM and a blocked VM and watching the results differ. From there, we deploy Azure Bastion for browser-based VM access with no public IP and no exposed SSH or RDP port, override Azure's default routing with a custom Route Table, and finish by grouping VMs into Application Security Groups so NSG rules target a role instead of a fragile IP address. Every demo today is built completely from scratch. 🚀

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

🔹 VNet Peering — connecting two separate VNets privately, no internet or gateway required
🔹 Demo — building a second VNet and peering it with your Day 10/11 VNet in one operation
🔹 Demo — deploying two test VMs to carry through the rest of today's demos
🔹 Service Endpoints vs Private Endpoints — the two ways to securely reach Azure services from inside a VNet
🔹 Demo — locking a storage account down to a single subnet with a Service Endpoint
🔹 Demo — proving the restriction is real with a SAS-authenticated `curl` request from an allowed VM and a blocked VM
🔹 Demo — creating a Private Endpoint so the storage account gets a private IP inside the VNet
🔹 Demo — verifying DNS resolution changes from inside a VM once the Private Endpoint exists
🔹 Azure Bastion explained — browser-based VM access with no public IP and no open SSH/RDP port
🔹 Demo — deploying Bastion and connecting to a VM straight from the browser
🔹 Route Tables and User-Defined Routes (UDR) — overriding Azure's invisible default routing
🔹 Demo — building a custom route table to force traffic through a virtual appliance
🔹 Application Security Groups (ASGs) — grouping VMs by role instead of by IP address
🔹 Demo — tagging VMs into ASGs and rewriting an NSG rule to target a group
🔹 What's coming next — Azure DNS, Public and Private DNS Zones

📌 *Who Is This Video For:*

💻 Beginners who completed Day 10 and are ready to connect multiple VNets together
🧑‍🎓 Students preparing for AZ-900 or AZ-104 certification
☁️ Anyone who needs Azure services reachable privately instead of over the public internet
🔥 Learners who want to actually *prove* a network restriction works, not just hear about it
🖥️ IT professionals tired of exposing SSH/RDP ports and looking for a Bastion-based alternative
🚀 Anyone designing multi-tier, multi-VNet architectures with custom routing and role-based security

🔍 *Chapters:*
0:00 Intro — From One VNet to a Connected, Secured Network
3:00 Part 1 — VNet Peering Explained
9:00 Demo — Build a Second VNet and Peer It
17:00 Demo — Deploy Two Test VMs
24:00 Part 2 — Service Endpoints vs Private Endpoints
32:00 Demo — Lock a Storage Account With a Service Endpoint
42:00 Demo — Prove the Restriction From Both VMs
52:00 Demo — Create a Private Endpoint and Verify DNS
62:00 Part 3 — Azure Bastion Explained
68:00 Demo — Deploy Bastion and Connect via Browser
78:00 Part 4 — Route Tables and User-Defined Routes
86:00 Demo — Build a Custom Route Table
94:00 Part 5 — Application Security Groups Explained
100:00 Demo — Tag VMs Into ASGs and Rewrite an NSG Rule
108:00 Summary and What's Next (Azure DNS)

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #AzureVNet #VNetPeering #PrivateEndpoint #ServiceEndpoint #AzureBastion #AzureNetworking #RouteTable #ApplicationSecurityGroup #AzureForBeginners #MicrosoftAzure #AZ900 #AZ104 #LearnAzure #AzurePortal #AzureTutorial #LearnWithMithran #CloudComputing #AzureBeginner #GreensTechnologies #AzureTraining #AzureCertification #AzureFreeTier #AzureVM #PrivateVsPublicIP

---

## Tags

azure vnet peering, azure private endpoint, azure service endpoint, azure bastion tutorial, azure route table, user defined routes azure, application security group azure, az-900, az-104, azure for beginners, azure portal, azure tutorial, learnwithmithran, greens technologies, azure training, azure certification, microsoft azure, azure beginner, azure free tier, azure networking fundamentals, azure cloud networking, azure vm deployment, ssh rdp azure, azure storage account security
