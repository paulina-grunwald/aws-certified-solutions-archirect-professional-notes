# AWS Glue

> **Fully managed, serverless ETL + data integration. Discover, prepare, move, and integrate data from many sources. Glue 5.0 on Spark 3.5.4, Python 3.11, Java 17, 32% faster vs Glue 4.0. Glue Data Catalog is the central metadata store (Hive-compatible). Crawlers auto-discover schemas. DataBrew for no-code prep. Glue Studio for visual ETL. Glue Streaming ETL on Spark Structured Streaming. Native support for Apache Iceberg, Hudi, Delta Lake.**

Maps to: **Domain 2.5 — Data integration / ETL**, **Domain 4.2/4.3 — Migration / modernization**, **Domain 1.2 — Lake Formation governance**

---

## Overview

- Fully managed, **serverless ETL** + data integration
- Catalog metadata, discover schemas, run Spark/Python ETL jobs, orchestrate workflows
- Pay per **DPU-hour** (Data Processing Unit) for jobs; Catalog billed per object/request
- **Glue 5.0**: Spark 3.5.4, Python 3.11, Scala 2.12.x, Java 17; **32% faster** than Glue 4.0; vectorized Iceberg / Hudi / Delta Lake
- Supports both **batch** and **streaming** ETL

---

## Core Components

### Glue Data Catalog

- **Central metadata repository** — **Hive Metastore-compatible**
- Tables, databases, partitions, schemas, table properties
- Used by **Athena, Redshift Spectrum, EMR, Glue ETL, Lake Formation, SageMaker**
- **Cross-account sharing** via Resource Access Manager (RAM) or **Lake Formation**
- One Catalog per account per Region (shareable)

### Crawlers

- Auto-discover schema in S3, JDBC, DynamoDB, MongoDB, Delta Lake, Iceberg, Hudi
- Infer table definitions and partition schemes
- Schedule (cron) or on-demand
- Update existing tables, add new partitions
- **Native recognition of Iceberg / Hudi / Delta Lake** table formats

### Glue ETL Jobs

- PySpark or Scala scripts (or Python Shell for lightweight Python jobs)
- Or **visual job** in Glue Studio (no code)
- **Job Bookmarks** — track processed data to prevent re-processing
- **Workers**:
  - **G.1X / G.2X / G.4X / G.8X** — Spark workers (G.4X / G.8X for memory-intensive jobs)
  - **G.025X** — Python Shell (small)
- **Execution classes**:
  - **Standard** — runs immediately
  - **Flex** — up to **35% cheaper**; for non-urgent jobs that can tolerate variable start time
- **Auto-scaling** — automatically adjust worker count during job

### Glue Studio (visual ETL)

- Visual ETL designer: drag-and-drop sources, transforms, targets
- Source nodes: S3, JDBC, Kafka, MSK, Kinesis, etc.
- Transforms: filter, join, custom transforms via PySpark / SQL
- **Generative AI assistant** — natural language to ETL code

### Glue DataBrew

- **No-code** visual data preparation
- **250+ pre-built transformations** for clean / normalize
- Profiles data quality, suggests transforms
- Good for analysts / data scientists; output to S3

### Glue Data Quality

- Rule-based + ML-based data quality checks
- **DQDL** (Data Quality Definition Language) rules
- Run on Catalog tables or inline in ETL jobs
- Integrates with Glue Studio, Athena, Lake Formation
- Auto-generates rules from sampled data

### Glue Streaming ETL

- Built on **Spark Structured Streaming**
- Sources: Kinesis Data Streams, MSK, self-managed Kafka, MSK Connect
- Output to S3, JDBC, Glue Catalog, etc.
- Window aggregations, joins with reference data
- **Tumbling, sliding, session** windows

### Glue Workflows / Triggers

- DAG-based orchestration of Crawlers + Jobs
- **Triggers**: schedule (cron), on-demand, on-event (EventBridge), conditional (job success/fail)
- Alternative for simple pipelines: **Step Functions** integrates with Glue

### Glue Interactive Sessions

- Jupyter / Zeppelin notebooks backed by Glue Spark
- Develop / debug ETL interactively
- Integrates with **SageMaker Studio**

### Glue for Apache Iceberg / Hudi / Delta Lake

- Native read/write to all three table formats
- Use case: transactional data lakes with UPDATE / DELETE / MERGE
- Glue 5.0 includes vectorized Iceberg / Hudi / Delta readers

### Glue Elastic Views (DISCONTINUED)

- Was preview-only; **discontinued by AWS**
- Never the right answer on the exam

---

## Security

- **Encryption at rest** (KMS) for Data Catalog, Job Bookmarks, S3 outputs, CloudWatch logs
- **Encryption in transit** with TLS
- **Identity-based IAM policies** — grant Glue access to users / groups / roles
- **Resource-based policies on Data Catalog** — cross-account access
- **Lake Formation** for fine-grained access control (column / row / cell-level)
- **VPC connections** to reach private data sources (RDS in private subnet)
- **AWS PrivateLink** for Glue API calls without internet

---

## Common Patterns

- **Data Lake on S3** — ingest → Crawler discovers → Catalog → ETL → Athena / Redshift Spectrum / EMR
- **Data warehouse modernization** — on-prem DB → Glue ETL → Redshift
- **Log analytics** — app logs in S3 → Crawler → Athena
- **ML preprocessing** — SageMaker preps via Glue DataBrew or Glue ETL
- **Real-time data processing** — Streaming ETL from Kinesis / MSK → S3 / Redshift
- **Transactional data lake** — Glue + Iceberg / Hudi / Delta on S3 for UPDATE / DELETE / MERGE

---

## Decision: Glue vs Alternatives

| Need | Service |
|---|---|
| Serverless Spark ETL with Catalog | **Glue** |
| Streaming ETL (Spark) | **Glue Streaming ETL** |
| Streaming analytics (Flink) | **Managed Service for Apache Flink** (formerly Kinesis Data Analytics) |
| Stream delivery + lightweight transforms | **Kinesis Data Firehose** |
| Visual data prep (no code) | **Glue DataBrew** |
| Long-running Spark / Hive / Presto clusters | **EMR** |
| Visual / SQL workflow orchestration | **Step Functions** (or Glue Workflows for Glue-only) |
| Workflow scheduling with retries | **Amazon Managed Workflows for Apache Airflow (MWAA)** |

---

## Exam Tips

- "Serverless ETL with Spark" → **Glue ETL**
- "Fully managed data catalog used by Athena / Redshift Spectrum / EMR" → **Glue Data Catalog**
- "Auto-discover schema in S3" → **Glue Crawler**
- "No-code visual data preparation" → **Glue DataBrew**
- "Visual ETL designer" → **Glue Studio**
- "Streaming ETL on Kinesis / MSK" → **Glue Streaming ETL**
- "Cheaper ETL for non-urgent jobs" → **Glue Flex execution class** (~35% off)
- "Memory-intensive ETL workers" → **G.4X / G.8X** workers
- "Transactional data lake with UPDATE / DELETE on S3" → **Glue + Apache Iceberg** (or Hudi / Delta Lake)
- "Data quality rules in ETL pipeline" → **Glue Data Quality**
- "Prevent re-processing already-processed data" → **Glue Job Bookmarks**
- "Cross-account Data Catalog sharing" → **RAM** or **Lake Formation**
- "Generative AI to write ETL code" → **Glue Studio with GenAI assistant**

---

## Exam Traps

- **Glue Elastic Views is discontinued** — never the right answer
- **Glue ETL is Spark / Python — not real-time streaming analytics** — for Flink-based real-time analytics use **Managed Service for Apache Flink**
- **Crawlers can over-create partitions** if your S3 path layout is messy — design partition keys (year/month/day) deliberately
- **Job Bookmarks need consistent partition / file ordering** — irregular file timestamps can confuse bookmarks
- **Glue Data Catalog ≠ Lake Formation** — Glue Data Catalog stores metadata; **Lake Formation governs access** (column / row / cell-level)
- **Glue Flex execution can take longer to start** — not for jobs with strict SLAs
- **Glue Streaming ETL** is Spark Structured Streaming — does NOT replace Kafka or Kinesis; it consumes from them
- **DataBrew is separate from Glue ETL** — different pricing, different UI, no native PySpark
- **VPC connection is required** for Glue to reach RDS / Redshift in private subnets
- **Crawler IAM role** needs `s3:GetObject` + `glue:*` permissions; common cause of "Crawler succeeded but no tables created"
