# Day 24 — Azure Container Registry (ACR) & Docker on Azure

> Phase 7 — Containers + AKS

## Goal

Store and manage Docker container images in Azure's private registry and run a container without a Kubernetes cluster.

## Key Topics

- Docker recap (brief): a Docker image is a packaged snapshot of your app and all its dependencies; a container is a running instance of that image; a Dockerfile describes how to build the image
- Azure Container Registry (ACR): a private registry hosted in Azure where you store, manage, and secure your Docker images — only your authorized services and pipelines can pull from it
- ACR SKUs:
  - Basic (~$5/month): development and testing — 10GB storage included
  - Standard (~$20/month): production workloads — 100GB storage, geo-replication not available
  - Premium (~$50/month): geo-replication, content trust, private link, dedicated throughput
- Pushing images to ACR: authenticate with `az acr login`, tag your image with the ACR login server, then `docker push`
- ACR Tasks: build Docker images directly in the cloud — no local Docker installation needed; trigger builds on code push to GitHub
- Image scanning with Microsoft Defender for Containers: automatically scan images for known CVEs when pushed to ACR (requires Defender plan)
- Integrating ACR with Azure DevOps Pipelines: add a Docker build-and-push step in your CI pipeline — images go straight to ACR on every merge
- Azure Container Instances (ACI): run a single container or a small group of containers instantly — no cluster, no orchestration, pay per second
  - Best for: batch jobs, event-driven tasks, quick tests, dev/test workloads
  - Not for: long-running production apps that need load balancing and auto-scaling (use AKS for that)

## Hands-On Demo

**Account Requirements:** ACR Basic SKU is ~$5/month (paid). ACI is pay-per-use (very low cost for short tests). Instructor will demonstrate all steps — students can follow along if using a free trial or paid account.

- 💳 Create an Azure Container Registry (Basic SKU — ~$5/month)
- 💳 Build a simple Docker image locally (or using ACR Tasks) and push it to ACR
- 💳 Create an Azure Container Instance from the ACR image and get its public IP
- 💳 Set up an ACR Task to automatically rebuild the image when code is pushed to GitHub

## Summary

ACR is your secure, private home for Docker images in Azure. ACI is the fastest way to run a container without any infrastructure overhead — great for testing and short-lived workloads. For production-grade container orchestration, that is what Day 25 and 26 are for.
