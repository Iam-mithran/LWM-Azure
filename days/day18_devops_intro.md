# Day 17 — Azure DevOps Introduction & Organization Setup

> Phase 5 — Azure DevOps

## Goal

Set up Azure DevOps from zero and understand what each of its five services does.

## Key Topics

- What is Azure DevOps? Microsoft's integrated platform for planning, developing, testing, and delivering software — everything your team needs in one place
- The five services inside Azure DevOps:
  - **Boards**: plan and track work with backlogs, sprints, Kanban boards, and work items (epics → features → user stories → tasks)
  - **Repos**: host Git repositories — unlimited private repos, pull requests, branch policies, code reviews
  - **Pipelines**: automate build, test, and deployment — trigger on code changes, run on Microsoft-hosted or self-hosted agents
  - **Test Plans**: manage and execute manual and automated tests, track results and coverage
  - **Artifacts**: host private package feeds for npm, NuGet, Maven, PyPI, and Universal Packages
- Azure DevOps vs GitHub Actions: both can do CI/CD; Azure DevOps is better for organizations that need integrated project management (Boards) and a full suite in one place; GitHub Actions is simpler for open-source or GitHub-native projects
- Organizations and Projects:
  - Organization: the top-level container — usually one per company (e.g., dev.azure.com/yourcompany)
  - Project: a workspace inside an organization — one per team or product area; contains its own Boards, Repos, Pipelines, etc.
- Linking Azure DevOps to an Azure Subscription: required for Pipelines to deploy resources to Azure — done via a Service Connection
- Free tier: 5 users free (Basic plan), 1 Microsoft-hosted pipeline with 1,800 minutes/month free, unlimited private Git repositories

## Hands-On Demo

**Account Requirements:** Free tier is sufficient for all steps below.

- ✅ Create an Azure DevOps organization at dev.azure.com
- ✅ Create a project using the Agile process template
- ✅ Set up Boards: create an epic, user stories, and tasks
- ✅ Invite a team member (or explore the invitation flow)

## Summary

Azure DevOps brings your entire software delivery lifecycle into one platform. Start with Boards to plan, Repos to store your code, and Pipelines to automate — you can add Test Plans and Artifacts as your team grows. The free tier is generous enough to run a real project.
