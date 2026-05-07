# Day 11 — Azure Cosmos DB — NoSQL at Global Scale

> Phase 3 — Storage + Databases

## Goal

Understand Azure's globally distributed NoSQL database and decide when to use it over a relational database.

## Key Topics

- What is Cosmos DB? A fully managed, globally distributed, multi-model NoSQL database built for low-latency, high-scale applications
- APIs supported (choose based on your workload):
  - Core (SQL): query with familiar SQL syntax — most common choice
  - MongoDB: compatible with MongoDB drivers — migrate existing apps without rewriting queries
  - Cassandra: compatible with Apache Cassandra
  - Gremlin: graph database for relationship-heavy data
  - Table: key-value storage with a Table Storage-compatible API
- Global distribution: replicate your data to any Azure region with a single click; reads are served from the nearest region for ultra-low latency
- Multi-region writes: all replicas can accept writes simultaneously — ideal for globally active apps
- Consistency levels (trade-off between consistency and latency):
  - Strong: reads always return the latest write (highest latency)
  - Bounded Staleness: reads may lag behind writes by a configured amount
  - Session: consistent within a single client session (default — good for most apps)
  - Consistent Prefix: reads never see out-of-order writes
  - Eventual: highest availability, lowest latency, reads may be stale
- Request Units (RUs): Cosmos DB's unified capacity metric — one RU = cost to read a 1KB item; all operations (reads, writes, queries) are measured in RUs
- Partitioning: Cosmos DB distributes data across physical partitions using a partition key you define — choose a key with high cardinality and even distribution
- Throughput options: Provisioned (fixed RU/s) or Serverless (pay per RU consumed)
- Cosmos DB Free Tier: 1,000 RU/s + 25GB storage — always free, one account per subscription
- Change Feed: a persistent log of every change to your data — use it to trigger downstream processing or sync to other systems

## Hands-On Demo

**Account Requirements:** Enable the Cosmos DB Free Tier during account creation (1,000 RU/s + 25GB free forever). All steps below are free tier.

- ✅ Create a Cosmos DB account with Core (SQL) API (enable Free Tier)
- ✅ Create a database and container, choose a partition key
- ✅ Insert documents and query them using the Data Explorer in the portal
- ✅ Observe RU consumption for read vs write operations

## Summary

Cosmos DB is built for applications that need single-digit millisecond latency and the ability to scale globally. The Free Tier is generous enough to learn and build small projects on. Choose your API based on your team's existing skills and driver compatibility.
