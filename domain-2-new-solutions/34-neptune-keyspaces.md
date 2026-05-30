# Neptune + Keyspaces (niche purpose-built databases)

> **Two purpose-built databases for specific use cases. Amazon Neptune: managed graph database (property graph + RDF) for relationship-heavy queries (social networks, fraud detection, recommendation engines, knowledge graphs). Amazon Keyspaces: managed Apache Cassandra-compatible (CQL) for wide-column NoSQL at scale. Pick when DynamoDB doesn't fit; both rarely tested but appear as distractors or "right tool for the job" answers.**

Maps to: **Domain 2.5 — Database selection (purpose-built)**, **Domain 4.3 — Modernization**

---

# Amazon Neptune

## Overview

- **Fully managed graph database**
- Supports **Property Graph (Gremlin / openCypher)** and **RDF (SPARQL)** models
- **Multi-AZ** with up to **15 read replicas**
- **Sub-millisecond query latency** for relationship traversals
- Use cases:
  - **Social networks** (friends-of-friends, recommendations)
  - **Fraud detection** (transaction-account-device graph patterns)
  - **Recommendation engines** (collaborative filtering via graph)
  - **Knowledge graphs** (entity relationships, RAG context for GenAI)
  - **Identity management** (entitlements, role hierarchies)
  - **Network / IT operations** (topology + dependency mapping)

## Neptune Serverless

- Auto-scaling compute (NCU = Neptune Capacity Unit)
- Pay per NCU-hour consumed
- Ideal for variable / unpredictable graph workloads

## Neptune ML

- Machine learning predictions on graph data
- Embeds graph structures, runs predictions (link prediction, node classification)
- Uses SageMaker under the hood

## Neptune Analytics

- **Separate engine** for **graph analytics** (vs OLTP Neptune Database)
- In-memory graph for fast analytical queries
- Vector similarity search on graph embeddings

## Backup & DR

- **Continuous backup** to S3 (PITR within 35 days)
- **Cross-Region snapshot copy**
- **Multi-AZ writer + readers**
- **Global Database** for cross-Region read replicas (single primary)

## Security

- **VPC-only** deployment
- **KMS encryption at rest** + TLS in transit
- **IAM authentication** for graph API requests

---

# Amazon Keyspaces (for Apache Cassandra)

## Overview

- **Fully managed Cassandra-compatible** database
- **CQL-compatible** — use existing Cassandra drivers + tooling
- **Serverless** — no nodes / clusters to manage
- **Single-digit-ms read/write latency** at any scale
- **Multi-AZ** by default with auto-scaling
- Use cases:
  - **Wide-column at scale** (IoT telemetry, time-series, fleet management)
  - **Migrate on-prem Cassandra to AWS** without re-engineering drivers
  - **Cassandra-API workloads** when you don't want to operate self-managed Cassandra on EC2

## Capacity Modes

- **On-Demand** — pay per request; auto-scale
- **Provisioned** — set RCU / WCU per second

## Point-in-Time Recovery

- Restore to any second within last 35 days

## Multi-Region Replication

- Multi-Region, multi-active **Global Tables** for Keyspaces
- Active-active reads + writes across Regions

## Security

- **VPC endpoints** (PrivateLink)
- **KMS encryption at rest**
- **IAM-based authentication** OR Cassandra-native auth
- **TLS** in transit

---

# Neptune vs Keyspaces vs Other AWS DBs

| Need | Service |
|---|---|
| Graph queries (friend-of-friend, fraud rings) | **Neptune** |
| Cassandra API compatibility / wide-column at scale | **Keyspaces** |
| Key-value / document NoSQL | **DynamoDB** |
| Time-series at extreme scale | **Timestream** |
| Document / MongoDB compatibility | **DocumentDB** |
| Multi-Region NoSQL | **DynamoDB Global Tables** OR Keyspaces Multi-Region |
| Knowledge graph for GenAI RAG | **Neptune** (or OpenSearch vector search) |

---

## Exam Tips

- "Highly connected data / relationship traversal" → **Neptune**
- "Social network / fraud detection / recommendation engine" → **Neptune**
- "Lift-and-shift Cassandra workload to managed AWS" → **Amazon Keyspaces**
- "Cassandra CQL at scale, no cluster ops" → **Keyspaces**
- **Neptune Serverless** for variable graph workloads
- **Neptune Analytics** for in-memory graph + vector similarity
- **Keyspaces Multi-Region** for active-active
- **Keyspaces serverless on-demand** for unpredictable workloads
- Both: **KMS encryption, VPC-only deployment, IAM auth**

## Exam Traps

- **Neptune ≠ DynamoDB** — Neptune is graph; DynamoDB is key-value / document
- **Neptune is NOT real-time analytics** — for OLAP / dashboards use Athena / Redshift
- **Keyspaces is NOT a full Cassandra cluster substitute** for all features — check compatibility (some Cassandra extensions unsupported)
- **Neptune Global Database is single-writer** — for active-active multi-Region writes on relationships, look at app-level patterns
- **Keyspaces supports Cassandra 3.11 + 4.x CQL** — newer Cassandra-specific features may not be supported
- **QLDB (Quantum Ledger Database) is being discontinued** (Jul 2025) — never the right answer
