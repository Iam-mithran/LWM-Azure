# Day 2 — Azure Virtual Machines — Part 1: Creating Your First VM

> Phase 1 — Compute

## Goal

Create and connect to an Azure VM from scratch.

## Key Topics

- What is Azure Virtual Machines? Renting a computer in Microsoft's data center that you control completely
- VM sizes and series:
  - B-series: Burstable, cost-effective for low-to-medium workloads
  - D-series: General purpose, balanced CPU/memory
  - F-series: Compute optimized
  - E-series: Memory optimized
- VM pricing models: Pay-as-you-go, Reserved Instances (1 or 3 year commitment), Spot VMs (use spare capacity at lower cost)
- OS options: Windows Server, Ubuntu, RHEL, and many more from Marketplace
- Managed Disks: OS disk (where the OS lives) and Data disks (additional storage you attach)
- Public IP addresses and DNS labels
- Network Security Groups (NSG) — the firewall rules that control what traffic can reach your VM

## Hands-On Demo

**Account Requirements:** B1s VM (1 vCPU, 1GB RAM) is included in the Azure Free Tier (750 hours/month for 12 months). All steps below are free tier.

- ✅ Create an Ubuntu VM via Azure Portal (choose B1s size)
- ✅ Configure size, disk, and NSG rules (open port 22 for SSH)
- ✅ Connect to the VM via SSH
- ✅ Stop and deallocate the VM (explain cost saving: stopped ≠ deallocated)

## Summary

VMs give you full control over a virtualized computer in the cloud. You pick the OS, size, and disk, and Azure handles the physical hardware. Always deallocate (not just stop) to avoid being billed for compute when not in use.
