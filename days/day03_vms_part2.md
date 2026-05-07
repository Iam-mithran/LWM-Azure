# Day 3 — Azure Virtual Machines — Part 2: VM Management & Availability

> Phase 1 — Compute

## Goal

Manage VMs like a professional — availability, scaling, snapshots, and secure access.

## Key Topics

- Availability Sets vs Availability Zones: how Azure protects your VMs from hardware and datacenter failures
- VM Scale Sets: deploy and auto-scale a group of identical VMs based on demand
- VM snapshots and custom images: capture a VM's disk state to back it up or create new VMs from it
- Custom Script Extensions: run scripts on a VM automatically after deployment
- Azure Bastion: secure, browser-based SSH/RDP access to your VM without exposing a public IP address
- VM monitoring basics: CPU, memory, disk metrics visible directly in the portal
- Shutting down vs deallocating: the common cost trap beginners fall into

## Hands-On Demo

**Account Requirements:** Snapshots and alerts are free tier. Azure Bastion is a paid service — instructor will demonstrate; students should watch or skip this step.

- ✅ Create a VM snapshot (Disks → Create snapshot)
- 💳 Enable Azure Bastion and connect to the VM via browser (Bastion is paid ~$0.19/hr — instructor demo; students can skip or use SSH instead)
- ✅ Set up a basic alert on CPU > 80% (Azure Monitor → Alerts)

## Summary

Availability Sets and Zones protect your apps from failures. Scale Sets let you handle traffic spikes automatically. Azure Bastion is the secure way to connect to VMs without exposing SSH ports to the internet — though it comes with a cost.
