# Day 10 — Azure SQL Database — Managed Relational Database

> Phase 3 — Storage + Databases

## Goal

Deploy and connect to a fully managed SQL database on Azure — no server patching, no OS management.

## Key Topics

- Azure SQL Database: a fully managed relational database as a service built on SQL Server — Microsoft handles patching, backups, and high availability
- Deployment models:
  - Single Database: one dedicated database with its own resources
  - Elastic Pool: multiple databases that share a pool of resources — cost-efficient when databases have unpredictable or variable usage
  - Managed Instance: near 100% SQL Server compatibility — best for migrating on-premises SQL Server workloads
- Purchasing models:
  - DTU (Database Transaction Unit): bundled CPU + memory + IO — simpler, less flexible
  - vCore: choose CPU and memory separately — more control, eligible for Azure Hybrid Benefit savings
- Service tiers: Basic → Standard → Premium (DTU model) / General Purpose → Business Critical → Hyperscale (vCore model)
- Serverless compute tier: automatically pauses the database when inactive and resumes on the next connection — pay only for what you use
- Backups and restore: automatic backups every 5–12 minutes; point-in-time restore available within the retention period (7–35 days)
- Firewall rules: control which IP addresses can connect to your database
- Private endpoints: give your database a private IP inside your VNet — no public internet exposure
- Azure SQL vs SQL Server on a VM: choose Azure SQL for fully managed simplicity; choose SQL Server on VM for full OS/SQL Server access

## Hands-On Demo

**Account Requirements:** Azure SQL Database Serverless (General Purpose) has a free offer (32GB storage). Basic tier is very low cost. Instructor uses a paid account for richer tier features.

- ✅ Create an Azure SQL Database (choose Serverless or Basic tier — minimal cost)
- ✅ Configure the server firewall to allow your current IP address
- ✅ Connect using the Query Editor in the Azure portal
- ✅ Run basic SQL queries (CREATE TABLE, INSERT, SELECT)

## Summary

Azure SQL Database removes the burden of managing SQL Server infrastructure. Pick the Serverless tier to start — it auto-pauses when idle so you pay almost nothing during learning. Use Managed Instance when you need full SQL Server feature parity.
