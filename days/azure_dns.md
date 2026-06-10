# Day 12 — Azure DNS: Public & Private Zones

> Phase 3 — Networking

## Why This Topic Exists As Its Own Day

This content was originally part of the combined VNet day (Parts 5–6 of the old `day09_virtual_network.md`). During the Day 9–11 networking restructure, the VNet day was split into three videos:

- `day09_intro_to_networking.md` — IP fundamentals, bits/bytes, public/private IP, CIDR, classes
- `day10_virtual_network.md` — VNet core: address space, subnets, NSGs
- `day11_vnet_advanced.md` — VNet Peering, Service/Private Endpoints, Azure Bastion

Azure DNS (Public & Private Zones) was pulled out of `day11_vnet_advanced.md` to keep that video focused, and now has its own dedicated day as Day 12 — `course_outline.md` and `recording_plan.md` have been updated accordingly.

## Goal

Host a domain's public DNS records in Azure, and enable private name resolution inside a VNet.

## Key Topics

- What DNS is and why Azure hosts it — translating human-readable hostnames to IP addresses
- DNS record types: A, AAAA, CNAME, MX, TXT, NS, SOA
- Azure DNS Public Zones — Azure assigns four NS records per zone; point your domain registrar at them to delegate
- TTL (Time to Live) — how long resolvers cache a record, and trade-offs between short/long TTLs
- Azure DNS Private Zones — DNS that resolves only inside a VNet
- VNet links and autoregistration — every VM in a linked VNet automatically gets an A record
- Azure-provided DNS resolver (`168.63.129.16`) — what it resolves out of the box (public hostnames, Azure service hostnames, linked Private Zone records)
- Custom DNS — when you'd point a VNet at your own DNS server (split-horizon resolution, conditional forwarding for hybrid/on-prem scenarios)

## Hands-On Demo

**Account Requirements:** All free tier. Requires a VNet (Day 10) and at least one VM inside it for the resolution test.

- ✅ Create an Azure DNS Public Zone (any test domain/subdomain) and add an A record
- ✅ Create an Azure DNS Private Zone (e.g., `internal.learnwithmithran.com`)
- ✅ Link the Private Zone to `vnet-demo`, enable autoregistration, and confirm the existing VM gets an A record automatically
- ✅ Add a manual A record (e.g., `db → 10.0.2.10`)
- ✅ SSH into the VM and use `nslookup` to verify: the auto-registered VM record, the manual record, and that public DNS (`google.com`) still resolves via Azure's built-in resolver

## Summary

Azure DNS Public Zones host your domain's records at global scale — delegate via NS records at your registrar. Azure DNS Private Zones give your VNet internal name resolution: link the zone, enable autoregistration, and every VM gets a hostname automatically. Together they cover both "the world reaching you" and "your resources reaching each other by name."
