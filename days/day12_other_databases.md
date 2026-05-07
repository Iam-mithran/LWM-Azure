# Day 12 — Other Azure Databases + Storage Deep Dive

> Phase 3 — Storage + Databases

## Goal

Survey the remaining database options in Azure and explore advanced storage features.

## Key Topics

- Azure Database for PostgreSQL (Flexible Server): fully managed PostgreSQL — choose your version, scale independently, built-in high availability
- Azure Database for MySQL (Flexible Server): fully managed MySQL — similar flexibility and scaling options
- Azure Cache for Redis: fully managed in-memory data store for caching, session management, and real-time leaderboards
- Azure Disk Storage — Managed Disks: the block storage attached to VMs
  - Standard HDD: lowest cost, suitable for dev/test and infrequently accessed data
  - Standard SSD: better performance, suitable for web servers and lightly loaded apps
  - Premium SSD: high performance for production workloads (required for many VM sizes)
  - Ultra Disk: extreme low-latency and high-throughput for mission-critical databases
- Azure NetApp Files: enterprise-grade, high-performance NFS and SMB file shares — built for SAP, HPC, and latency-sensitive workloads
- Storage security:
  - Encryption at rest: enabled by default on all Azure Storage — no configuration needed
  - Encryption in transit: enforced via HTTPS
  - Customer-Managed Keys (CMK): bring your own key stored in Azure Key Vault for additional control
- Azure Storage Explorer: free desktop tool to manage blobs, files, queues, and tables — drag-and-drop, cross-platform

## Hands-On Demo

**Account Requirements:** PostgreSQL Flexible Server (Burstable B1ms) is low-cost. Azure Cache for Redis is a paid service — instructor will demonstrate.

- ✅ Create an Azure Database for PostgreSQL — Flexible Server (Burstable tier — lowest cost option)
- ✅ Connect via the built-in query editor or pgAdmin
- 💳 Enable Azure Cache for Redis and test basic SET/GET commands via the Redis Console in the portal (Redis is paid — instructor demo; students can watch)

## Summary

Azure offers a managed database for every engine: PostgreSQL, MySQL, Redis, and more. Managed Disks underpin VM storage with multiple performance tiers. All Azure storage is encrypted at rest by default — security is baked in, not bolted on.
