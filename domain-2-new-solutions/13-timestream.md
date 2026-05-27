# Amazon Timestream

> **Fully managed, serverless time-series database. Two engines: Timestream for LiveAnalytics (purpose-built, default) and Timestream for InfluxDB (managed InfluxDB-compatible). Tiered storage (memory → magnetic), built-in time-series functions, SQL queries. 1000s× faster + 1/10 cost vs RDBMS for time-series workloads.**

Maps to: **Domain 2.5 — Database selection (purpose-built)**, **Domain 4.3 — Modernization (IoT, observability)**

---

## What's a Time-Series DB

- Data arrives in **time order**, **append-only**
- Queries always span a **time interval**
- Examples: IoT sensor readings, app metrics, financial ticks, server telemetry, clickstreams
- Relational DBs are inefficient at this scale — Timestream is purpose-built

---

## Two Engines

| Engine | When to choose |
|---|---|
| **Timestream for LiveAnalytics** | AWS-native, purpose-built; greenfield workloads, IoT, AWS-integrated observability |
| **Timestream for InfluxDB** | Lift-and-shift existing **InfluxDB OSS / Enterprise** workloads to a managed service without rewriting queries (InfluxQL / Flux compatible) |

---

## Timestream for LiveAnalytics

### Features

- **Serverless** — auto-scales compute + storage independently
- **Tiered storage**:
  - **Memory tier** — recent data, fast writes + reads
  - **Magnetic tier** — historical data, low-cost, slower reads
  - Automatic migration between tiers based on retention policies
- **SQL-compatible** query layer with **time-series-specific functions** (interpolation, smoothing, derivatives, anomaly detection)
- **Scheduled queries** — pre-compute aggregations on a schedule for dashboards
- **Multi-measure records** — multiple measurements in one record (more efficient than single-measure)
- **Encryption** in transit (TLS) + at rest (KMS, AWS-owned or customer-managed)
- **Trillions of events / day** scale
- Pay per **write, query, storage** — no fixed cluster cost
- Built-in integration with **IoT Core, Kinesis Data Streams, MSK, Lambda, Grafana, QuickSight**

### Data Model

```
Database → Tables → Records

Record = Timestamp + Dimensions (tags) + Measure(s)

  Dimensions:  device_id=abc, region=eu-west-1
  Measure(s):  temperature=23.5, humidity=45 (multi-measure)
  Timestamp:   2026-05-18T13:00:00Z
```

- **Multi-measure records** group several measures with the same timestamp + dimensions — cheaper and faster than one record per measure

### Retention

- Per-table retention for memory tier (1 h – 1 y) and magnetic tier (1 d – 200 y)
- Records older than total retention are **deleted** (export to S3 for archive)

---

## Timestream for InfluxDB

- Managed **InfluxDB OSS 2.x** engine (Flux + InfluxQL)
- Same API as self-managed InfluxDB
- Lift-and-shift existing dashboards / Grafana / Telegraf agents without code changes
- Up to **3-AZ Multi-AZ** deployment
- Up to **16 TB EBS volumes** per node
- Pay for instance + storage; **not serverless**
- Use when: existing InfluxDB workloads, Flux query language preference, tight Telegraf integration

---

## Use Cases

- **IoT** — millions of devices reporting sensor data; downsampling + alerting on anomalies
- **Observability** — application + infrastructure metrics, custom telemetry
- **Industrial / SCADA** — equipment monitoring, predictive maintenance
- **Real-time analytics** — clickstream, ad tech, financial market data
- **DevOps** — APM-style metrics where CloudWatch is insufficient or too costly

---

## Integration Patterns

- **IoT Core → Timestream** via rule actions
- **Kinesis Data Streams → Lambda → Timestream** for stream processing
- **MSK / Kafka → Lambda or Connector → Timestream**
- **Grafana** + Timestream data source plugin for dashboards
- **QuickSight** for BI on time-series aggregates
- **Lambda** for scheduled batch loads or custom transformations
- **SageMaker** for ML on historical time-series (anomaly detection, forecasting)

---

## Differences from Related Services

| Service | When |
|---|---|
| **Timestream LiveAnalytics** | Serverless, AWS-native, greenfield time-series |
| **Timestream InfluxDB** | Managed InfluxDB compatibility, lift-and-shift |
| **DynamoDB** | Key-value, sub-ms reads — can store time-series but lacks time-series query functions |
| **RDS / Aurora** | Relational; works for small time-series but poor scaling vs Timestream |
| **OpenSearch** | Log analytics, full-text search; can do time-series but more expensive |
| **CloudWatch Metrics** | AWS-native infra metrics; 15-month retention; limited custom dimensions |
| **MSK / Kinesis** | Ingestion + buffering pipelines; not a database |
| **S3 + Athena** | Long-term archive of time-series in Parquet; cheaper but higher query latency |

---

## Exam Tips

- "Trillions of events / day of time-series at low cost" → **Timestream LiveAnalytics**
- "Lift-and-shift InfluxDB workload to AWS managed" → **Timestream for InfluxDB**
- "IoT sensor readings + real-time alerts + dashboards" → **IoT Core + Timestream + Grafana / QuickSight**
- "Built-in time-series functions" (interpolation, smoothing, anomaly) → **Timestream**
- "Serverless time-series database" → **Timestream LiveAnalytics**
- "Auto-tier hot data in memory, cold in magnetic" → **Timestream** tiered storage
- "SQL-compatible time-series" → **Timestream LiveAnalytics**
- "Flux / InfluxQL existing dashboards" → **Timestream for InfluxDB**

---

## Exam Traps

- **Timestream is NOT a general-purpose database** — only use for time-series workloads
- **LiveAnalytics is NOT InfluxDB-compatible** — different query language (SQL vs Flux/InfluxQL); use the InfluxDB engine if you need compatibility
- **Timestream LiveAnalytics is serverless; Timestream for InfluxDB is NOT** (instance-based)
- **Memory tier is required** — can't write directly to magnetic
- **Records older than retention are deleted** — Timestream is NOT an archive solution; export to S3 if you need long-term retention
- **Single-measure records are inefficient** — use multi-measure for production workloads
- **Cross-Region replication is NOT built-in** — implement at the ingest layer (dual-publish, MSK/Kinesis fan-out) for multi-Region time-series
- **AWS Backup is not supported for Timestream LiveAnalytics** — for InfluxDB engine, Multi-AZ + snapshots cover DR
