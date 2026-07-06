# Day 14 — YouTube Metadata

---

## Video Title

Azure Traffic Manager, Front Door & WAF Explained (Hands-On) | Day 14 | LearnWithMithran

---

## Thumbnail

**Main text (large, bold):** `Go Global`
**Sub text:** `Day 14 — Traffic Manager, Front Door & WAF`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- A globe with several PoP dots lighting up, arrows converging from different continents into one edge node
- A shield icon labeled "WAF" blocking a red "SQLi" arrow
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** Route users worldwide, cache at the edge, and block attacks before they reach your app — all hands-on, using Azure's current, non-retiring services.

---

## Description

*Welcome back to Learn With Mithran! Everything you've built so far in this course lives in one Azure region. That's fine until your users are on the other side of the planet, or your one region has a bad day. Today we go global — and we deal with something genuinely current while we're at it: two of the classic services this topic used to be built on, Azure CDN and Azure Front Door (classic), stopped accepting new deployments back in August 2025. So everything we build today uses the modern replacement Microsoft actually wants you using.*

In this hands-on session, you'll build an Azure Traffic Manager profile across two regions, watch Performance routing pick the closest one, then stop an app outright and watch DNS fail over automatically with zero manual intervention. From there we build an Azure Front Door Standard profile in front of a static website and prove edge caching is real by watching the `X-Cache` response header flip from a cache miss to a cache hit. Finally, we lock it down: you'll create a WAF Policy built on the OWASP Core Rule Set, start it in Detection mode to see what would be blocked, then flip it to Prevention mode and fire a real SQL injection attempt at your own endpoint — and watch it get stopped cold at the edge, never reaching your origin at all. Every resource is built from scratch and deleted at the end. 🚀

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

🔹 Why a single region is always a single point of failure, even with Load Balancer and VMSS already in place
🔹 Azure Traffic Manager explained — pure DNS-based global load balancing that never touches your actual traffic
🔹 The six routing methods: Performance, Priority, Weighted, Geographic, Subnet, and MultiValue
🔹 Demo — build a Traffic Manager profile across two App Service regions
🔹 Demo — stop an app and watch DNS fail over automatically via health probes, no manual DNS edits
🔹 Azure Front Door explained — global anycast edge routing, caching, SSL offload, and WAF in one service
🔹 Why Azure CDN (classic) and Azure Front Door (classic) stopped accepting new deployments on August 15, 2025 — and what replaces them
🔹 Azure CDN from Edgio's shutdown — the legacy signal to migrate immediately if you ever see it
🔹 Front Door Standard vs Premium — what Premium adds (full managed rules, Bot Manager, Private Link)
🔹 Demo — build a Front Door Standard profile in front of a Blob Storage static website
🔹 Demo — prove edge caching is real by watching the X-Cache header flip from TCP_MISS to TCP_HIT
🔹 Web Application Firewall (WAF) explained — the OWASP Core Rule Set and Microsoft Threat Intelligence rules
🔹 Detection mode vs Prevention mode — why you always roll out WAF in Detection first
🔹 Custom rules and the Bot Manager rule set — Bad/Good/Unknown bot categories
🔹 Why WAF Policy is now the only way to configure WAF, on both Front Door and Application Gateway
🔹 Demo — create a WAF Policy, attach it to Front Door, and review Detection mode logs
🔹 Demo — flip to Prevention mode and fire a real SQL injection attempt — watch it get blocked at the edge
🔹 Traffic Manager vs Front Door vs Application Gateway — the full decision framework for AZ-104 and AZ-305
🔹 Cleanup — delete every resource so nothing keeps billing

📌 *Who Is This Video For:*

💻 Beginners who've built regional networking (Days 9–13) and are ready to think globally
🧑‍🎓 Students preparing for AZ-104 (Administrator) or AZ-305 (Solutions Architect) certification
☁️ Anyone still running Azure CDN or Front Door (classic) who needs to understand what's changing and why
🔥 Learners who want to *see* a DNS failover and a blocked SQL injection happen live, not just hear about them
🖥️ Developers who need edge caching and SSL offload in front of a web app or static site
🚀 Anyone designing globally resilient, security-hardened architectures for production traffic

🔍 *Chapters:*
0:00 Intro — One Region Is Not Enough
3:00 Part 1 — Azure Traffic Manager Explained
9:00 The Six Routing Methods
15:00 Demo — Build a Traffic Manager Profile Across Two Regions
24:00 Demo — Force a Failover and Watch DNS Respond
32:00 Part 2 — Azure Front Door Explained
40:00 Why CDN and Front Door Classic Are Retiring
46:00 Front Door Standard vs Premium
50:00 Demo — Build a Front Door Profile in Front of a Static Website
58:00 Demo — Prove Edge Caching With the X-Cache Header
64:00 Part 3 — Web Application Firewall Explained
72:00 Detection Mode vs Prevention Mode
78:00 Custom Rules and Bot Manager
84:00 Demo — Create a WAF Policy and Attach It to Front Door
92:00 Demo — Detection Mode Logs, Then Block a Real SQL Injection
100:00 Traffic Manager vs Front Door vs Application Gateway — Decision Framework
106:00 Summary, Exam Tips, and Cleanup

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #AzureTrafficManager #AzureFrontDoor #AzureWAF #WebApplicationFirewall #AzureCDN #AzureNetworking #OWASP #AzureForBeginners #MicrosoftAzure #AZ104 #AZ305 #LearnAzure #AzurePortal #AzureTutorial #LearnWithMithran #CloudComputing #AzureBeginner #GreensTechnologies #AzureTraining #AzureCertification #AzureGlobalLoadBalancing #FrontDoorDemo #AzureSecurity

---

## Tags

azure traffic manager tutorial, azure front door tutorial, azure waf demo, web application firewall azure, azure cdn retirement, front door classic retirement, owasp core rule set azure, azure bot manager, az-104, az-305, azure for beginners, azure portal, azure tutorial, learnwithmithran, greens technologies, azure training, azure certification, microsoft azure, azure beginner, azure global load balancing, azure edge caching, azure sql injection demo, traffic manager vs front door, azure waf policy
