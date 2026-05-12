# Day 28 — Capstone Part 2: CI/CD Pipeline, Monitoring & Course Wrap-Up

> Phase 8 — Capstone Project

## Goal

Complete the capstone with a full CI/CD pipeline, production monitoring, and a look at what comes next.

## Key Topics

- Full Azure DevOps pipeline for the 3-tier capstone application:
  - **Stage 1 — CI**: build the API Docker image, run tests, push the image to ACR with a versioned tag
  - **Stage 2 — Deploy to Dev**: run `helm upgrade` to deploy the new image to the AKS dev namespace; smoke test the API endpoint
  - **Stage 3 — Manual Approval Gate**: a designated reviewer approves the production deployment in the Azure DevOps environment
  - **Stage 4 — Deploy to Production**: run `helm upgrade` on the AKS production namespace — Kubernetes performs a zero-downtime rolling update
- Application Insights integration:
  - Add the Application Insights SDK to the API backend — instrument requests, dependencies, and exceptions automatically
  - Enable distributed tracing: follow a single user request from the App Service frontend through the API backend to the SQL database
- Log Analytics and KQL for the capstone:
  - Query AKS container logs (`ContainerLogV2` table)
  - Query App Service HTTP logs
  - Build a KQL query that shows API error rate over time
- Azure Monitor alerts for production:
  - Alert on API error rate > 1% over 5 minutes
  - Alert on average API response time > 500ms
  - Alert on AKS node CPU > 80% sustained for 10 minutes
- Course wrap-up — what you have built across all 9 phases:
  - Phase 0–1: account, portal, VMs, App Service, Functions
  - Phase 2: networking (VNet, load balancing, hybrid connectivity)
  - Phase 3: storage and all major database engines
  - Phase 4: identity, RBAC, secrets, monitoring
  - Phase 5: full Azure DevOps — Boards, Repos, Pipelines, Artifacts
  - Phase 6: IaC with Bicep and Terraform
  - Phase 7: containers with ACR, ACI, and AKS
  - Phase 8: full production-grade 3-tier application
- What's next — Azure certifications to consider:
  - AZ-900: Azure Fundamentals (concepts validation)
  - AZ-104: Azure Administrator (operations and management)
  - AZ-204: Azure Developer (building apps on Azure)
  - AZ-400: Azure DevOps Engineer Expert (advanced pipelines and DevOps practices)

## Hands-On Demo

**Account Requirements:** Continuation of the paid capstone infrastructure from Day 27. Instructor demo throughout.

- 💳 Trigger the full CI/CD pipeline end-to-end — watch it build, test, deploy, and wait for approval
- 💳 Show the live 3-tier application running: browser → App Service → AKS API → Azure SQL
- 💳 Simulate high API load and watch the HPA scale pods; see the Cluster Autoscaler add a node
- 💳 View Application Insights Live Metrics stream while the app handles traffic

## Summary

You now have a complete, production-grade Azure application with infrastructure as code, automated CI/CD, secrets management, and observability — built entirely on Azure. The path forward is certification: AZ-104 for administration, AZ-204 for development, AZ-400 for DevOps mastery. Thank you for learning with LearnWithMithran.
