# Day 22 — Azure Pipelines — CI (Continuous Integration)

> Phase 6 — Azure DevOps

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

**Account Requirements:** All steps below are free tier — **but only if the Microsoft-hosted parallelism
grant was enabled back on Day 20.**

!!! danger "Verify this before recording, and make the student verify it too"
    The free grant is **one Microsoft-hosted job, 1,800 minutes/month, with a 60-minute cap per job** —
    and it is **not** granted automatically to a new organization. If Day 20's billing-linking step was
    skipped, every pipeline in this day fails immediately with:

    ```text
    No hosted parallelism has been purchased or granted
    ```

    The fix is **Organization settings → Billing → Set up billing** (link an Azure subscription), and if
    the automatic grant is still withheld, the **parallelism request form**, which takes **~4–5 business
    days**. That is a week of being blocked, so this day must open by confirming
    **Organization settings → Pipelines → Parallel jobs** shows a Microsoft-hosted job available.

    See the corrected section in `days/devops_intro.md` for the full detail. Note also that **public
    projects are retired** — the old "make it public for unlimited free minutes" workaround no longer
    exists.

- ✅ Create an `azure-pipelines.yml` file in your repo
- ✅ Configure trigger: run on every push to main
- ✅ Add steps: checkout → install Node.js → install dependencies → run tests → publish artifact
- ✅ Run the pipeline and review the logs, test results, and published artifact

## Summary

A CI pipeline runs automatically on every push so you never merge broken code into main. YAML pipelines are the right choice — your build definition lives in version control alongside your app. Start simple: checkout → install → test → publish.
