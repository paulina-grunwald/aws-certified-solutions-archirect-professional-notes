# Amazon DocumentDB (with MongoDB Compatibility)

> **Fully managed, MongoDB-compatible document database with separated compute + storage (similar to Aurora architecture). Stores JSON documents (BSON), supports MongoDB drivers + query API (compatibility with MongoDB 3.6 / 4.0 / 5.0 APIs). Cluster has 1 primary writer + up to 15 replicas. Storage auto-scales up to 128 TB. Use when migrating off MongoDB on-prem / Atlas to AWS-managed offering. Common SAP-C02 distractor against DynamoDB (NoSQL key-value) or MongoDB Atlas (third-party).**

Maps to: **Domain 2.5 — High-performance architectures**, **Domain 4.2 — Migration**

---

## Overview

- **MongoDB-compatible** document database — JSON / BSON documents
- **Decoupled compute + storage** (like Aurora) — storage layer scales automatically
- **Cluster volume** replicated 6× across 3 AZs
- **Compute**: 1 primary + up to **15 read replicas**
- **Endpoints**: cluster endpoint (writer), reader endpoint (round-robin replicas), instance endpoints
- API compatibility: **MongoDB 3.6, 4.0, 5.0** (not 100% — some operators missing)

## Architecture

- **Storage layer**: distributed, log-structured, auto-scales 10 GB → 128 TB
- **Compute layer**: independent of storage; resize / add replicas without downtime
- **6 replicas across 3 AZs**, quorum-based reads/writes
- **Continuous backup** to S3 (point-in-time recovery up to 35 days)

## Use Cases

- Migrate **existing MongoDB** workloads to AWS without rewriting app
- Document-style data with **JSON-native queries**
- Catalog, user profile, content management, mobile app backends
- When you need **MongoDB drivers** but want AWS-managed

## DocumentDB vs MongoDB Atlas

| | DocumentDB | MongoDB Atlas on AWS |
|---|---|---|
| Managed by | AWS | MongoDB Inc. |
| API compatibility | Partial (3.6 / 4.0 / 5.0) | 100% (native) |
| AWS service integration | Tight (IAM, VPC, KMS, CloudWatch) | Via PrivateLink |
| Storage scaling | Auto (128 TB) | Auto (configured) |
| Pricing | AWS billing | MongoDB billing |
| When to pick | AWS-native, simpler MongoDB workloads | Latest features, hybrid cloud |

## DocumentDB vs DynamoDB

| | DocumentDB | DynamoDB |
|---|---|---|
| Model | Document (JSON) | Key-value + document |
| Query | MongoDB query language (rich) | PartiQL, key/index queries (limited) |
| Joins | No (document model) | No |
| Schema | Flexible | Flexible (key + attributes) |
| Scaling | Vertical (writer) + read replicas | Horizontal (partitions) |
| Multi-Region | Global Clusters (read-only secondary regions) | Global Tables (multi-master) |
| Best for | MongoDB lift-and-shift, complex queries | Massive scale, simple access patterns |
| Cost | Instance-hour | Read/write capacity or on-demand |

**Rule of thumb**: existing MongoDB workload? → **DocumentDB**. New design, massive scale, simple access? → **DynamoDB**.

## Performance Features

- **Up to 15 read replicas** with sub-100ms replica lag
- **Reader endpoint** load-balances reads
- **Auto-scaling** for read replicas (CPU / connections target)
- **Storage auto-scaling** in 10 GB increments
- **Sharding NOT supported natively** — vertical writer is the constraint

## Global Clusters

- **Multi-Region** read-only secondaries
- 1 primary Region + up to 5 secondary Regions
- < 1s replication lag
- Promote secondary in DR
- Read scale-out across continents

## Backup & Restore

- **Continuous backup** to S3 (automatic, no perf impact)
- **Point-in-time recovery** to any second within retention window (1–35 days)
- **Snapshots** for cross-Region copy / cross-account share
- **AWS Backup integration**

## Security

- **VPC-only** (no public endpoint)
- **TLS in transit** (mandatory, can disable but don't)
- **KMS encryption at rest** (mandatory)
- **IAM auth** is NOT supported (unlike Aurora) — use username/password (in Secrets Manager)
- **Secrets Manager rotation** for password rotation
- **Cluster security group** controls network access

## Limitations vs Native MongoDB

- Not 100% API-compatible — some operators / commands missing
- **No native sharding** — vertical scale only
- **No `$where`** + **no MapReduce**
- **No change streams cross-cluster** (intra-cluster only)
- Limited support for Atlas-only features (Realm, Charts, Search)

Check **AWS docs for compatibility version** before assuming a feature works.

## Common Patterns

### Lift-and-shift MongoDB
- Use **DMS** to migrate data + change capture from on-prem MongoDB
- Cutover to DocumentDB endpoint
- App keeps using MongoDB driver — minimal code change

### Multi-Region read scale
- Primary cluster in us-east-1
- Global Cluster secondaries in eu-west-1, ap-southeast-1
- Apps in each Region read locally
- Writes go back to us-east-1

### High-availability ops DB
- Primary + 2 replicas across 3 AZs
- Multi-AZ failover < 30s
- Application uses **cluster endpoint** (auto-failover)

## Pricing

- Per-instance hour (compute) — sized like RDS instances
- Storage: $0.10 per GB-month
- I/O: $0.20 per million requests
- Backup storage: $0.021 per GB-month (free for backup ≤ DB size)
- Reserved Instances available for compute

## Exam Tips

- "Managed MongoDB-compatible database on AWS" → **DocumentDB**
- "Migrate MongoDB workload to managed AWS DB" → **DocumentDB via DMS**
- Up to **15 read replicas**, multi-AZ, 6× storage replication
- **Global Clusters** for multi-Region read scale + DR
- API compatibility: **MongoDB 3.6 / 4.0 / 5.0** (partial)
- **TLS + KMS mandatory**
- No native IAM auth — use Secrets Manager

## Exam Traps

- **DocumentDB is NOT 100% MongoDB-compatible** — test queries before assuming portability
- **No sharding** — vertical writer is the scaling ceiling (~768 GB memory on largest instance)
- **No IAM database auth** — unlike Aurora, must use username/password
- **DocumentDB ≠ DynamoDB** — totally different services despite both being "NoSQL"
- **Global Cluster secondaries are read-only** — single writer Region
- **Storage auto-scales in 10 GB increments** — no need to pre-provision
- **TLS is on by default; disabling not recommended** but possible
- **MongoDB Atlas is a separate product** (third-party SaaS) — not DocumentDB
- **No native change streams to other AWS services** — use DMS or Lambda polling for CDC patterns
