# Day 8 — VPN Gateway & ExpressRoute

> Phase 2 — Networking

## Goal

Connect an on-premises network (or your laptop) to Azure securely using hybrid networking.

## Key Topics

- Hybrid networking: connecting your existing office/data center network to Azure so both sides can communicate privately
- Azure VPN Gateway: establishes an encrypted IPsec/IKE tunnel over the public internet between Azure and your on-premises network
  - Site-to-Site VPN: connects an entire office/data center network to Azure
  - Point-to-Site VPN: connects a single laptop or workstation to an Azure VNet
  - VPN Gateway SKUs: VpnGw1 through VpnGw5 (higher SKU = more throughput and tunnels)
- ExpressRoute: a private, dedicated circuit provisioned through a connectivity provider — never touches the public internet
  - Ideal for: high bandwidth, low latency, regulatory requirements
  - Requires a physical circuit from a partner provider (takes days/weeks to set up)
- VPN vs ExpressRoute trade-offs:
  - VPN: cheaper, faster to set up, goes over internet (encrypted), lower bandwidth cap
  - ExpressRoute: expensive, slower to provision, fully private, higher and more consistent bandwidth
- Azure Virtual WAN: Microsoft-managed hub-and-spoke networking for large organizations with many branches

## Hands-On Demo

**Account Requirements:** VPN Gateway is a paid resource (~$27/month for the lowest SKU). ExpressRoute requires a physical circuit. Both are instructor-only demos — students should watch.

- 💳 Create a VPN Gateway in the portal (note: deployment takes 30–45 minutes — explain the architecture while it provisions)
- 💳 Walk through the Point-to-Site VPN configuration (certificate generation, client download)
- 💳 Show the ExpressRoute blade in the portal — conceptual walkthrough of how a circuit is ordered (no physical line required for the tour)

## Summary

VPN Gateway is your quick, affordable path to hybrid connectivity — great for learning and small offices. ExpressRoute is the enterprise-grade option when you need private, high-bandwidth, SLA-backed connectivity between Azure and your data center.
