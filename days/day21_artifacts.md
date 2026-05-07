# Day 21 — Azure Artifacts & Pipeline Best Practices

> Phase 5 — Azure DevOps

## Goal

Manage internal package dependencies with Azure Artifacts and wrap up Azure DevOps with reusable pipeline patterns.

## Key Topics

- Azure Artifacts: a private package registry built into Azure DevOps — publish and consume internal packages without exposing them to the public internet
- Supported package types: npm (Node.js), NuGet (.NET), Maven (Java), Python (PyPI), Universal Packages (any file format)
- Feeds: the container for your packages — you can have multiple feeds per project or organization
- Upstream sources: configure a feed to proxy requests to public registries (npmjs.com, nuget.org, PyPI) — all dependency traffic flows through your feed, giving you a bill of materials and the ability to block unwanted packages
- Publishing packages: build your library in a pipeline, then use a publish task to push the versioned package to your feed
- Pipeline caching: cache your `node_modules` or Maven `.m2` folder between runs — can cut build times significantly without changing your code
- Pipeline templates: define reusable YAML fragments in a separate file and reference them across multiple pipelines — the DRY (Don't Repeat Yourself) principle for pipelines
  - Step templates: reuse a set of steps
  - Job templates: reuse an entire job
  - Stage templates: reuse an entire stage (e.g., a standard deploy stage every team uses)
- Service connections: securely store credentials that pipelines use to connect to external systems (Azure subscription, Docker Hub, GitHub, npm registry) — credentials are stored encrypted and never visible in pipeline logs
- Azure DevOps security: control who can trigger pipelines, approve deployments, and access environments via project-level and organization-level permissions

## Hands-On Demo

**Account Requirements:** Azure Artifacts free tier includes 2GB of package storage. All steps below are free tier.

- ✅ Create an Azure Artifacts feed
- ✅ Publish an npm package to the feed from a pipeline (using `npm publish` with the feed URL)
- ✅ Reference the internal feed in a downstream project (`npm install` pulls from your private feed)
- ✅ Create a reusable step template YAML file and reference it from two different pipelines

## Summary

Azure Artifacts gives your team a private, controlled registry for internal packages — no more copying shared code between repos. Pipeline templates and caching are the two biggest levers for keeping pipelines fast and maintainable as your team grows.
