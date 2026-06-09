# Day 7 — Azure Load Balancer & VM Scale Sets

> Phase 2 — Networking

## Goal

Distribute traffic across multiple VMs automatically, and scale the number of VMs up and down based on demand.

## Key Topics

- **Azure Load Balancer (Layer 4 — TCP/UDP):** distributes incoming connections across a pool of backend VMs using a 5-tuple hash (source IP, source port, destination IP, destination port, protocol)
  - SKUs: Basic (free, retiring) and Standard (paid, recommended for production)
  - Components:
    - **Frontend IP configuration**: the public IP that clients connect to
    - **Backend pool**: the group of VMs receiving traffic
    - **Health probe**: periodic checks (HTTP/TCP) to detect unhealthy VMs and remove them from rotation
    - **Load balancing rule**: maps a frontend port to a backend port with a protocol
    - **Inbound NAT rules**: forward a specific port on the frontend IP to a specific VM — used for direct SSH/RDP access to individual backend VMs
  - Internal vs Public Load Balancer: Public LB has a public IP (internet-facing); Internal LB has a private VNet IP (for distributing traffic between tiers inside a VNet)
  - Session persistence: none (default — each connection may go to a different VM) or client IP (same client always goes to same VM)

- **VM Scale Sets (VMSS):** deploy and manage a group of identical load-balanced VMs as a single resource
  - All VMs in a Scale Set run the same OS image and configuration
  - **Autoscale**: Azure automatically adds or removes VM instances based on rules (CPU %, custom metrics, or a schedule)
    - Scale out: add VMs when load increases
    - Scale in: remove VMs when load drops
  - **Orchestration modes**:
    - Uniform: identical VMs, optimized for stateless workloads (web servers, batch jobs)
    - Flexible: mix different VM sizes and configurations in the same scale set
  - **Integration with Load Balancer**: VMSS registers its instances automatically as the Load Balancer's backend pool — no manual VM management needed
  - **Update policy**: how new images are rolled out — Manual, Automatic, or Rolling (replace VMs gradually, never taking the whole fleet down at once)
  - **Overprovisioning**: Azure creates extra VMs during a scale-out, keeps the fastest ones, discards the rest — reduces the risk of a VM creation failure causing a scaling event to fail

## Hands-On Demo

**Account Requirements:** Standard Load Balancer is a paid resource (~$0.005/hr for the frontend IP). VMSS VMs are billed per instance. Instructor will demonstrate all steps — students should watch or use free trial credits.

- 💳 Create a VM Scale Set (Uniform orchestration, Ubuntu 24.04, B1s size, 2 initial instances)
- 💳 Configure autoscale rules: scale out when average CPU > 70% for 5 minutes; scale in when CPU < 30% for 10 minutes
- 💳 Create a Standard Load Balancer with a public frontend IP
- 💳 Attach the VMSS as the Load Balancer backend pool
- 💳 Configure a health probe (HTTP port 80) and a load balancing rule
- 💳 Install Nginx on all VMSS instances using a custom script extension and test traffic distribution via the Load Balancer's public IP
- 💳 Manually trigger a scale-out and watch new instances appear in the portal

## Summary

Load Balancer distributes traffic across your VMs at Layer 4 (TCP/UDP) — simple, fast, and handled entirely by Azure. VM Scale Sets let you manage a fleet of identical VMs as one resource and scale automatically based on real demand. Together, they are the standard pattern for scalable, resilient compute in Azure.
