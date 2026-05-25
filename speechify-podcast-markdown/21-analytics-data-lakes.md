# Podcast 21 — Analytics & Data Lakes

**Length target**: 15 min
**Repo references**: `domain-2-new-solutions/14-redshift.md`, `15-athena.md`, `16-opensearch.md`, `17-lake-formation.md`, `18-glue.md`, `19-quicksight.md`, `37-other-services.md` (EMR), `45-data-exchange.md`, `41-appflow.md`

## Topic & Scope

The analytics stack: Redshift (DW), Athena (serverless SQL on S3), Lake Formation (governance), Glue (ETL + catalog), QuickSight (BI), EMR (Hadoop/Spark), OpenSearch (search/logs), plus AppFlow + Data Exchange for ingestion.

## Service Coverage Depth

**Deep**:
- Redshift — clusters vs Serverless, RA3 nodes with managed storage, Spectrum, data sharing, federated queries
- Athena — serverless SQL on S3, partitioning, workgroups, federated query, ACID via Iceberg/Hudi/Delta
- Lake Formation — fine-grained access control on S3 data lakes
- Glue — Data Catalog, crawlers, ETL jobs

**Brief**:
- QuickSight (BI dashboards, SPICE)
- EMR (managed Hadoop / Spark / Hive / Presto)
- OpenSearch (logs + search)
- AppFlow + Data Exchange (ingestion)

## Structured Outline

1. **Open (30s)** — "Analytics stack: warehouse, lake, ETL, BI, search. Six services, one mission — turn data into decisions."
2. **Redshift (4 min)** — provisioned clusters (RA3 with managed storage) vs Redshift Serverless, leader + compute nodes, Spectrum (query S3), data sharing (cross-cluster), federated query (RDS/Aurora), Materialized Views, AQUA
3. **Athena (3 min)** — serverless SQL on S3, Parquet/ORC dramatically cheaper than CSV/JSON, partitioning critical, workgroups for cost control, federated query (Lambda connectors), Athena for Apache Spark
4. **Lake Formation + Glue (3 min)** — Lake Formation: fine-grained row/column-level access, LF-tags, cross-account sharing of catalog. Glue: Data Catalog (shared across Athena/EMR/Redshift Spectrum), crawlers (schema inference), ETL jobs (Spark)
5. **EMR + OpenSearch (2 min)** — EMR: managed Hadoop ecosystem (Spark, Hive, Presto, HBase), EMR Serverless, EMR on EKS. OpenSearch: search + log analytics
6. **QuickSight (1 min)** — BI dashboards, SPICE in-memory engine, ML insights, embedded analytics
7. **Ingestion bridges (30s)** — AppFlow (SaaS), Data Exchange (third-party datasets)
8. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- Redshift RA3: managed storage (S3-backed) — scales storage independently of compute
- Redshift Spectrum: query S3 data directly without loading
- Redshift Serverless: auto-scale + pay per RPU-hour, no cluster management
- Data sharing: read-only cross-cluster access without copying
- Federated query: Redshift can query RDS/Aurora live
- Athena costs $5 per TB scanned — use Parquet + partitions to slash cost
- Athena federated query: query non-S3 sources via Lambda connector
- Athena for Apache Spark: serverless Spark notebook environment
- Lake Formation: single permissions layer for S3 data lakes across Athena/Redshift/EMR
- LF-tags: tag-based access control instead of resource-by-resource grants
- Glue Data Catalog: AWS's metastore (Hive-compatible), shared across analytics services
- Glue crawlers: auto-discover schema in S3
- Glue ETL: serverless Spark + Python Shell jobs, DataBrew for visual data prep
- EMR Serverless: serverless Hadoop, no cluster management
- EMR on EKS: run Spark on existing EKS clusters
- OpenSearch: search engine + log analytics, OpenSearch Dashboards (Kibana fork)
- OpenSearch Serverless: serverless OpenSearch
- QuickSight: SPICE in-memory accelerator, per-session pricing, embedding API
- AppFlow: SaaS app data sync (Salesforce, ServiceNow, Slack)
- Data Exchange: subscribe to third-party datasets (financial, weather, demographics)

## Must-Mention Exam Traps

- EXAM TRAP: Redshift is OLAP / data warehouse — NOT OLTP (use RDS/Aurora for OLTP)
- EXAM TRAP: Redshift RA3 separates compute from storage; older DC2 doesn't
- EXAM TRAP: Athena is for one-off / interactive queries on S3 — for scheduled large jobs, EMR or Glue ETL is cheaper
- EXAM TRAP: Athena scans entire file if no partitioning — partition + Parquet is the cost rule
- EXAM TRAP: Lake Formation enforces permissions ONLY when accessed via integrated services (Athena, Redshift Spectrum, EMR, Glue) — direct S3 access bypasses
- EXAM TRAP: Glue ETL is Spark + serverless — not a workflow orchestrator (use Step Functions or MWAA)
- EXAM TRAP: Glue Data Catalog ≠ Lake Formation — Catalog is metadata; Lake Formation is permissions
- EXAM TRAP: EMR vs Glue ETL — EMR for complex/custom Hadoop ecosystem; Glue for simple Spark ETL
- EXAM TRAP: OpenSearch ≠ Athena — OpenSearch indexes for search; Athena queries S3
- EXAM TRAP: QuickSight uses SPICE for fast in-memory queries — direct query option exists but slower for large data
- EXAM TRAP: AppFlow ≠ Glue — AppFlow connects SaaS apps; Glue does AWS-native ETL
- EXAM TRAP: Data Exchange ≠ AppFlow — Data Exchange brings in third-party datasets; AppFlow syncs SaaS data
- EXAM TRAP: Redshift Spectrum is BILLED PER TB SCANNED of S3 — same partition pruning rules as Athena
- EXAM TRAP: OpenSearch UltraWarm + Cold tiers exist for cost-effective log retention

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| Massive structured analytics, SQL warehouse | Redshift |
| Serverless ad-hoc SQL on S3 | Athena |
| ETL with Spark / Python | Glue ETL |
| Full Hadoop ecosystem (HBase, Presto, Hive, Spark) | EMR |
| Search + log analytics | OpenSearch |
| BI dashboards | QuickSight |
| Fine-grained S3 lake permissions | Lake Formation |
| Centralized data catalog | Glue Data Catalog |
| Subscribe to third-party data | Data Exchange |
| Sync Salesforce / ServiceNow data | AppFlow |
| Real-time low-latency analytics | OpenSearch / Timestream / Kinesis Analytics |
| Cross-cluster Redshift sharing without copy | Redshift Data Sharing |

## Tone & Style

- Frame stack as ingest → catalog → transform → store → query → visualize
- For each service, give the "owner" (DBA / Data Engineer / Analyst)
- Repeat "partition + Parquet" as the Athena cost rule

## Rapid-Fire Closer

"Serverless SQL on S3?" — "Athena." "OLAP warehouse?" — "Redshift." "Hadoop ecosystem?" — "EMR." "Fine-grained S3 lake permissions?" — "Lake Formation." "Search engine?" — "OpenSearch." "BI dashboards?" — "QuickSight."
