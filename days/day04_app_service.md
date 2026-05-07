# Day 4 — Azure App Service — Deploying Web Apps Without Managing Servers

> Phase 1 — Compute

## Goal

Deploy a web application to App Service (Platform-as-a-Service) and understand when to choose it over VMs.

## Key Topics

- What is Azure App Service? A fully managed platform for hosting web apps, REST APIs, and mobile backends — no server management required
- App Service Plans define the compute resources your app runs on:
  - Free (F1): Shared infrastructure, no custom domain, no SSL
  - Basic (B1–B3): Dedicated compute, custom domains, manual scale
  - Standard (S1–S3): Auto-scale, deployment slots, daily backups
  - Premium / Isolated: Higher performance, VNet integration
- Supported runtimes: Node.js, Python, Java, .NET, PHP, Ruby
- Deployment methods: ZIP deploy, GitHub Actions, Azure DevOps Pipelines, FTP
- Custom domains and SSL certificates
- Application settings and environment variables (injected at runtime, not hardcoded)
- Deployment slots: separate staging environment on the same App Service Plan — swap to production with zero downtime (Standard+ only)
- Scaling: Manual scale (Basic+), Auto-scale rules based on CPU/memory/schedule (Standard+)

## Hands-On Demo

**Account Requirements:** Creating an App Service on the Free (F1) plan is free tier. Auto-scale and deployment slots require a Standard plan (paid) — instructor will demonstrate those steps.

- ✅ Create an App Service Plan (Free F1 tier)
- ✅ Deploy a simple Node.js or Python app via ZIP deploy
- ✅ Set environment variables in Application Settings
- 💳 Configure Auto-scale rules (Standard plan required — instructor demo; students can try manual scale on Basic plan)
- 💳 Create a staging deployment slot and swap to production (Standard plan — instructor demo)

## Summary

App Service removes the need to manage servers. You bring your code; Azure handles patching, load balancing, and availability. Start on Free for learning, move to Standard when you need auto-scale and deployment slots for real projects.
