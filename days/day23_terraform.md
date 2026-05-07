# Day 23 — Terraform on Azure

> Phase 6 — Infrastructure as Code (IaC)

## Goal

Use Terraform — the open-source, multi-cloud IaC tool — to provision Azure infrastructure with a remote state backend.

## Key Topics

- What is Terraform? An open-source IaC tool by HashiCorp that uses declarative configuration files to manage infrastructure across any cloud provider
- Azure Provider for Terraform (`azurerm`): the official plugin that translates Terraform resource definitions into Azure API calls
- Authenticating Terraform with Azure:
  - Azure CLI login (`az login`): simplest for local development — Terraform picks up your credentials
  - Service Principal: a dedicated app identity with a client ID and secret — used in automated pipelines
  - Managed Identity: when running Terraform from an Azure-hosted agent or VM — no credentials needed
- Terraform remote state on Azure:
  - By default Terraform stores state locally in `terraform.tfstate` — unsafe for teams
  - Store state in an Azure Blob Storage container — enables locking (only one person runs apply at a time) and team collaboration
  - Configuration: `backend "azurerm"` block pointing to your storage account and container
- Core `azurerm` resources:
  - `azurerm_resource_group`: create a resource group
  - `azurerm_virtual_network`: create a VNet
  - `azurerm_subnet`: create subnets inside the VNet
  - `azurerm_linux_virtual_machine`: provision a Linux VM
- Terraform workspaces: separate state files for dev, staging, and production within the same configuration — run `terraform workspace new dev` to create one
- Integrating Terraform with Azure DevOps Pipelines:
  - On PR: run `terraform plan` and post the output as a comment — reviewers see what infrastructure will change
  - On merge to main: run `terraform apply` automatically

## Hands-On Demo

**Account Requirements:** Terraform itself is free. Resources deployed (Resource Group, VNet, VM) cost the same as creating them manually. B1s VM is free tier (750 hrs/month).

- ✅ Write Terraform configuration: Resource Group + VNet + Subnet + Linux VM
- ✅ Configure Azure Blob Storage as the remote state backend
- ✅ Run `terraform init` → `terraform plan` → `terraform apply`
- ✅ Add a `terraform plan` step to an Azure DevOps pipeline

## Summary

Terraform is the go-to tool when you need to manage infrastructure across multiple clouds or prefer a mature ecosystem with a large community. Store state in Azure Blob Storage from day one — local state files are a liability in any team environment.
