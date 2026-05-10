# Day 3 — Azure Virtual Machines — Part 2: VM Management & Availability

> Phase 1 — Compute

## Goal

Manage VMs like a professional — availability, scaling, snapshots, and secure access.

## Key Topics

- Availability Sets vs Availability Zones: how Azure protects your VMs from hardware and datacenter failures
- VM Scale Sets (VMSS): deploy and auto-scale a group of identical VMs based on demand; used with Load Balancer and Application Gateway as the backend pool
- VM snapshots and custom images: capture a VM's disk state to back it up or create new VMs from it
- Custom Script Extensions: run scripts on a VM automatically after deployment
- Azure Bastion: secure, browser-based SSH/RDP access to your VM without exposing a public IP address
- VM monitoring basics: CPU, memory, disk metrics visible directly in the portal

### Azure Backup for VMs

- **Azure Backup**: Microsoft's native backup-as-a-service — backs up VMs, SQL databases, Azure Files, and more without managing backup infrastructure
- **Recovery Services Vault**: the container that stores all backup data; you create one vault per region and register your VMs to it
- **Backup policy**: defines the schedule (daily/weekly) and retention period (how many days/weeks/months to keep recovery points)
- **Recovery points**: each backup creates a recovery point — a point-in-time snapshot you can restore from
- **Restore options**:
  - Replace existing VM (overwrites the running VM with the backed-up state)
  - Create new VM (restore as a brand-new VM — leaves the original untouched)
  - Restore disks only (restore the managed disk, attach it manually)
  - File-level recovery (mount the backup as a drive and restore individual files)
- **Backup cost**: charged for storage used by recovery points + a small per-VM protected instance fee
- **Soft delete**: deleted backups are retained for 14 days before permanent removal — protects against accidental or malicious deletion

## Hands-On Demo

**Account Requirements:** Snapshots and VM backup are low-cost (storage only). Azure Bastion is paid — instructor demo. Alerts are free.

- ✅ Create a VM snapshot (Disks → Create snapshot)
- ✅ Create a Recovery Services Vault and enable Azure Backup for the VM (configure daily backup policy, 7-day retention)
- ✅ Trigger an on-demand backup and watch the backup job complete in the portal
- ✅ Browse recovery points and explore the restore options (do not restore — just explore the wizard)
- 💳 Enable Azure Bastion and connect to the VM via browser (Bastion is paid ~$0.19/hr — instructor demo; students can skip or use SSH instead)
- ✅ Set up a basic alert on CPU > 80% (Azure Monitor → Alerts)

## Summary

Availability Sets and Zones protect your apps from failures. VM Scale Sets handle traffic spikes automatically. Azure Backup gives you a fully managed backup service — create a Recovery Services Vault, attach a policy, and Azure handles scheduling, retention, and storage. Always have a backup policy on any VM holding important data.
