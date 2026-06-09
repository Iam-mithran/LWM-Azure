# Day 5 — Azure Functions — Serverless Computing

> Phase 1 — Compute

## Goal

Build and deploy a serverless function and understand the event-driven model.

## Key Topics

- What is serverless? Run code without provisioning or managing any servers — you pay only when your code runs
- Azure Functions triggers: what causes a function to execute
  - HTTP trigger: called via a web request
  - Timer trigger: runs on a schedule (like a cron job)
  - Blob Storage trigger: fires when a file is added to storage
  - Queue trigger: fires when a message arrives in a queue
  - Event Hub trigger: fires on streaming events
- Bindings: declarative connections to input data sources and output destinations — no manual connection code needed
- Function App vs individual Functions: a Function App is the container; you can have many Functions inside one
- Hosting plans:
  - Consumption (serverless): pay per execution, scales to zero — free tier eligible
  - Premium: pre-warmed instances, VNet support
  - Dedicated (App Service): runs on an existing App Service Plan
- Cold start: the delay when a function scales from zero — brief latency on first request after idle
- Durable Functions: stateful workflows built on top of Functions (chaining, fan-out, human interaction patterns)
- Monitoring Functions with Application Insights

## Hands-On Demo

**Account Requirements:** Consumption plan is free tier (1 million executions/month free forever). All steps below are free tier.

- ✅ Create a Function App (Consumption plan)
- ✅ Write an HTTP-triggered function in the in-portal code editor
- ✅ Test it by hitting the function URL in a browser
- ✅ View execution logs and traces in Application Insights

## Summary

Azure Functions let you run event-driven code without thinking about servers. The Consumption plan is the most beginner-friendly — free up to 1 million executions, scales automatically, and costs nothing at idle.
