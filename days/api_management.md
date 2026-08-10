# Day 35 (Optional) — Azure API Management — The API Gateway

> Bonus Day — Optional

## Who This Day Is For

This is an optional bonus session for viewers who want to publish APIs properly rather than exposing a backend URL directly. API Management is not required for AZ-900 or AZ-104, and no main-course day depends on it. Record after Day 34 (Azure Functions) — the demo fronts the Function App built there.

## Goal

Understand what an API gateway actually does, and publish a Function App through API Management with keys, rate limits, and a developer portal in front of it.

## Key Topics

- What problem an API gateway solves: one stable published URL in front of many backends, with authentication, throttling, and monitoring applied centrally instead of coded into every service
- **Where APIM sits vs. what students already learned** — this is the section that prevents the most confusion:
  - Application Gateway (Day 12): regional Layer 7 load balancer + WAF — routes HTTP, knows nothing about APIs
  - Front Door (Day 14): global Layer 7 entry point + CDN + WAF — same, but worldwide
  - API Management: API-aware — understands operations, consumers, keys, and quotas. It is not a load balancer
- Core objects and how they nest: **API** → **Operations**; **Products** bundle APIs; **Subscriptions** grant a consumer access to a Product and issue **subscription keys**
- Importing an API: from an OpenAPI/Swagger definition, from a Function App or App Service directly, or defined by hand operation by operation
- **The policy engine** — the heart of the service. XML policies applied at global / product / API / operation scope, in inbound, backend, outbound, and on-error sections:
  - `rate-limit-by-key` and `quota-by-key` — throttling per consumer
  - `validate-jwt` — reject unauthenticated calls at the gateway, before the backend is touched
  - `ip-filter` — allow/deny by address range
  - `cache-lookup` / `cache-store` — response caching at the gateway
  - `set-header`, `set-body`, `xml-to-json` — request/response transformation
  - `cors` — browser access without touching backend code
  - `mock-response` — return a fake response so front-end teams can build before the backend exists
- Backends and abstraction: the published URL stays stable while the backend moves from App Service to Functions to AKS — the consumer never knows
- Versions and revisions: run v1 and v2 side by side; test a revision privately before promoting it to live
- Developer portal: auto-generated, self-service API docs and key signup for consumers
- Named values: reusable (and secret-capable) config referenced from policies — integrates with Key Vault (Day 19)
- Self-hosted gateway: run the gateway as a container in your own datacenter or another cloud, still managed from Azure
- **Tiers and what they cost** — worth being blunt about, because this drives what students can follow:
  - Consumption: serverless, pay per call, generous monthly free call grant — the only tier students should use
  - Developer: full feature set, no SLA, ~$50/month — instructor demo only
  - Basic / Standard / Premium (and the v2 variants): production tiers, VNet integration, multi-region on Premium
- Monitoring: built-in analytics, plus Application Insights integration (Day 20)

## Hands-On Demo

**Account Requirements:** Create the APIM instance on the **Consumption** tier — provisioning is near-instant and demo-volume calls fall inside the free grant. The Developer tier is shown for the features Consumption lacks, but takes 30–45 minutes to provision, so pre-create it before recording.

- ✅ Create an API Management instance (Consumption tier)
- ✅ Import the Day 34 Function App as an API
- ✅ Call the API through the gateway URL — show the `401` when the subscription key is missing, then succeed with `Ocp-Apim-Subscription-Key`
- ✅ Create a Product, add the API to it, and issue a subscription key against it
- ✅ Apply a `rate-limit-by-key` policy, then hammer the endpoint to trigger a `429 Too Many Requests`
- ✅ Add a `cache-lookup`/`cache-store` policy and show the latency drop on the second call
- ✅ Use `set-header` to strip a backend header before it reaches the client
- ✅ Create a v2 of the API and show both versions live at once
- ✅ Browse the auto-generated developer portal
- 💳 Show VNet integration and multi-region deployment on a pre-created Developer/Premium instance
- ✅ Clean up: delete the APIM instance and resource group

## Summary

An API gateway is not a load balancer — it is the layer that knows who is calling, how often they are allowed to, and whether they are authenticated, all applied before a request ever reaches your code. API Management gives you that plus a stable public contract, so the backend can be rewritten or moved without breaking a single consumer. Start on the Consumption tier: it costs almost nothing at learning volume and covers the policies that matter most.
