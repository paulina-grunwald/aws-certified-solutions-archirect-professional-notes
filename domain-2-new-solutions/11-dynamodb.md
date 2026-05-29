# Amazon DynamoDB

> **Fully managed, serverless, key-value + document NoSQL DB. Single-digit-millisecond reads at any scale. Global Tables for multi-Region active-active. MRSC for strong consistency across 3 Regions. DAX for microsecond reads. Streams + Lambda for event-driven patterns. Zero-ETL to Redshift for near-real-time analytics. Multi-account replication.**

Maps to: **Domain 2.5 — Database selection**, **Domain 2.4 — Decoupling / event-driven**, **Domain 1.3 — Reliability**, **Domain 4.3 — Modernization**

---

## Overview

- Fully managed, serverless NoSQL
- **Single-digit-millisecond** performance at any scale
- Built-in security, backup/restore, in-memory caching (DAX)
- **No servers to provision**, automatic throughput + storage scaling
- Key-value AND document data models
- **ACID transactions** via `TransactWriteItems`, `TransactGetItems`
- **Max item size: 400 KB**
- Read consistency: **eventually consistent** (default), **strongly consistent**, **transactional**
- **SLA**: 99.999% for Global Tables, 99.99% for standard tables

---

## Core Concepts

### Tables, Items, Attributes

- Table → collection of items; item → collection of attributes
- **Primary key** uniquely identifies each item:
  - **Partition Key (PK) only**, or
  - **Partition Key + Sort Key (PK + SK)** composite
- **No fixed schema beyond the primary key** — each item can have different attributes

### Data Types

- **Scalar**: String, Number, Binary, Boolean, Null
- **Document**: List, Map
- **Set**: String Set, Number Set, Binary Set

---

## Capacity Modes

### On-Demand

- Pay per request — no capacity planning
- Instant accommodation of traffic ramps
- Best for: unpredictable workloads, new tables, spiky / bursty traffic
- Switch between provisioned and on-demand up to **4 times per rolling 24-hour period** (previously 1×/day)

### Provisioned

- You specify RCU / WCU per second
- **1 RCU** = 1 strongly consistent read/sec (up to 4 KB) OR 2 eventually consistent reads/sec
- **1 WCU** = 1 write/sec (up to 1 KB)
- Transactional reads/writes cost **2×**
- **Auto Scaling** with target tracking
- Best for: predictable workloads, cost optimization
- Free tier: 25 RCU + 25 WCU

### Auto Scaling Considerations

- Application Auto Scaling with **target tracking** (e.g., 70% utilization)
- Reacts to sustained changes — has cooldown
- Works on tables AND GSIs independently
- **Does NOT protect against sudden spikes** — use on-demand or over-provision

---

## Partition Key Design

- Data distributed across partitions by hash of partition key
- **Hot partition** = disproportionate traffic to a single key value
- **Best practices**:
  - High-cardinality attributes (userID, orderID)
  - Avoid time-based partition keys (date) that concentrate writes
  - **Write sharding** — append random suffix or calculated suffix to distribute writes
  - **Adaptive Capacity** auto-isolates frequently accessed items, but good key design is still essential
- **Burst capacity** — DynamoDB retains up to **5 minutes** of unused read/write capacity per partition for bursts

---

## Secondary Indexes

### Global Secondary Index (GSI)

- Alternate partition key + optional sort key (different from base table)
- **Own provisioned throughput** (RCU/WCU) — separate from base
- **Eventually consistent reads only**
- Can be **created or deleted any time**
- Max **20 GSIs per table**
- **GSI back-pressure**: if a GSI throttles, base table writes also throttle
- Attribute projections: `KEYS_ONLY`, `INCLUDE`, `ALL`
- **Sparse index pattern** — only items with the GSI key attribute are projected; useful for filtering

### Local Secondary Index (LSI)

- Same partition key as base; different sort key
- **Must be created at table creation** — cannot be added later
- **Supports strongly consistent reads**
- Max **5 LSIs per table**
- **Shares throughput** with base table
- **10 GB size limit per partition key value** (base + all LSIs combined)

| Need | Use |
|---|---|
| Different partition key for queries | **GSI** |
| Same partition key, different sort key, strong consistency | **LSI** (only if planned at table creation) |

---

## DynamoDB Streams

- Time-ordered sequence of item-level changes (insert/update/delete)
- **24-hour retention**
- View types: `KEYS_ONLY`, `NEW_IMAGE`, `OLD_IMAGE`, `NEW_AND_OLD_IMAGES`
- Common pattern: **Streams + Lambda triggers** for event-driven architectures
- Use cases: replication, materialized views, analytics, audit logs, cross-Region
- Each record appears **exactly once and in order per partition key**
- **Kinesis Adapter** — consume via KCL
- **Kinesis Data Streams for DynamoDB** alternative — higher throughput, up to 1 year retention, fan-out, Firehose/Analytics integration

---

## DynamoDB Accelerator (DAX)

- Fully managed **in-memory cache** for DynamoDB — **microsecond** read latency
- **Write-through** cache (writes go to DynamoDB first, then cached)
- API-compatible with DynamoDB — minimal code changes
- Caches: **item cache** (GetItem, BatchGetItem) + **query cache** (Query, Scan)
- Runs inside your VPC, not publicly accessible
- **Not suitable for**: write-heavy workloads, strongly consistent reads
- Cluster: 1–11 nodes; multi-AZ for HA
- Default TTL: 5 min (item cache), 1 min (query cache)
- DAX supports **AWS PrivateLink** for management APIs over private IPs

**DAX vs ElastiCache**: DAX is purpose-built for DynamoDB, no app code changes. ElastiCache is general-purpose and needs app-level cache logic.

---

## Global Tables

- Multi-Region, **multi-active** (read/write in any Region) replication
- Uses DynamoDB Streams under the hood
- **Eventual consistency** between replicas (typically sub-second)
- Conflict resolution: **last-writer-wins** by timestamp
- All replicas must share table name, key schema, and have Streams enabled
- Requires **on-demand** or auto-scaled provisioned capacity on all replicas
- **SLA: 99.999%**

### Multi-Region Strong Consistency (MRSC)

- **Strongly consistent reads from ANY Region** — **RPO of zero**
- Writes synchronously replicate to ≥ 1 other Region before returning success
- **Exactly 3 Regions required**:
  - 3 full replicas, OR
  - 2 full replicas + 1 **witness** (stores data, no read/write traffic, lower cost)
- Available Regions: us-east-1, us-east-2, us-west-2, eu-west-1, eu-west-2, eu-west-3, eu-central-1, ap-northeast-1, ap-northeast-2, ap-northeast-3
- **Higher write latency** due to synchronous cross-Region replication
- MRSC global tables support resiliency testing with **AWS FIS**

### Multi-Account Replication

- Global Tables support replication **across multiple AWS accounts**
- Enables distinct security / governance per account
- Improves organizational resiliency by distributing replicas across accounts

---

## TTL (Time to Live)

- Auto-deletes expired items at **no extra cost** (no WCU consumed)
- TTL attribute must be **Number** type storing a **Unix epoch (seconds)**
- Deletion within **~48 hours** of expiration (not instant)
- **Expired items still appear in reads until actually deleted** — filter in application logic
- TTL deletions appear in **DynamoDB Streams** (useful for archiving to S3 via Lambda)
- Use cases: session management, log expiration, compliance retention

---

## Backup & Restore

### On-Demand Backup

- Full table backup at any time, no performance impact
- Retained until explicitly deleted
- **Restores to a NEW table only** (cannot overwrite)

### Point-in-Time Recovery (PITR)

- Continuous backups; restore to any **second in the last 35 days**
- **Explicit opt-in per table**
- Restores to a new table; source unchanged
- Works with encrypted tables and Global Tables

---

## Export to S3

- Export full table data to S3 (DynamoDB JSON or Amazon Ion)
- **Requires PITR enabled**
- **Does NOT consume RCUs** — reads directly from backup
- Snapshot at any point within PITR window
- **Incremental export** — only changes since last export
- Use with Athena, Glue, EMR for analytics

---

## Zero-ETL Integration with Redshift

- Fully managed, **near-real-time** replication from DynamoDB → Redshift
- No coding, no ETL pipelines
- Updates replicated every **15–30 minutes** via incremental exports
- **Does NOT consume DynamoDB RCUs**, minimal production impact
- Multiple DynamoDB tables → single Redshift cluster or serverless workgroup
- Expanded regions (Thailand, Malaysia, Mexico, Taipei)
- Alternative to manual DynamoDB → Kinesis → Firehose → Redshift pipelines

---

## Security

- **Encryption at rest** by default (AWS-owned, AWS-managed, or customer-managed KMS key)
- **Encryption in transit** via TLS
- **Fine-grained access control** via IAM — restrict to items and attributes with `dynamodb:LeadingKeys`, `dynamodb:Attributes`
- **VPC endpoints (Gateway type)** for private access without internet
- **Resource Control Policies (RCPs)** support DynamoDB — centralized max-permission control across Organizations
- **CloudTrail** logs DynamoDB API calls (control plane; data plane logging adds cost)

---

## Integration Patterns

- **Lambda triggers** — Streams + Lambda for event-driven processing
- **AWS Glue** — DynamoDB connector with native Spark DataFrame support for ETL to data lake
- **API Gateway + Lambda + DynamoDB** — classic serverless API
- **AppSync** — direct DynamoDB resolver for GraphQL
- **EventBridge Pipes** — Streams as source, with filter / enrich / transform
- **S3 Import** — import CSV / DynamoDB JSON / Ion into a NEW table
- **Kinesis Data Streams for DynamoDB** — alternative to Streams (longer retention, broader integrations)

---

## Exam Tips

- "Microsecond read latency" → **DAX**
- "Multi-Region active-active" → **Global Tables**
- "Strongly consistent reads across Regions" → **MRSC Global Tables (3 Regions required)**
- "No capacity planning" / "unpredictable traffic" → **on-demand mode**
- "Analytics on DynamoDB data without ETL" → **zero-ETL with Redshift**
- "Auto-expire data" → **TTL** (deletes are free, ~48-hour delay)
- "Audit trail of changes" / "trigger Lambda on change" → **DynamoDB Streams**
- "Export without affecting production reads" → **Export to S3** (uses PITR, no RCU cost)
- **GSI throttling causes base table throttling** (back-pressure) — common architecture pitfall
- "Different partition key query" → **GSI**; "Same PK, different SK, strong consistency" → **LSI** (only if planned at table creation)
- **DAX vs ElastiCache** — DAX = drop-in for DynamoDB, no code changes; ElastiCache = general-purpose, app-level logic
- "Cross-account Global Tables replication" → supported

---

## Exam Traps

- **DAX does NOT support strongly consistent reads** — if a question requires strong consistency, DAX is wrong
- **LSI cannot be added after table creation** — "add index to existing table" → only GSI works
- **LSI has a 10 GB per partition key limit** — exceeding it causes writes to fail
- **TTL deletes are NOT instant** (up to 48 hours) — don't rely on TTL for real-time removal; filter expired items in queries
- **Auto Scaling does NOT handle sudden spikes** — reacts to sustained changes with cooldowns
- **PITR restores to a NEW table** — cannot restore in-place
- **Global Tables require DynamoDB Streams enabled** — if Streams aren't enabled, Global Tables won't work
- **MRSC Global Tables require exactly 3 Regions** — 2 Regions is NOT enough for multi-Region strong consistency
- **On-demand mode still has throttling** — initial throughput + doubling behavior; not truly unlimited
- **Export to S3 requires PITR** — without PITR, you cannot export
- **Zero-ETL → Redshift has 15–30 minute latency** — NOT real-time; do not select it when sub-second analytics latency is required
- **Conflict resolution in Global Tables = last-writer-wins** by timestamp — strong consistency is NOT default; use MRSC if needed
- **DynamoDB max item size is 400 KB** — store large blobs in S3 and keep a reference in DynamoDB
