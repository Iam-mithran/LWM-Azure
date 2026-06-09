# Day 11 — Azure Application Gateway & Web Application Firewall (WAF)

> Phase 3 — Storage, Databases & Global Delivery

## Goal

Understand Layer 7 HTTP/HTTPS load balancing, intelligent traffic routing, SSL termination, and protecting web apps from common attacks using WAF.

## Key Topics

- **What is Application Gateway?** A fully managed Layer 7 (HTTP/HTTPS) load balancer — unlike Azure Load Balancer which distributes at TCP level, Application Gateway understands the content of HTTP requests and routes based on URLs, hostnames, and headers
- **Key features:**
  - **URL path-based routing**: send `/api/*` to one backend pool, `/images/*` to another, `/` to a third — all through a single public IP
  - **Multi-site hosting**: host multiple websites on one Application Gateway using host header routing (e.g., `app1.example.com` → Backend Pool A, `app2.example.com` → Backend Pool B)
  - **SSL/TLS termination**: decrypt HTTPS at the gateway; backend VMs receive plain HTTP — reduces CPU load on backends and centralizes certificate management
  - **End-to-end SSL**: re-encrypt traffic between the gateway and backends for maximum security
  - **Session affinity (cookie-based)**: route the same user to the same backend VM across multiple requests — important for stateful apps
  - **Health probes**: custom HTTP probes check backend health; unhealthy backends are automatically removed from rotation
  - **Autoscaling**: Application Gateway v2 automatically scales its own capacity based on traffic — no manual sizing
  - **Redirection**: automatically redirect HTTP → HTTPS, or redirect one URL to another
  - **Rewrite headers**: modify HTTP request/response headers before forwarding — add CORS headers, strip internal headers, inject tracking values
- **SKUs:** Standard_v2 (load balancing only) and WAF_v2 (includes Web Application Firewall) — v2 SKU recommended for all new deployments
- **Backend pool types:** VMs, VM Scale Sets, App Service (Web Apps), IP addresses, FQDNs — any mix in the same pool
- **Listeners:** define how the gateway accepts traffic — Basic listener (single site) or Multi-site listener (multiple domains on the same IP/port)
- **Routing rules:** bind a listener to a backend pool (Basic rule) or to a path map (Path-based rule)

### Web Application Firewall (WAF)

- **What is WAF?** A managed rule set that inspects every HTTP request for known attack patterns before forwarding it to your backend
- **OWASP Core Rule Set (CRS):** WAF ships with rules covering OWASP Top 10 attack categories:
  - SQL injection, Cross-Site Scripting (XSS), Remote Code Execution, Directory Traversal, Local File Inclusion, HTTP protocol violations, and more
- **WAF modes:**
  - Detection mode: logs suspicious requests but does not block them — use this first to understand what would be blocked before enforcing
  - Prevention mode: actively blocks requests that match attack signatures — use in production
- **Custom rules:** write your own rules to block specific IPs, countries, request patterns, or rate-limit clients
- **WAF policy:** a standalone resource that defines the rule set, custom rules, and exclusions — attach the same WAF policy to multiple Application Gateways
- **Bot protection:** optional managed rule set that blocks known malicious bots (scrapers, scanners, exploit tools) while allowing legitimate bots (Googlebot, Bingbot)

## Hands-On Demo

**Account Requirements:** Application Gateway v2 is a paid resource (charged per gateway hour + capacity units consumed). Instructor demo — students should watch or use trial credits.

- 💳 Create an Application Gateway v2 (WAF_v2 SKU) with a public frontend IP
- 💳 Create two backend pools (pointing to two different VMs, each running Nginx with different content)
- 💳 Configure a path-based routing rule: `/app1/*` → Backend Pool 1, `/app2/*` → Backend Pool 2
- 💳 Test path-based routing by visiting `http://GATEWAY_IP/app1/` and `http://GATEWAY_IP/app2/` in a browser
- 💳 Enable WAF in Detection mode and review the WAF logs for any flagged requests
- 💳 Switch WAF to Prevention mode and attempt a simple SQL injection in the URL — confirm it is blocked

## Summary

Application Gateway is the right tool whenever you need HTTP-aware routing, multi-site hosting, SSL termination, or application-layer security. WAF adds a managed protection layer that blocks OWASP Top 10 attacks without writing any firewall rules yourself. For internet-facing web apps, Application Gateway + WAF is the standard production pattern in Azure.
