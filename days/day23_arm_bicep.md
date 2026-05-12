# Day 22 — ARM Templates & Bicep

> Phase 6 — Infrastructure as Code (IaC)

## Goal

Define Azure infrastructure as code using Azure-native tools — create, version, and repeat deployments reliably.

## Key Topics

- What is Infrastructure as Code (IaC)? Define your infrastructure in text files instead of clicking through the portal — reproducible, version-controlled, reviewable in PRs
- ARM Templates: Azure's native IaC format — JSON files that describe the desired state of your Azure resources
  - Structure: `$schema`, `parameters`, `variables`, `resources`, `outputs`
  - Idempotent: deploy the same template twice and Azure only makes changes if something differs
  - Limitation: verbose JSON is hard to read and write — easy to make mistakes in large templates
- Azure Bicep: a domain-specific language (DSL) that compiles down to ARM JSON — much cleaner syntax, same power
  - Bicep is not a different deployment engine — it's a better way to write ARM
  - Why choose Bicep over raw ARM: less boilerplate, type checking, modules, better IDE support
- Bicep syntax essentials:
  - `resource`: declares an Azure resource
  - `param`: defines an input parameter (with type, default, and allowed values)
  - `var`: defines a reusable value
  - `module`: reuse a Bicep file as a building block inside another file
  - `output`: expose values after deployment (e.g., the URL of a deployed app)
- Deploying Bicep files: `az deployment group create --template-file main.bicep`
- What-if deployments: `az deployment group what-if` — shows you exactly what will change before you apply — like a dry run
- Azure Deployment Stacks: manage a group of resources deployed together as a single unit — enforce that no one can add resources outside the stack

## Hands-On Demo

**Account Requirements:** Deploying a Storage Account via Bicep costs the same as creating it manually. Storage Account (LRS, Standard) is free tier. Azure CLI is free to install.

- ✅ Write a Bicep file that deploys a Storage Account
- ✅ Run `az deployment group what-if` to preview changes before deploying
- ✅ Deploy with `az deployment group create` (one of the few CLI moments in this course)
- ✅ Modify a parameter (e.g., change the SKU) and redeploy — observe that Azure only updates what changed (idempotency in action)

## Summary

Bicep is the right choice for Azure-native IaC today. It compiles to ARM, so you get full Azure platform support with far better developer experience. Use what-if before every deployment in production — it is the equivalent of reviewing a diff before merging code.
