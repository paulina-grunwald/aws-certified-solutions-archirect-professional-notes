# Amazon Athena

> **Interactive serverless query service for S3 data using standard SQL. Engine v3 = Trino. Pay per TB scanned ($5/TB on-demand) or DPU-based Capacity Reservations. Apache Iceberg + S3 Tables for ACID + time travel. Federated Query across 30+ sources. Workgroups for cost control. Pairs with QuickSight for BI.**

Maps to: **Domain 2.5 — Analytics / database selection**, **Domain 3.5 — Cost optimization**

---

## Overview

- Interactive, **serverless** query service to analyze S3 data using **standard SQL**
- **Engine v3** is built on **Trino** (successor to Presto); v2 (Presto) is deprecated
- AWS integrates upstream Trino releases within 60–90 days — 50+ new SQL functions, 30+ new features, 90+ perf improvements vs v2
- Supports **CSV, JSON, ORC, Avro, Parquet**
- **Pricing**: $5.00 per TB scanned (on-demand) — or DPU-based **Capacity Reservations** for predictable workloads
- No ETL setup needed — queries data **in place** on S3
- Default metadata store: **AWS Glue Data Catalog**

## Use Cases

- **Business intelligence / analytics / reporting** — Athena + QuickSight
- Query **log files in S3** — ELB logs, S3 access logs, CloudTrail trails, VPC Flow Logs
- Generate **business reports** on data in S3
- Analyze **AWS Cost and Usage Reports** (CUR)
- **Click-stream analytics**
- **Ad-hoc analysis** for security, audit, troubleshooting

> **Exam tip**: "Analyze data in S3 using serverless SQL" → Athena

---

## Performance & Cost Optimization

- **Columnar formats** (Parquet, ORC) — column pruning + higher compression vs CSV/JSON
- **Compression** — data scanned **before decompression**, so compressed columnar = faster + cheaper (Snappy, Zlib, LZO, GZIP)
- **Partitioning** — divide table by properties (year/month/day, region); Athena only scans relevant partitions
- **Partition Projection** — partition rules stored in table properties (not Glue Catalog); Athena computes partitions in-memory; ideal for **millions of partitions**
- **Bucketing** — group data into buckets within partitions for efficient joins / aggregations
- **CTAS (Create Table As Select)** — transform query results into Parquet, partition + compress in one step; in-Athena ETL
- **Aggregate small files** — combine to 128 MB–512 MB for optimal performance

> Athena pricing is per TB scanned → optimizing data format and partitioning directly reduces cost.

---

## Federated Queries

- Query data across **30+ data sources** beyond S3: RDS, DynamoDB, Redshift, Aurora, MSK, on-prem databases, other clouds
- Uses **Athena Data Source Connectors** (Lambda-based) for each source
- Join results across different sources in a single SQL query

**When federated, when ETL?**

- **Infrequent queries + need current data** → **Federated Query**
- **Frequent queries + slightly stale data OK** → ETL into S3, query with standard Athena

> "Near real-time" / "always current" → Federated. "Reporting, dashboards, many users" → ETL to S3.

---

## Apache Iceberg & ACID Transactions

- **Native Iceberg support** for ACID on S3
- **INSERT, UPDATE, DELETE, MERGE** on S3 data (not possible with standard Athena tables)
- **Time-travel queries** — query data as it existed at a point in time
- **Schema evolution** — add/remove/rename columns without rewriting data
- **S3 Tables** — fully managed Iceberg tables with automatic compaction, snapshot management, optimization; **up to 3× faster query throughput** vs self-managed Iceberg

### Why Iceberg matters for deletes

Parquet files pack thousands of rows together. Deleting one customer's rows would normally require reading, filtering, rewriting every file containing those rows — error-prone ETL.

Iceberg handles this with **delete markers** (metadata): future queries skip the marked rows; background **compaction** physically cleans up later. ACID-compliant — failed deletes don't corrupt data.

> "Need to delete/update individual records in a Parquet data lake on S3" → **Athena + Iceberg**

---

## Workgroups & Cost Control

- **Workgroups** separate users, teams, applications; set per-query or per-workgroup **data usage limits**
- Track costs per workgroup via CloudWatch metrics
- Enforce query result location + encryption settings per workgroup

### Capacity Reservations

- **Provisioned capacity (DPU-based)** for predictable workloads
- **1-minute reservations** with a 4 DPU minimum
- Up to **95% savings** vs on-demand for short-duration workloads
- Configure DPU limits at workgroup or query level for concurrency control

---

## Integrations

| Service | Role |
|---|---|
| **AWS Glue Data Catalog** | Default metadata store; shared with EMR, Redshift Spectrum |
| **AWS Lake Formation** | Fine-grained (column / row / cell-level) access control on Athena queries |
| **Amazon QuickSight** | Primary BI / dashboarding tool |
| **AWS Step Functions** | Orchestrate Athena queries in workflows |
| **S3 Inventory / S3 Select** | Complement specific use cases |
| **Materialized Views** (Glue Catalog) | Pre-computed query results stored as Iceberg tables, auto-updated as source data changes |
| **Athena for Apache Spark** | Run Spark workloads on Athena infra, available in SageMaker notebooks |

---

## Exam Tips

- "Analyze data in S3 using serverless SQL" → **Athena**
- "Reduce Athena query costs" → **columnar (Parquet/ORC) + partition + compress**
- "Query data across multiple sources without moving it" → **Athena Federated Query**
- "Need to UPDATE / DELETE records in S3 data lake" → **Athena + Iceberg tables**
- "Separate Athena costs by team / department" → **Workgroups** with data usage limits
- "Ad-hoc querying of logs (CloudTrail, VPC Flow Logs, ELB, S3 access)" → **Athena**
- "Serverless analytics + dashboards" → **Athena + QuickSight**
- "Fine-grained access control on columns/rows in data lake" → **Athena + Lake Formation**
- "Pre-compute expensive query results that auto-update" → **Materialized Views** (stored as Iceberg)
- **Partition Projection** = best for tables with **millions of partitions** (avoids Glue Catalog lookups)
- **Athena Engine v3 = Trino**; Engine v2 (Presto) is deprecated
- **S3 Tables** for managed Iceberg with up to 3× faster query throughput

---

## Exam Traps

- **Athena vs Redshift Spectrum**: Both query S3 with SQL. Athena = fully serverless. Redshift Spectrum requires an existing Redshift cluster. "No infrastructure to manage" → **Athena**
- **Athena vs EMR**: Athena = simple SQL on S3. EMR = complex data processing (Spark, Hive, Flink). Don't pick Athena for heavy ETL / ML
- **Presto vs Trino**: Engine v3 = Trino. "Presto-based" answer for Athena is outdated
- **Athena is NOT real-time** — ad-hoc / batch analytical queries only; for real-time use **Managed Service for Apache Flink** (formerly Kinesis Data Analytics)
- **$5 / TB is on-demand pricing only** — Capacity Reservations are DPU-based; don't assume all Athena = per-TB
- **Federated queries ≠ ETL** — they query in-place across sources; "consolidate into a single store" = Glue ETL, not federated
- **Iceberg ≠ standard Athena tables** — only Iceberg supports UPDATE / DELETE / MERGE; standard Athena tables on S3 are append-only
- **QuickSight + Athena + KMS** — QuickSight IAM role needs `kms:Decrypt` to read KMS-encrypted query results (common troubleshooting question)
- **Athena does not store data** — it queries data in S3; the lifecycle and durability are S3's job
- **Workgroups don't prevent over-scanning** — they only set soft / hard limits; partitioning is still required for true cost control
