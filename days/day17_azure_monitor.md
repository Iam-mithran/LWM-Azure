# Day 16 — Azure Monitor, Log Analytics & Alerts

> Phase 4 — Identity, Security + Monitoring

## Goal

Set up monitoring, logging, and alerting so you know what your Azure resources are doing — and get notified when something goes wrong.

## Key Topics

- Azure Monitor: the unified platform for collecting, analyzing, and acting on telemetry from all Azure resources
- Metrics vs Logs:
  - **Metrics**: numerical time-series data (e.g., CPU %, requests/sec, bytes in/out) — stored for 93 days, queried in near real-time
  - **Logs**: structured or unstructured records of events (e.g., sign-in logs, diagnostic events, app traces) — stored in Log Analytics Workspace
- Log Analytics Workspace: the central repository where logs from multiple resources are collected and queried
  - First 5GB of data ingested per day is free
  - Supports queries across multiple resources simultaneously
- Kusto Query Language (KQL): the query language for Log Analytics — SQL-like but optimized for time-series log data
  - Basic syntax: `TableName | where TimeGenerated > ago(1h) | summarize avg(CounterValue) by bin(TimeGenerated, 5m)`
- Application Insights: application performance monitoring (APM) built on Azure Monitor
  - Tracks: request rates, response times, failure rates, dependency calls, exceptions, page views
  - Free tier: 5GB/month of data ingestion
- Diagnostic Settings: the mechanism to route a resource's logs and metrics to a destination
  - Destinations: Log Analytics Workspace, Azure Storage Account, Event Hub (for streaming to external SIEMs)
- Azure Monitor Alerts:
  - Metric alerts: trigger when a metric crosses a threshold (e.g., CPU > 80%)
  - Log alerts: trigger when a KQL query returns results (e.g., error count > 10 in 5 minutes)
  - Activity log alerts: trigger on control-plane events (e.g., "someone deleted a resource group")
- Action Groups: define what happens when an alert fires — send email, SMS, call a webhook, run an Azure Function, trigger a Logic App
- Azure Dashboard: build a custom monitoring view by pinning charts, metrics, and log query results to a shared dashboard
- Azure Workbooks: interactive, parameterized reports built from metrics and log queries — shareable and version-controlled

## Hands-On Demo

**Account Requirements:** Basic metrics, 5GB/day of Log Analytics, and Application Insights free tier are all free. All steps below are free tier.

- ✅ Enable Diagnostic Settings on a VM — route logs to a Log Analytics Workspace
- ✅ Write a basic KQL query in Log Analytics (show CPU usage over the last hour)
- ✅ Create a metric alert (CPU > 80%) with an email Action Group
- ✅ Build a simple Azure Dashboard with 3–4 pinned metrics charts

## Summary

Azure Monitor is your single pane of glass for everything happening in your Azure environment. Diagnostic Settings route data to Log Analytics, KQL lets you query it, alerts tell you when thresholds are crossed, and Action Groups automate the response. Set this up before going to production — not after an incident.
