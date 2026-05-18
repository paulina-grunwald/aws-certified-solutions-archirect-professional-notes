# Amazon Redshift

> **Fully managed, petabyte-scale cloud data warehouse. Columnar storage + MPP for fast OLAP / BI queries. PostgreSQL-compatible. Available as provisioned clusters or Redshift Serverless.**

Maps to: **Domain 2.5 — Design high-performance architectures** (data warehouse)

---

## Overview

- **Columnar store** (vs row-based) — drastically reduces disk I/O for analytics queries
- **MPP (Massively Parallel Processing)** — distributes data and queries across compute nodes
- **OLAP** workloads (BI, dashboards, ad-hoc analytics) — NOT for OLTP (use RDS / Aurora for that)
- **Petabyte scale** — independent scaling of compute and storage (with RA3 / Serverless)
- **PostgreSQL-compatible** for most BI tools out of the box (Tableau, Power BI, QuickSight, Looker)
- Load data from S3, EMR, DynamoDB, Kinesis, MSK, RDS, Aurora — many ingest paths

---

## Provisioned vs Redshift Serverless

### Provisioned Clusters

- You choose node type and node count
- Predictable workloads where you control capacity
- Pay per node-hour
- **Cluster relocation** supported on RA3 — move cluster to a different AZ

### Redshift Serverless

- **No nodes to manage** — capacity scales automatically
- Pay per second for **RPU** (Redshift Processing Unit) consumed
- **AI-driven scaling**: automatically adjusts based on data volume, concurrent users, query complexity
- Best for intermittent / unpredictable workloads (dev / test, sporadic analytics)
- Same SQL surface as provisioned; same data sharing / federated query / Spectrum / streaming ingestion

> 💡 **Exam pick**: predictable steady-state workload → provisioned. Spiky / unpredictable / no DBA → **Redshift Serverless**.

---

## Node Types (Provisioned Clusters)

### RA3 (current generation, recommended)

- **Separates compute from storage** — managed storage in S3 transparent to you
- Pay for **compute by node hour** + **storage by GB/month** (separate)
- Sizes: `ra3.xlplus`, `ra3.4xlarge`, `ra3.16xlarge`
- Required for: **data sharing, zero-ETL, Multi-AZ, AQUA**

## Multi-AZ Deployment

- Redshift supports **Multi-AZ on RA3 clusters** for high availability
- **Synchronous replication** to a standby in another AZ
- **Automatic failover** in case of AZ failure — no manual intervention
- Both AZs are **active** — read queries can run on either (active-active for reads)
- Multi-AZ also supported with **zero-ETL integration**

> 💡 **Exam pattern**: scenario requires "highly available data warehouse with automatic failover" → **Redshift Multi-AZ on RA3**.

---

## Architecture — Leader + Compute Nodes

### Leader node

- Parses queries, builds execution plans, distributes compiled code to compute nodes
- Aggregates results from compute nodes
- Free of charge

### Compute nodes

- Execute query plan steps in parallel (MPP)
- Each compute node is divided into **slices** (typically 2–16 per node depending on type)
- Each slice processes a portion of the data

### Massively Parallel Processing (MPP)

- Automatically distributes data and queries across all slices
- Adding nodes increases parallelism

---

## Distribution Keys, Sort Keys, Compression

### Distribution styles (how rows are spread across slices)

- **KEY** — rows with the same value in the distribution column go to the same slice. Best for joins on that column.
- **ALL** — small dimension tables; replicated to every node. Best for star schemas.
- **EVEN** — round-robin distribution. Default; reasonable for many cases.
- **AUTO** (default for new tables) — Redshift picks based on table size and access patterns

### Sort keys (physical row order on disk)

- **Compound sort key** — multiple columns, prioritized left-to-right
- **Interleaved sort key** — equal weighting for multiple columns
- **AUTO** (recommended) — Redshift picks based on workload
- Sort keys enable **zone maps** that skip blocks of data not matching predicates

### Compression encodings

- Columnar data is highly compressible
- Redshift **automatically samples** new data on first COPY into an empty table and picks the best encoding
- Encodings: AZ64 (general default), ZSTD (large strings), Delta, Mostly-N, Runlength, Text255 / Text32k, etc.
- Compression reduces disk I/O = faster queries

---

## Concurrency Scaling

- Automatically adds **transient cluster capacity** to handle bursts of concurrent read queries
- Free for 1 hour per day of credit per cluster; pay per second beyond that
- Eligible queries: read-only on tables with appropriate properties
- Reduces queueing during peak periods (dashboards, end-of-quarter reporting)

**Exam pattern**: "many concurrent BI users overwhelming the cluster" → enable **Concurrency Scaling**.

---

## AQUA (Advanced Query Accelerator)

- Hardware-accelerated cache for **RA3 clusters** (built into the cluster)
- Uses **custom FPGAs** to speed up large filters and aggregations directly on compressed columnar data near the storage
- Transparent — no query changes
- Best for: scan-heavy queries with LIKE / SIMILAR_TO predicates on large datasets

---

## Workload Management (WLM)

### Auto WLM (recommended)

- ML-based dynamic management of memory and concurrency
- Adjusts query queue depth based on workload characteristics
- 2024 improvements: better resource prediction

### Manual WLM

- Define custom queues with fixed memory % and slot count
- Assign queries to queues by user group or query group
- Useful when you need predictable resource allocation per workload

### Other WLM features

- **Short Query Acceleration (SQA)** — sends short queries to a dedicated express queue (works with manual WLM)
- **Query Monitoring Rules (QMR)** — define rules that abort or log queries based on metrics (rows returned, execution time, etc.)

---

## Materialized Views

- Pre-computed query results stored as a table
- **Auto-refresh** — Redshift can automatically refresh views when base tables change
- **Incremental refresh** — only the deltas are applied, not the full recompute (faster)
- **Auto and incremental refresh on data sharing consumer tables** (Feb 2024) — consumers can have MVs that incrementally refresh from shared base tables
- Use cases: pre-aggregate dashboards, accelerate joins across large tables

---

## Data Sharing

- Share **live, transactional data** between Redshift clusters / Serverless workgroups without copying or ETL
- **Producer** creates a **datashare**; adds objects (schemas, tables, MVs); grants to consumer namespace / account
- **Consumer** creates a database from the datashare and queries it directly
- **Requires RA3** or Serverless on producer AND consumer
- **Cross-account** and **cross-Region** supported
- Producer charged for storage; consumer charged for compute on read
- **Governed by Lake Formation** since 2022 — apply column / row security to shared data
- **Data sharing writes** (newer) — consumer can also write back to producer datashare

> 💡 **Exam pattern**: "share live Redshift data with another business unit, no S3 copy, no ETL" → **Redshift Data Sharing on RA3**.

---

## Federated Query

- Query **Aurora PostgreSQL / MySQL, RDS PostgreSQL / MySQL** directly from Redshift — no ETL
- Uses **Secrets Manager** for credentials
- Common pattern: join warehouse fact tables with live operational data (e.g., current pricing in RDS + historical sales in Redshift)

> ⚠️ Federated query is NOT Spectrum, NOT data sharing, NOT zero-ETL. It's a direct live query against an operational DB.

---

## Redshift Spectrum

- Query data **directly on S3** without loading into Redshift — uses an **external schema** mapped to a Glue / Hive Metastore database
- Spectrum compute is separate from your cluster — pays per TB scanned
- Supports **Lake Formation** fine-grained permissions (column / row / cell-level)
- Best for: querying cold or massive datasets that don't fit cost-effectively in cluster storage
- Combine with cluster tables in a single query (joins between cluster and S3)

> 💡 **Exam pattern**: "query terabytes of S3 data alongside warehouse tables, no ETL" → **Redshift Spectrum**.

---

## Streaming Ingestion

- **Low-latency, high-speed** ingestion from streaming sources directly into Redshift (no S3 staging)
- Sources:
  - **Amazon Kinesis Data Streams**
  - **Amazon Managed Streaming for Apache Kafka (MSK)**
  - **Amazon Data Firehose** (newer)
- Materialized view based: define a streaming MV on the stream; queries see fresh data within seconds
- Works with both provisioned RA3 and Serverless

> 💡 **Exam pattern**: "ingest streaming data into Redshift with seconds-latency, no S3 staging" → **Redshift Streaming Ingestion**.

---

## Zero-ETL Integrations

Replicate transactional data from operational stores into Redshift **continuously, in near real-time**, with no ETL pipeline to build:

- **Aurora MySQL → Redshift** (GA 2023)
- **Aurora PostgreSQL → Redshift**
- **RDS for MySQL → Redshift** (GA 2024)
- **RDS for PostgreSQL → Redshift**
- **DynamoDB → Redshift**
- **Salesforce → Redshift**
- **Multi-AZ deployment** supported with zero-ETL (April 2024)

**How it works**: AWS-managed continuous replication using internal change streams; no triggers / CDC tooling needed.

> 💡 **Exam pattern**: "near-real-time analytics on operational data without building ETL" → **Zero-ETL integration**.

---

## Redshift ML

- Train and deploy **machine learning models using SQL** in Redshift
- Behind the scenes: uses **Amazon SageMaker AutoPilot** to train; deploys as a function inside Redshift
- Inference via SQL function call — no data movement
- Use cases: predict churn, fraud scoring, customer LTV, demand forecasting
- **BYOM (Bring Your Own Model)** — import existing SageMaker models

---

## Data Loading

- **`COPY`** command — bulk load from S3 (parallel across slices), DynamoDB, EMR, remote hosts via SSH
- **`UNLOAD`** — export data to S3 (use Parquet for compact format)
- **Streaming Ingestion** — direct from Kinesis / MSK / Firehose
- **Zero-ETL** — continuous replication from operational stores
- **Federated Query** — live query (no copy)
- **Glue ETL / EMR / Spark** — batch ETL into Redshift

---

## Backup and Disaster Recovery

- **Automated snapshots** every 8 hours or every 5 GB of data change
- Default retention: **1 day**; configurable up to **35 days**
- Manual snapshots: retained until you delete them
- **3 copies of data** maintained: original, replica on compute nodes, S3 backup
- **Cross-Region snapshot copy** — async replicate to another Region for DR
- **Cross-account snapshot sharing** for backup centralization
- **Multi-AZ on RA3** provides automatic failover within Region (HA, not DR)
- For cross-Region DR: cross-Region snapshots OR cross-Region data sharing on RA3

---

## Security

- **Data in transit**: SSL / TLS
- **Data at rest**: AES-256, encrypted with **KMS** or **CloudHSM**
- You can **modify an unencrypted cluster to enable KMS encryption** — Redshift transparently migrates data to a new encrypted cluster
- **VPC enhanced routing** — force all traffic through your VPC (for VPC endpoints, NAT, etc.)
- **Audit logging** to CloudWatch Logs or S3 (connection log, user log, user activity log)
- **IAM authentication** for database users (no password management)
- **Row-level security** and **column-level security** via Lake Formation (when using Spectrum) or native Redshift RLS (provisioned + Serverless)
- **Dynamic data masking** — mask sensitive columns at query time based on user role
- **Trusted Identity Propagation (TIP)** with IAM Identity Center (March 2025) — propagate user identity from Identity Center into Redshift queries, including Redshift Data API

---

## Networking

- Cluster lives in your **VPC**
- **Cluster relocation** (RA3) — move cluster between AZs without snapshot restore
- **VPC endpoints** for private connectivity from VPC (no internet)
- **PrivateLink** for connecting to Redshift from other VPCs / accounts
- **Enhanced VPC routing** — required if you want all `COPY` / `UNLOAD` traffic to go through VPC endpoints / NAT

---

## Pricing

### Provisioned

- Per-node-hour billing (only compute nodes; leader is free)
- Reserved Instances: 1-year or 3-year commitments for significant discounts
- RA3: pay separately for compute (node-hour) and managed storage (GB-month)

### Serverless

- Per-second billing in **RPU-hours** (Redshift Processing Units)
- Pay only when queries run

### Additional charges

- Concurrency Scaling beyond free daily credit
- Spectrum: per TB scanned
- Cross-Region snapshot copy: data transfer
- Data sharing: producer storage + consumer compute on read

---

## Exam Tips

- **Redshift is OLAP, not OLTP** — for transactional workloads, use Aurora or RDS
- **RA3 or Serverless** for any new design — DC2 is legacy, DS2 is deprecated
- **Multi-AZ on RA3** (2023+) gives you HA within Region — older notes saying "Single-AZ only" are obsolete
- **Redshift Serverless** for spiky / unpredictable workloads; **provisioned** for predictable steady-state
- **Data Sharing** requires RA3 (or Serverless) on both producer AND consumer — DC2 is wrong answer
- **Federated Query** = Redshift → Aurora / RDS direct (no S3, no ETL)
- **Spectrum** = Redshift → S3 query (no load, no ETL)
- **Streaming Ingestion** = Kinesis / MSK / Firehose → Redshift directly (no S3 staging)
- **Zero-ETL** = Aurora / RDS MySQL / RDS PostgreSQL / DynamoDB → Redshift continuously
- **Concurrency Scaling** for BI dashboard spikes (1 hour free per day per cluster)
- **AQUA** is automatic on RA3 — for scan-heavy queries
- **Auto WLM** > manual WLM in most cases
- **Materialized Views with auto + incremental refresh** = the answer for pre-aggregated dashboards
- **Redshift ML** = train + infer via SQL using SageMaker AutoPilot
- For **fine-grained per-user audit** on Redshift queries → **Trusted Identity Propagation with IAM Identity Center**
- For **column / row / cell-level security**: native Redshift RLS, or Lake Formation when using Spectrum
- **Cross-Region DR**: snapshot copy OR cross-Region data sharing

---

## Exam Traps

- **"Redshift Single-AZ only"** — OBSOLETE. Multi-AZ on RA3 has been available since 2023
- **"Data sharing on DC2"** — wrong. Data sharing requires **RA3** or Serverless
- **DC2 / DS2 in correct answers** — both are legacy. RA3 or Serverless for new designs
- **Confusing Spectrum, Federated Query, Data Sharing, Zero-ETL** — they're four different patterns:
  - **Spectrum** = Redshift queries S3
  - **Federated Query** = Redshift queries Aurora / RDS live
  - **Data Sharing** = Redshift ↔ Redshift live sharing
  - **Zero-ETL** = Aurora / RDS / DDB → Redshift continuous replication
- **"Use Redshift for OLTP"** — wrong; columnar storage is awful for single-row CRUD. Use Aurora / RDS / DynamoDB
- **Leader node is NOT charged** — but compute nodes ARE. Some questions trick on this
- **"Use indexes for query performance in Redshift"** — Redshift does NOT use indexes. Use **sort keys + distribution keys + zone maps + compression**
- **"Modify cluster encryption requires recreating from snapshot"** — actually Redshift handles the migration transparently; no manual restore
- **"Spectrum charges per query"** — actually per **TB scanned**; partition / Parquet / compression to reduce cost
- **"Streaming Ingestion via Firehose to S3 then COPY"** — that's the OLD pattern. Streaming Ingestion is direct (no S3 hop)
- **"Zero-ETL replaces ETL pipelines fully"** — it covers replication; transformations / business logic still need Glue / Lambda / dbt downstream
- **"Federated Query supports MongoDB / DynamoDB"** — NO. Federated Query = PostgreSQL / MySQL (Aurora + RDS only)
- **Concurrency Scaling = unlimited free** — wrong; 1 hour free per day, then pay per second
- **"Serverless can't use data sharing"** — wrong; Serverless supports data sharing both as producer and consumer
- **"Cross-account data sharing requires VPC peering"** — wrong; uses RAM under the hood, no networking changes

---

## Quick Reference

| Feature                                 | Service / Setting                          | Notes                            |
| --------------------------------------- | ------------------------------------------ | -------------------------------- |
| OLAP analytics, petabyte scale          | Redshift (provisioned RA3 or Serverless)   | Columnar, MPP                    |
| HA within Region                        | Multi-AZ on RA3                            | 2023+ feature                    |
| Cross-Region DR                         | Cross-Region snapshot copy or data sharing | RA3 required for sharing         |
| Query S3 directly                       | Redshift Spectrum                          | Pay per TB scanned               |
| Query Aurora / RDS directly             | Federated Query                            | PostgreSQL / MySQL only          |
| Share live data between clusters        | Data Sharing                               | RA3 required                     |
| Replicate operational data continuously | Zero-ETL                                   | Aurora, RDS MySQL / PG, DynamoDB |
| Ingest streaming data                   | Streaming Ingestion                        | Kinesis, MSK, Firehose           |
| ML in SQL                               | Redshift ML                                | Backed by SageMaker AutoPilot    |
| Burst capacity                          | Concurrency Scaling                        | 1 hr free / day                  |
| Hardware acceleration                   | AQUA                                       | Automatic on RA3                 |
| Workload tuning                         | Auto WLM (recommended)                     | ML-based                         |
| Per-user audit                          | TIP + IAM Identity Center                  | March 2025                       |
