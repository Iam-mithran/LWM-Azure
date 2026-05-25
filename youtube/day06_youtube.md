# Day 6 — YouTube Metadata

---

## Video Title

Azure App Service: Deploy Web Apps Without Managing Servers | Day 6 | LearnWithMithran

---

## Thumbnail

**Main text (large, bold):** `App Service`
**Sub text:** `Day 6 — Deploy Code, Not Servers`
**Suggested visual elements:**
- Azure blue background (#0078D4)
- Left side: VM icon with a red X (you manage everything)
- Right side: App Service icon with a green tick (Azure manages everything)
- Channel name: LearnWithMithran (bottom corner)

**Key message to convey at a glance:** Stop managing servers — just deploy your code.

---

## Description

*Welcome back to Learn With Mithran! In Day 4 you hosted a website by creating a VM, installing Nginx, opening ports, and managing everything yourself. Today you'll learn a smarter way — Azure App Service. You bring the code. Azure handles the OS, the web server, the runtime, the patches, and the scaling.*

In this session we cover PaaS vs IaaS, all five App Service Plan tiers, supported runtimes, deploying a real Node.js app via ZIP Deploy through the Kudu console, application settings and environment variables, custom domains and managed SSL certificates, zero-downtime production swaps with deployment slots, and auto-scale rules that add or remove instances automatically based on CPU load. 🚀

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

🔹 IaaS vs PaaS — what changes when you move from a VM to App Service
🔹 What Azure App Service manages for you — OS, web server, runtime, patches, load balancing
🔹 When to use App Service vs a VM — the honest trade-off
🔹 App Service Plans — Free F1, Basic B1–B3, Standard S1–S3, Premium P1v3, Isolated
🔹 Free F1 limitations — 60 CPU min/day, no custom domain, app sleeps after 20 min idle
🔹 Multiple apps on one plan — how apps share compute and why it saves cost
🔹 Supported runtimes — Node.js, Python, .NET, Java, PHP, Ruby — no installation needed
🔹 Deployment methods overview — ZIP Deploy, GitHub Actions, Azure DevOps, Local Git
🔹 Demo — Create an App Service Plan and Web App in the portal
🔹 Demo — Deploy a real Node.js app via ZIP Deploy through the Kudu console
🔹 Why the ZIP must contain files at root level — not inside a subfolder
🔹 Application Settings — inject environment variables at runtime without touching code
🔹 Demo — Add APP_NAME as an application setting and watch the page update instantly
🔹 Why secrets must never be hardcoded — git history exposure explained
🔹 Custom domains — how to bind your own domain and prove ownership via DNS
🔹 Managed SSL certificates — App Service issues and auto-renews free HTTPS for custom domains
🔹 Deployment slots — staging environment on the same plan with its own URL
🔹 Zero-downtime swap — Azure warms up staging before flipping it to production
🔹 Instant rollback — one swap click reverts production to the previous version
🔹 Demo — Create a staging slot, deploy v2, swap to production, roll back (💳 Paid Demo)
🔹 Manual scale — drag the instance slider on Basic plan and above
🔹 Auto-scale rules — CPU threshold triggers that add or remove instances automatically
🔹 Demo — Configure scale-out and scale-in rules on Standard plan (💳 Paid Demo)

📌 *Who Is This Video For:*

💻 Complete beginners who have used VMs and want to move to a managed platform
🧑‍🎓 Students preparing for AZ-900 or AZ-104 certification
☁️ Developers who want to deploy Node.js, Python, or .NET apps to Azure quickly
🖥️ IT professionals who want to stop managing web servers and focus on code
🚀 Anyone building toward a production-ready Azure web application

🔍 *Chapters:*
0:00 Intro — Why Managing Servers Yourself Doesn't Scale
2:00 IaaS vs PaaS — What Azure App Service Takes Off Your Plate
10:00 VM vs App Service — Side-by-Side Comparison
16:00 Demo — Create an App Service Plan and Web App (Free F1)
24:00 App Service Plan Tiers — Free, Basic, Standard, Premium, Isolated
34:00 Demo — Explore App Service Plan and Tier Options in the Portal
38:00 Supported Runtimes — Node.js, Python, .NET, Java, PHP
42:00 Deployment Methods — ZIP Deploy, GitHub Actions, Local Git
46:00 Demo — Build a Node.js App and Deploy via ZIP (Kudu Console)
58:00 Application Settings and Environment Variables — Why Not Hardcode
64:00 Demo — Add APP_NAME Setting and Watch the App Update Live
70:00 Custom Domains and Managed SSL Certificates
76:00 Deployment Slots — Zero-Downtime Production Swaps Explained
82:00 Demo — Staging Slot, Swap to Production, Instant Rollback (💳 Paid)
92:00 Manual Scale vs Auto-Scale — How Instances Work
96:00 Demo — Auto-Scale Rules: Scale Out and Scale In (💳 Paid)
104:00 Cleanup — Delete Resource Group
106:00 Summary and What's Next (Day 7 — Azure Virtual Networks)

👍 If this video helps you, like, subscribe, and turn on notifications for more hands-on content on Azure, DevOps, AWS, Linux, and Python.

#Azure #AzureAppService #AzureWebApp #ZIPDeploy #KuduAzure #DeploymentSlots #AzureAutoScale #AzurePaaS #AppServicePlan #AzureNodejs #AzureEnvironmentVariables #AzureManagedSSL #AzureCustomDomain #AzureScaling #IaaSvsPaaS #AzureForBeginners #MicrosoftAzure #AZ900 #AZ104 #LearnAzure #AzurePortal #AzureTutorial #LearnWithMithran #CloudComputing #AzureBeginner #GreenstTechnologies #AzureTraining #AzureCertification #AzureCompute #AzureFreeT1er

---

## Tags

azure app service, azure web app, zip deploy azure, kudu azure, azure deployment slots, azure auto scale, azure app service plan, azure paas, azure environment variables, azure free tier, az-900, az-104, node js azure, azure hosting, learnwithmithran, greens technologies, azure training, azure portal, paas vs iaas, azure staging slot, azure ssl, azure scale out, microsoft azure, azure web app tutorial, azure beginner, azure compute, azure custom domain, azure managed ssl, azure nodejs
