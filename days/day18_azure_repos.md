# Day 18 — Azure Repos — Git Version Control in Azure DevOps

> Phase 5 — Azure DevOps

## Goal

Use Azure Repos for version control and set up branch policies for safe, collaborative development.

## Key Topics

- Azure Repos: fully managed Git repositories hosted inside Azure DevOps — unlimited private repos included in all tiers
- Git vs TFVC (Team Foundation Version Control): always choose Git — it is the industry standard, supports distributed development, and all Azure DevOps features are Git-first
- Creating and cloning a repository: create via the Azure DevOps portal, then clone to your local machine with standard git commands
- Branching strategies: how your team organizes work in branches
  - Feature branches: one branch per feature or bug fix, merged back to main via PR
  - GitFlow: main + develop + feature/release/hotfix branches — structured, good for scheduled releases
  - Trunk-based development: short-lived branches or direct commits to main — enables continuous integration
- Branch policies: rules that protect your main branch and enforce quality gates
  - Require a Pull Request before merging (no direct pushes to main)
  - Minimum number of reviewers
  - Link a work item (ties code changes to Boards tasks)
  - Build validation (a pipeline must pass before the PR can complete)
- Pull Requests (PRs) in Azure DevOps:
  - Create a PR from your feature branch to main
  - Reviewers leave comments, approve, or request changes
  - Required checks (policies) must pass before the PR can be completed
  - Completion options: merge, squash merge, or rebase
- Connecting VS Code to Azure Repos: use the built-in Git integration or the Azure Repos extension — clone, push, and create PRs directly from the editor

## Hands-On Demo

**Account Requirements:** All steps are free tier (unlimited private repos).

- ✅ Create a new repository in Azure DevOps
- ✅ Push a sample app (Node.js or Python) from your local machine
- ✅ Create a feature branch, make a code change, and open a PR
- ✅ Set up a branch policy on main: require at least 1 reviewer
- ✅ Complete the PR and merge to main

## Summary

Azure Repos gives you enterprise-grade Git hosting inside the same platform as your pipelines and boards. Branch policies are the guardrail that keeps main stable — set up the PR requirement and build validation before your first real project goes live.
