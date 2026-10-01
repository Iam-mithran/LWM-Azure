# Day 20 — YouTube Metadata

---

## Video Title

Azure Capstone Project: Build a Complete Three-Tier App From Scratch (Hands-On) | Day 20

---

## Thumbnail

**Main text (large, bold):** `19 Days. One System.`
**Sub text:** `Day 20 — Azure Capstone`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- Three stacked boxes, top to bottom: App Service (blue), two VMs behind a load balancer (green), Azure SQL (purple)
- A padlock over the SQL box with "NO PUBLIC ACCESS" stamped on it
- A Key Vault icon off to the side with a single key, and a browser bar reading https://www.learnwithmithran.com with a padlock
- Small corner badge: "Whole build ≈ 35 cents"
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** Every service from the course, built from an empty subscription into one working, private, monitored three-tier application.

---

## Description

*Welcome back to Learn With Mithran! For nineteen days we've learned Azure one service at a time, and deleted every one of them at the end of the video. Today we start from an empty subscription and build them all again, from scratch, as one working system.*

One brief, one architecture, one continuous build. By the end, a browser opens a web page showing data from a SQL database that nothing on the internet can reach. The page is served by VMs with no public IP addresses, and the database password lives in exactly one place. Phase One is pure design: six real decisions, each argued both ways with a price attached. Then we build the network and data tier, the app and web tiers, move the password into Key Vault, and finish with a real domain: learnwithmithran.com, delegated from GoDaddy to Azure DNS, with a free auto-renewing HTTPS certificate. Monitoring is your homework, and the notes include two add-ons.

💰 Everything up to the custom domain, you can build yourself; that part needs a domain you own. A four-hour build, deleted at the end, costs about 35 cents. Four meters never sleep, though, so the budget alert goes in before anything billable exists. 🏗️

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

🔹 Turning a plain-English brief into an Azure architecture
🔹 Six design decisions argued both ways, each with a price
🔹 Address planning: four subnets in 10.20.0.0/16, and why
🔹 NSGs and ASGs, plus the default rule that undoes tier isolation
🔹 Azure SQL built from scratch, then public access switched off
🔹 Private endpoint and private DNS, with three proofs it's private
🔹 Why a VM with no public IP has no internet, and the NAT gateway fix
🔹 Two VMs across availability zones, configured by cloud-init
🔹 Azure Bastion Developer SKU: free browser SSH
🔹 Internal load balancer, health probes and a deliberate failure
🔹 App Service VNet integration: watch it fail, then work
🔹 Key Vault references and managed identity, no stored password
🔹 Azure DNS public zone and GoDaddy name-server delegation
🔹 Custom domain with CNAME and TXT, plus a free managed certificate
🔹 Homework: one workspace, KQL across every tier, live alerts

📌 *Who Is This Video For:*

💻 Beginners ready to connect everything they've learned
🧑‍🎓 AZ-104 and AZ-305 candidates
🏗️ Anyone who wants a real architecture project for their portfolio
🚀 Anyone preparing for cloud architect interviews

🔍 *Chapters:*
0:00 Intro: 19 Days Become One System
4:00 What Today Costs
8:00 The Architecture at a Glance
11:00 Part 1: The Requirement
17:00 Part 2: Why Three Tiers
23:00 Part 3: Six Decisions, Argued Both Ways
38:00 Part 4: Address Planning and Naming
45:00 Part 5: Budget, Resource Group and VNet
53:00 Part 6: NSGs and ASGs Between Tiers
1:04:00 Part 7: SQL From Scratch, Then Private
1:16:00 Part 8: Three Proofs It's Private
1:19:00 Part 9: App Tier VMs and the NAT Gateway
1:33:00 Part 10: Bastion Developer, Free
1:43:00 Part 11: The Internal Load Balancer
1:52:00 Part 12: Prove It, Then Break It
1:58:00 Part 13: Web Tier and VNet Integration
2:13:00 Part 14: Watch It Fail, Then Work
2:21:00 Part 15: Locking the Front Door
2:25:00 Part 16: Connection String to Key Vault
2:36:00 Part 17: Azure DNS and GoDaddy Delegation
2:46:00 Custom Domain and Free HTTPS Certificate
2:58:00 Your Homework: Monitor Everything
3:01:00 Summary and Key Takeaways

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #AzureCapstone #AzureArchitecture #ThreeTierArchitecture #AzureSQL #PrivateEndpoint #AzureKeyVault #AzureDNS #AZ104 #AZ305 #AzureForBeginners #MicrosoftAzure #LearnWithMithran #GreensTechnologies

---

## Tags

azure capstone project, azure project for beginners, three tier architecture azure, azure solution architect, azure sql private endpoint, app service vnet integration, azure internal load balancer, azure bastion developer, nat gateway azure, azure key vault reference, managed identity, azure dns custom domain, app service managed certificate, az-104, az-305, azure for beginners, azure tutorial, microsoft azure, learnwithmithran
