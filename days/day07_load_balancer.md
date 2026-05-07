# Day 7 — Azure Load Balancer & Application Gateway

> Phase 2 — Networking

## Goal

Distribute traffic across multiple VMs and understand the difference between Layer 4 and Layer 7 load balancing.

## Key Topics

- Azure Load Balancer (Layer 4 — TCP/UDP): distributes incoming connections across a pool of VMs based on a hash of source IP, source port, destination IP, destination port, and protocol
  - SKUs: Basic (free, retiring) and Standard (paid, recommended)
  - Components: Frontend IP, Backend Pool, Health Probe, Load Balancing Rule
- Application Gateway (Layer 7 — HTTP/HTTPS): routes requests based on URL path, host headers, or cookies
  - URL-based routing: send /api traffic to one backend, /images to another
  - SSL termination: decrypt HTTPS at the gateway so backend VMs handle plain HTTP
  - Web Application Firewall (WAF): block common web attacks (OWASP top 10)
- Azure Front Door: global load balancing with built-in CDN, SSL offload, and WAF — routes users to the nearest healthy backend worldwide
- Traffic Manager: DNS-based routing that directs users to the closest or healthiest endpoint across regions

## Hands-On Demo

**Account Requirements:** Standard Load Balancer is a paid resource. Instructor will demonstrate all steps — students should watch or use free trial credits to follow along.

- 💳 Create a Standard Load Balancer (paid — instructor demo)
- 💳 Add two VMs to the backend pool
- 💳 Configure a health probe (HTTP on port 80)
- 💳 Create a load balancing rule and test traffic distribution across both VMs

## Summary

Use Azure Load Balancer for simple TCP/UDP distribution across VMs. Use Application Gateway when you need HTTP-aware routing, SSL termination, or a WAF. Use Front Door for global multi-region traffic routing.
