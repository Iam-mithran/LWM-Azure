# Day 27 — Capstone Part 1: Architecture Design & Infrastructure Setup

> Phase 8 — Capstone Project

## Goal

Design and build the infrastructure for a real-world 3-tier application on Azure — applying everything from Phases 0 through 7.

## Capstone Application

A 3-tier web application:
- **Frontend**: Azure App Service (serves the web UI)
- **API Backend**: AKS (containerized API, scales independently)
- **Data Layer**: Azure SQL Database (relational data store)

## Key Topics

- Architecture design walkthrough — understanding every component and why it was chosen:
  - Azure App Service: managed hosting for the frontend — no server management
  - AKS: container orchestration for the API — independent scaling, rolling updates
  - Azure SQL Database: managed relational database with automatic backups
  - Azure Key Vault: all secrets (DB connection string, API keys) stored here — no hardcoded credentials anywhere
  - Azure Container Registry (ACR): private registry for the API's Docker images
  - Azure Monitor + Application Insights: end-to-end observability across all tiers
  - Virtual Network with subnets: frontend subnet, backend subnet, database subnet with private endpoint
  - Entra ID + Managed Identities + RBAC: App Service and AKS authenticate to Key Vault and ACR without credentials
- Infrastructure provisioning with Terraform (or Bicep): the entire infrastructure defined as code, version-controlled, and repeatable
- Networking design: VNet with separate subnets for each tier; NSGs controlling inter-tier traffic; private endpoint for Azure SQL so the database is never exposed to the public internet
- Secrets management: all connection strings and keys go into Key Vault; App Service and AKS pods use Managed Identity to read them at runtime

## Hands-On Demo

**Account Requirements:** This capstone uses multiple paid resources simultaneously — AKS worker nodes, ACR, App Service (at minimum Basic tier), Azure SQL. Instructor runs all steps on a paid account. Students can follow along with free trial credits or watch and build their own architecture choices.

- 💳 Provision the full infrastructure with Terraform (Resource Group, VNet, subnets, NSGs, App Service Plan)
- 💳 Deploy Azure SQL Database with a private endpoint (database accessible only from inside the VNet)
- 💳 Set up Key Vault and store the database connection string as a secret
- 💳 Create ACR and push the API Docker image

## Summary

Good architecture is not about using every service — it is about using the right service for each layer. This capstone applies the smallest set of services that gives you security (Managed Identity, private endpoint, Key Vault), scalability (AKS, App Service auto-scale), and observability (Application Insights). Infrastructure as code ensures you can tear it down and rebuild it in minutes.
