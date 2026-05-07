# Day 19 — Azure Pipelines — CI (Continuous Integration)

> Phase 5 — Azure DevOps

## Goal

Build an automated CI pipeline that runs on every code push — compile, test, and package your app automatically.

## Key Topics

- What is CI (Continuous Integration)? Automatically build and test your code every time someone pushes a change — catch bugs early, not during deployment
- Azure Pipelines architecture: define your entire build process as code in a YAML file committed to your repo
- YAML pipelines vs Classic (GUI) pipelines: always use YAML — it is version-controlled, reviewable in PRs, and portable
- Pipeline anatomy:
  - **Trigger**: what event starts the pipeline (push to main, PR, schedule, manual)
  - **Pool**: which agent runs the pipeline
  - **Stages**: major phases (Build, Test, Deploy) — optional grouping
  - **Jobs**: a unit of work that runs on one agent; stages contain jobs
  - **Steps**: individual actions within a job — a step is either a script or a Task
  - **Tasks**: pre-built actions (e.g., NodeTool@0, DotNetCoreCLI@2, PublishBuildArtifacts@1)
- Microsoft-hosted agents: virtual machines provisioned and managed by Microsoft for each pipeline run — pay per minute, or use the free 1,800 min/month
- Self-hosted agents: your own VMs or containers that you register with Azure DevOps — useful for custom software, faster builds, or private network access
- Common pipeline steps: checkout source code → install runtime → install dependencies → run tests → publish artifacts
- Pipeline variables and variable groups: store reusable values (build version, environment name) without hardcoding
- Secrets in pipelines: link a Key Vault to a Variable Group — pipeline reads secrets at runtime without them being visible in logs
- Build artifacts: the output of your build (compiled binaries, Docker images, zip files) — published and stored for later use in a release pipeline

## Hands-On Demo

**Account Requirements:** Free tier includes 1 Microsoft-hosted agent with 1,800 minutes/month. All steps below are free tier.

- ✅ Create an `azure-pipelines.yml` file in your repo
- ✅ Configure trigger: run on every push to main
- ✅ Add steps: checkout → install Node.js → install dependencies → run tests → publish artifact
- ✅ Run the pipeline and review the logs, test results, and published artifact

## Summary

A CI pipeline runs automatically on every push so you never merge broken code into main. YAML pipelines are the right choice — your build definition lives in version control alongside your app. Start simple: checkout → install → test → publish.
