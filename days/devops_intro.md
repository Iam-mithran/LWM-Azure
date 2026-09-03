# Day 20 — Azure DevOps Introduction & Organization Setup

> Phase 6 — Azure DevOps

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
- Free tier: **5 users free** (Basic plan), **unlimited private Git repositories**, and a free Microsoft-hosted
  pipeline grant — **but that pipeline grant is not automatic. See the warning below.**

## ⚠️ CORRECTED — Verified September 2026

This file previously said the free tier includes *"1 Microsoft-hosted pipeline with 1,800 minutes/month free"*
as though it arrives with the organization. **It does not, and this breaks the Day 22 lab if you don't
handle it on Day 20.**

Microsoft's documentation is explicit: *"The self-hosted free tier is automatically granted, **but you must
enable the Microsoft-hosted free tier**."*

**What is actually true:**

| Claim | Reality |
|---|---|
| 1,800 minutes/month | Correct — **but only once enabled**, and with a **60-minute cap per individual job** |
| Arrives with a new org | **False.** A brand-new organization has **zero** Microsoft-hosted parallelism |
| What you'll see instead | `No hosted parallelism has been purchased or granted` — the pipeline fails before running a step |
| The documented fix | **Link the organization to a valid Azure subscription** (Organization settings → Billing → Set up billing). The free grant then applies automatically to private projects. |
| If that doesn't work | Microsoft withholds the automatic grant from some new organizations as an anti-abuse control (crypto mining). The fallback is the **Azure DevOps parallelism request form**, which takes **~4–5 business days**. |

**⚠️ Recording and teaching implication — this must be Part 1 or 2 of Day 20, not a footnote in Day 22.**
Students need to link billing (and, if refused, submit the form) **days before** they reach Pipelines on
Day 22, or they will be blocked for a working week with a pipeline that fails on its first run. Set the
expectation on Day 20 while they are creating the organization anyway.

**Also corrected: public projects are retired.** New public projects **can no longer be created**, and
existing ones **convert to private in 2027**. A great deal of Azure DevOps tutorial content still says
"make the project public for unlimited free build minutes" — that advice is dead, and the script should
say so, because students will find it.

**Minor:** new organizations are capped at **25 parallel jobs** for Microsoft-hosted agents.

## Hands-On Demo

**Account Requirements:** Free tier is sufficient for all steps below. **One step — linking billing — is
time-critical: it unblocks Day 22 and may need several days' lead time.**

- ✅ Create an Azure DevOps organization at dev.azure.com
- ✅ Create a project using the Agile process template — note that **Public is no longer offered**
- ✅ **Organization settings → Billing → Set up billing**, linking a valid Azure subscription. **This is what
  enables the free Microsoft-hosted pipeline grant. Do it today, not on Day 22.**
- ✅ **Organization settings → Pipelines → Parallel jobs** — confirm Microsoft-hosted shows **1** free job.
  If it shows 0, submit the parallelism request form immediately (~4–5 business days)
- ✅ Set up Boards: create an epic, user stories, and tasks
- ✅ Invite a team member (or explore the invitation flow)

## Summary

Azure DevOps brings your entire software delivery lifecycle into one platform. Start with Boards to plan, Repos to store your code, and Pipelines to automate — you can add Test Plans and Artifacts as your team grows. The free tier is generous enough to run a real project — **provided you link billing to enable the Microsoft-hosted pipeline grant, which is the single most important thing to do on Day 20.**
