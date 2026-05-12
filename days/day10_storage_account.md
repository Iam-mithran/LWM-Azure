# Day 9 — Azure Storage Account — Blob, Files, Queues, Tables

> Phase 3 — Storage + Databases

## Goal

Understand Azure's unified storage offering and work hands-on with Blob Storage.

## Key Topics

- Azure Storage Account: a single account that provides access to multiple storage services under one namespace
- Storage types within one account:
  - **Blob Storage**: store any unstructured data — files, images, videos, backups, logs. Organized into Containers → Blobs.
  - **Azure Files**: fully managed file shares accessible via SMB or NFS — mount them on VMs or local machines
  - **Queue Storage**: send and receive messages between application components (decoupled architecture)
  - **Table Storage**: simple NoSQL key-value store — fast and cheap for structured, non-relational data
- Replication options (what happens if a data center or region fails):
  - LRS (Locally Redundant Storage): 3 copies within one datacenter — cheapest
  - ZRS (Zone-Redundant Storage): copies across 3 availability zones in one region
  - GRS (Geo-Redundant Storage): LRS + async copy to a paired region
  - GZRS: ZRS + async copy to a paired region — highest durability
- Blob access tiers (optimize cost based on how often data is read):
  - Hot: frequent access — highest storage cost, lowest access cost
  - Cool: infrequent access (once/month) — lower storage cost
  - Cold: rare access (once/quarter) — even lower storage cost
  - Archive: long-term retention — lowest storage cost, hours to retrieve
- Storage Account performance tiers: Standard (HDD-backed) and Premium (SSD-backed)
- Blob types: Block Blob (general files), Append Blob (log files), Page Blob (VM disks)
- Shared Access Signatures (SAS): generate a time-limited URL that grants specific access to a blob or container — no need to share your account key
- Storage lifecycle management policies: automatically move blobs between tiers or delete them based on age

## Hands-On Demo

**Account Requirements:** 5GB of Blob Storage (LRS, Hot tier) is free for 12 months. All steps below are free tier.

- ✅ Create a Storage Account (choose LRS replication, Standard performance)
- ✅ Create a container and upload files to Blob Storage via the portal
- ✅ Generate a SAS URL and open the file in a browser to verify access
- ✅ Set up a lifecycle management policy (transition blobs to Cool tier after 30 days)

## Summary

A single Azure Storage Account gives you four different storage services. Blob Storage is the most commonly used — perfect for files, backups, and static content. Use access tiers and lifecycle policies to control costs as data ages.
