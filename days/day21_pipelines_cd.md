# Day 20 — Azure Pipelines — CD (Continuous Delivery/Deployment)

> Phase 5 — Azure DevOps

## Goal

Extend the CI pipeline with automated deployment to Azure — with approval gates before production.

## Key Topics

- Continuous Delivery vs Continuous Deployment:
  - Continuous Delivery: every successful build is ready to deploy, but a human approves the production push
  - Continuous Deployment: every successful build deploys to production automatically — no manual gate
- Environments in Azure Pipelines: named targets (e.g., dev, staging, production) that track deployments and support approval gates
- Deployment strategies:
  - **Rolling**: replace old instances gradually with new ones
  - **Blue-Green**: run old (blue) and new (green) in parallel; switch traffic instantly; rollback by switching back
  - **Canary**: route a small percentage of traffic to the new version; monitor; gradually increase
- Deployment jobs: a special job type in YAML that records deployment history and supports rollback
- Approvals and gates: require a specific person or group to manually approve before a stage proceeds — essential before production
- Deploying to Azure App Service from a pipeline: use the `AzureWebApp@1` task with a Service Connection
- Deploying to Azure VM: use a Deployment Group — register VMs as targets and push updates to them from the pipeline
- Rollback strategies: re-run a previous deployment, use deployment slots (swap back), or use Terraform to restore known state
- Multi-stage YAML pipeline: combine CI and CD in a single `azure-pipelines.yml` file with multiple stages

## Hands-On Demo

**Account Requirements:** Basic deployment to a Free-tier App Service is free. Deployment slots require an App Service Standard plan (paid) — instructor will demonstrate those steps.

- ✅ Extend the Day 19 YAML pipeline with a Deploy stage
- ✅ Deploy to Azure App Service using the AzureWebApp@1 task (Free or Basic plan)
- ✅ Add a manual approval gate on the Production environment
- 💳 Deploy to a staging slot and swap to production (App Service Standard plan required — instructor demo; students on Free/Basic plan can deploy directly to the production slot instead)

## Summary

A CD pipeline automates the path from code to running application. Manual approval gates are your safety net before production. Deployment slots let you test in a production-like environment and cut over instantly with zero downtime — worth the cost of a Standard plan for real projects.
