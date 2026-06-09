# Day 6 — Azure Virtual Network (VNet) — Core Networking

> Phase 2 — Networking

## Goal

Understand Azure networking from scratch and create a working VNet with subnets.

## Key Topics

- What is a Virtual Network (VNet)? Your private, isolated network inside Azure — all resources you create communicate through it
- VNet address space: an IP range (CIDR block) that defines all available addresses in your network (e.g., 10.0.0.0/16)
- Subnets: subdivisions of a VNet that let you segment resources (e.g., web servers in one subnet, databases in another)
- Public vs Private subnets: resources in a public subnet can reach the internet; private subnet resources cannot (unless routed through a gateway)
- Network Security Groups (NSG): a set of inbound and outbound traffic rules attached to a subnet or NIC
  - Rules have a priority number (lower = evaluated first), action (Allow/Deny), protocol, port range, and source/destination
- VNet Peering: connect two VNets so resources in each can communicate privately — no internet, no gateway needed
- Service Endpoints: extend your VNet's private address space to reach Azure services (like Storage or SQL) directly
- Private Endpoints: give an Azure service a private IP address inside your VNet — the most secure way to access Azure services
- Azure DNS — Public Zones: host your domain's DNS records in Azure (A, CNAME, MX, TXT, NS) — Azure manages the DNS infrastructure at global scale
- Azure DNS — Private Zones: private name resolution inside a VNet — resources resolve each other by name (e.g., `db.internal`) without needing custom DNS servers; link private zones to VNets for automatic registration
- Custom DNS on a VNet: point a VNet to your own DNS server (on-premises or a VM-based DNS) instead of Azure's default resolver
- Azure-provided DNS vs custom DNS: Azure gives every VNet a built-in resolver at `168.63.129.16` — sufficient for most scenarios; bring your own DNS only when you need split-horizon or conditional forwarding
- Azure Bastion: a fully managed PaaS service that lives inside a dedicated `AzureBastionSubnet` in your VNet and lets you connect to VMs over HTTPS (port 443) from the browser — no public IP on the VM, no exposed port 22 or 3389; paid service (~$0.19/hr Basic SKU)

## Hands-On Demo

**Account Requirements:** VNets, subnets, and NSGs are all free. All steps below are free tier.

- ✅ Create a VNet with 2 subnets (name them public-subnet and private-subnet)
- ✅ Attach an NSG to the subnet
- ✅ Create NSG inbound rules to allow SSH (port 22) and HTTP (port 80)
- ✅ Deploy a VM into the private subnet
- ✅ Create an Azure DNS Public Zone (use a test domain or subdomain) and add an A record pointing to a VM's public IP
- ✅ Create an Azure DNS Private Zone, link it to the VNet, and verify private name resolution from inside a VM
- 💳 Deploy Azure Bastion into an `AzureBastionSubnet` and connect to a VM via the browser (Instructor Demo — ~$0.19/hr)

## Summary

A VNet is the foundation of every Azure network. Subnets segment your resources, NSGs control traffic flow, and VNet Peering lets multiple VNets talk to each other. Getting this right is essential before building anything multi-tier.
