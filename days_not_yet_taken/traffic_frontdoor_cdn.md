# Day 12 — Traffic Manager, Azure Front Door & Azure CDN

> Phase 3 — Storage, Databases & Global Delivery

## Goal

Route users to the closest, healthiest endpoint worldwide and deliver content at the edge using Azure's global delivery services.

## Key Topics

### Azure Traffic Manager

- **What is Traffic Manager?** A DNS-based global load balancer — it does not sit in the data path; instead it returns a DNS answer that directs users to the best endpoint
- **How it works:** User queries DNS → Traffic Manager evaluates all endpoints against the active routing method → returns the IP of the best endpoint → user connects directly to that endpoint
- **Routing methods:**
  - **Performance**: routes to the endpoint with the lowest network latency from the user's location — the most common choice
  - **Priority**: sends all traffic to a primary endpoint; failover to secondary only if primary is unhealthy
  - **Weighted**: distributes traffic in a ratio you define (e.g., 80/20 between two regions) — useful for gradual rollouts
  - **Geographic**: routes users to endpoints based on their country or continent — required for data residency compliance
  - **Subnet**: routes specific IP ranges to specific endpoints (e.g., corporate IP range → internal backend)
  - **MultiValue**: returns multiple healthy endpoint IPs in a single DNS response — client picks one
- **Endpoints:** Azure services (VMs, App Service, Cloud Services), external endpoints (any public IP or FQDN), nested Traffic Manager profiles
- **Health probes:** Traffic Manager polls endpoints on a configurable interval; unhealthy endpoints are automatically removed from DNS responses
- **TTL:** Traffic Manager responses have a short DNS TTL (30–300 seconds) — failover takes effect quickly after a probe failure is detected
- **Use cases:** multi-region active-active apps, blue-green deployments, disaster recovery failover, geographic compliance

### Azure Front Door

- **What is Front Door?** Microsoft's global application delivery network (ADN) — it combines global load balancing, CDN, SSL termination, WAF, and DDoS protection in one service
- **How it differs from Traffic Manager:** Front Door sits in the data path (traffic flows through Microsoft's network edge), while Traffic Manager is DNS-only (traffic flows directly to your backend)
- **Key features:**
  - **Anycast routing:** user requests are received at the nearest of Microsoft's 200+ edge PoPs (Points of Presence) worldwide — reduces latency significantly
  - **Caching:** static content (images, JS, CSS) is cached at the edge — backends only receive requests for dynamic content
  - **SSL offload:** certificates are managed and terminated at the edge
  - **WAF integration:** same WAF rule sets as Application Gateway, enforced at the global edge
  - **Session affinity:** route the same user to the same backend origin
  - **Health probes + failover:** routes around unhealthy backends automatically
  - **URL rewrite and redirect:** rewrite paths and redirect HTTP → HTTPS at the edge
- **Origins:** App Service, Storage, VMs, Application Gateway, custom endpoints
- **Front Door vs Application Gateway:** use Application Gateway for traffic within a region; use Front Door for global multi-region delivery
- **Front Door vs Traffic Manager:** Front Door is active-in-path (caches, inspects, transforms); Traffic Manager is DNS-only (pure routing)

### Azure CDN

- **What is CDN?** A Content Delivery Network — a global network of edge servers that cache your static content close to users, reducing latency and origin server load
- **Azure CDN profiles:** choose a provider:
  - **Azure CDN from Microsoft (built into Front Door Standard/Premium)** — recommended for new deployments
  - **Azure CDN from Edgio (formerly Verizon)** — legacy profiles, being retired
- **How CDN works:** user requests `cdn.example.com/logo.png` → nearest CDN edge checks its cache → if cached (cache hit): serve immediately from edge → if not cached (cache miss): fetch from origin, cache it, serve to user
- **Origin types:** Azure Blob Storage (most common), Web Apps, VMs, any public endpoint
- **Cache rules:** configure TTL per content type, path, or query string — images cache for 30 days, API responses cache for 60 seconds
- **Purge:** force the CDN to evict cached content immediately — useful after a deployment that changes static assets
- **Custom domains + HTTPS:** map your own domain (e.g., `assets.example.com`) to the CDN endpoint and provision a free managed SSL certificate
- **CDN use cases:** serving website assets (images, JS, CSS, fonts), software download distribution, video streaming, API response caching

## Hands-On Demo

**Account Requirements:** Traffic Manager has a small charge per DNS query + per month per endpoint. Front Door Standard has a minimum monthly commitment. Azure CDN charges per GB transferred + per request. Instructor demo — students should watch or use free trial credits.

- 💳 Create a Traffic Manager profile with Performance routing; add two endpoints (App Service apps or VMs in two different regions); simulate a failover by disabling one endpoint and confirming DNS switches
- 💳 Create an Azure Front Door profile (Standard tier); add a backend origin (App Service); test global routing and edge caching by checking response headers for `X-Cache`
- ✅ Create an Azure CDN endpoint backed by a Blob Storage container (static files); upload an image; access it via the CDN URL; purge the cache and re-access to observe a cache miss then cache hit

## Summary

Traffic Manager routes at the DNS level — zero latency overhead, pure global failover and distribution. Front Door routes at the application level — it sits in the path and adds caching, WAF, and SSL at Microsoft's global edge. Azure CDN caches static content at hundreds of edge locations worldwide. Together, these three services give you the tools to build globally resilient, low-latency applications at any scale.
