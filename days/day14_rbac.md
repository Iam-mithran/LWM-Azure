# Day 14 — Azure RBAC — Role-Based Access Control

> Phase 4 — Identity, Security + Monitoring

## Goal

Control who can do what in Azure using roles and assignments — implement the principle of least privilege.

## Key Topics

- Azure RBAC model: three components work together
  - **Who** (Security Principal): a user, group, service principal, or managed identity
  - **What** (Role Definition): a collection of permissions (actions, not-actions, data actions)
  - **Where** (Scope): the level at which the role applies — Management Group, Subscription, Resource Group, or individual Resource
- Built-in roles:
  - **Owner**: full access including the ability to grant access to others
  - **Contributor**: full access to create and manage resources, but cannot grant access
  - **Reader**: view-only access to all resources
  - 70+ service-specific roles (e.g., Storage Blob Data Contributor, Key Vault Secrets User)
- Custom roles: define your own role with exactly the permissions you need — useful when built-in roles are too broad or too narrow
- Role assignments: the act of attaching a role definition to a security principal at a specific scope — assignments are additive (a user gets the union of all assigned roles)
- Deny assignments: explicitly block specific actions regardless of what roles are assigned — currently only used by Azure Blueprints and managed applications
- Azure Policy: define rules that enforce compliance across your subscriptions (e.g., "all resources must have a CostCenter tag", "VMs must be in approved regions")
  - Policy effects: Audit (log non-compliance), Deny (block the action), DeployIfNotExists (auto-remediate)
- Azure Blueprints: package RBAC assignments, policies, ARM templates, and resource groups into a repeatable governance template — apply it to new subscriptions instantly
- Principle of Least Privilege: assign the minimum permissions needed for the task, at the narrowest scope

## Hands-On Demo

**Account Requirements:** All steps are free tier.

- ✅ Assign the Reader role to a user at Resource Group scope (IAM → Add role assignment)
- ✅ Create a custom role with specific permissions (e.g., start/stop VMs only)
- ✅ Create an Azure Policy to require a CostCenter tag on all resources
- ✅ Verify policy enforcement by attempting to create a resource without the required tag

## Summary

RBAC answers the question: who can do what, where? Always assign roles at the narrowest scope possible. Azure Policy enforces governance rules at scale — it's how you prevent misconfigurations before they happen, not after.
