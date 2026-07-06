# Day 12 — YouTube Metadata

---

## Video Title

Azure Load Balancer, VM Scale Sets & Application Gateway Explained (Hands-On) | Day 12 | LearnWithMithran

---

## Thumbnail

**Main text (large, bold):** `Scale & Distribute Traffic`
**Sub text:** `Day 12 — Load Balancer, VMSS & App Gateway`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- Three icons in a row: a load balancer splitting arrows to two VMs, a VMSS icon with a "+" scaling symbol, and an Application Gateway routing two URL paths to separate backends
- A small "Layer 4 vs Layer 7" label underscoring the two routing icons
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** Stop relying on one VM — spread traffic, scale automatically, and route by URL path, all hands-on.

---

## Description

*Welcome back to Learn With Mithran! Every VM you've built in this course so far has been a single box. One public IP, one machine — and if it falls over, your app falls over with it. Today we fix that problem from three angles.*

In this hands-on session, you'll build an Azure Load Balancer from scratch — frontend IP, backend pool, health probe, load balancing rule, and NAT rules — and prove it's distributing traffic across two VMs in real time. Then you'll create a VM Scale Set so Azure can spin up and tear down instances automatically based on CPU demand, and watch a live scale-out event trigger right in the portal. Finally, you'll build an Application Gateway v2 with path-based routing rules so two completely different backends are served from a single public IP, and prove which backend answers each URL. Along the way we'll break down exactly when to pick Load Balancer versus Application Gateway — Layer 4 vs Layer 7 — including the exam framing AZ-104 and AZ-305 use for that decision. Every resource is built from scratch and deleted at the end. 🚀

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

🔹 Why a single VM is always a single point of failure — and the three Azure services that solve it
🔹 Azure Load Balancer explained — Layer 4 (TCP/UDP) traffic distribution using a 5-tuple hash
🔹 The four building blocks: frontend IP, backend pool, health probe, and load balancing rule
🔹 Inbound NAT rules — reaching a specific backend VM directly without going through the pool
🔹 Public vs Internal Load Balancer — when to use each, and how to tell them apart on the exam
🔹 Standard SKU only — why Basic Load Balancer no longer exists and what changed on September 30, 2025
🔹 Session persistence — None vs Client IP affinity, and when it matters for stateful apps
🔹 Demo — build a VNet with a NAT Gateway, deploy two plain VMs, and wire up a Standard Load Balancer from scratch
🔹 Demo — verify traffic is actually being distributed across both VMs in real time
🔹 VM Scale Sets (VMSS) explained — managing a fleet of identical VMs as one resource
🔹 Uniform vs Flexible orchestration modes — which to choose and why
🔹 Demo — create a VMSS with an integrated Load Balancer and install a web server fleet-wide using a custom script extension
🔹 Autoscale — building scale-out and scale-in rules driven by CPU metrics
🔹 Demo — watch a live scale event trigger and new instances join the backend pool automatically
🔹 Application Gateway explained — Layer 7 (HTTP/HTTPS) routing that understands URLs, paths, and headers
🔹 Listeners, routing rules, backend pools, and URL path maps — how they connect
🔹 Path-based routing — sending /api/* to one backend and /images/* to another behind a single IP
🔹 Multi-site hosting and SSL termination — why Application Gateway replaces a separate reverse proxy
🔹 Demo — build an Application Gateway v2 with two backend pools and a path-based routing rule, and prove both paths work
🔹 Load Balancer vs Application Gateway — the Layer 4 vs Layer 7 decision and how AZ-104/AZ-305 frames it
🔹 Cleanup — delete every resource so nothing keeps billing

📌 *Who Is This Video For:*

💻 Beginners who've deployed VMs and are ready to make them scalable and resilient
🧑‍🎓 Students preparing for AZ-104 (Administrator) or AZ-305 (Solutions Architect) certification
☁️ Anyone who needs to stop guessing VM capacity and let Azure scale automatically
🔥 Learners who want to *see* traffic distribution happen, not just hear about it in theory
🖥️ Developers whose apps need path-based routing or SSL termination at the gateway layer
🚀 Anyone designing production-grade, multi-VM architectures that have to survive a VM failure

🔍 *Chapters:*
0:00 Intro — One VM Is Not Enough
3:00 Part 1 — Azure Load Balancer Explained (Layer 4)
11:00 The Four Building Blocks: Frontend, Backend Pool, Health Probe, Rule
19:00 Public vs Internal Load Balancer + SKU changes
25:00 Session Persistence and Inbound NAT Rules
30:00 Demo — Build a VNet, NAT Gateway, and Two Backend VMs
45:00 Demo — Wire Up a Standard Load Balancer and Prove Traffic Distribution
62:00 Part 2 — VM Scale Sets Explained
70:00 Uniform vs Flexible Orchestration
75:00 Demo — Build a VMSS with an Integrated Load Balancer
88:00 Demo — Deploy a Web Server Fleet-Wide with a Custom Script Extension
96:00 Autoscale — Scale-Out and Scale-In Rules
102:00 Demo — Trigger a Live Scale Event and Watch Instances Join
112:00 Part 3 — Application Gateway Explained (Layer 7)
120:00 Listeners, Routing Rules, Path Maps, and SSL Termination
128:00 Demo — Build an Application Gateway v2 with Path-Based Routing
142:00 Load Balancer vs Application Gateway — The Decision Framework
148:00 Summary, Exam Tips, and Cleanup

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #AzureLoadBalancer #VMScaleSets #VMSS #ApplicationGateway #AzureScaling #AzureAutoscale #AzureNetworking #PathBasedRouting #Layer4vsLayer7 #AzureForBeginners #MicrosoftAzure #AZ104 #AZ305 #LearnAzure #AzurePortal #AzureTutorial #LearnWithMithran #CloudComputing #AzureBeginner #GreensTechnologies #AzureTraining #AzureCertification #AzureVM #AzureLoadBalancerDemo #StandardLoadBalancer

---

## Tags

azure load balancer tutorial, vm scale sets azure, azure vmss autoscale, application gateway azure, path based routing azure, azure load balancer vs application gateway, layer 4 vs layer 7 azure, azure autoscale, standard load balancer azure, azure application gateway v2, az-104, az-305, azure for beginners, azure portal, azure tutorial, learnwithmithran, greens technologies, azure training, azure certification, microsoft azure, azure beginner, azure scaling, azure traffic distribution, azure vm scale sets, azure load balancer demo
