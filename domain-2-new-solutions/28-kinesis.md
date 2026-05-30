# Amazon Kinesis (+ MSK)

> **AWS's streaming data platform. Kinesis Data Streams for real-time ingest with replay + multiple consumers. Amazon Data Firehose (formerly Kinesis Firehose) for near-real-time managed delivery to S3/Redshift/OpenSearch. Amazon Managed Service for Apache Flink (formerly Kinesis Data Analytics — SQL version discontinued Jan 2026) for real-time stream processing. Kinesis Video Streams for video. Amazon MSK for Kafka-API-compatible workloads. Pick Kinesis for AWS-native streaming, MSK for existing Kafka ecosystems.**

Maps to: **Domain 2.4 — Streaming / decoupling**, **Domain 4.3 — Modernization**

---

# Kinesis platform

Four services under the Kinesis umbrella (some renamed):

| Service | New name | Purpose |
|---|---|---|
| **Kinesis Data Streams** | (same) | Real-time ingestion, ordered shards, replay-capable |
| **Kinesis Firehose** | **Amazon Data Firehose** | Managed near-real-time delivery to AWS / third-party sinks |
| **Kinesis Data Analytics** | **Amazon Managed Service for Apache Flink** (SQL variant discontinued Jan 2026) | Real-time stream processing |
| **Kinesis Video Streams** | (same) | Video ingest from cameras / IoT |

---

## Kinesis Data Streams

- Real-time streaming data ingestion
- **Shards** partition the stream — records ordered per shard via partition key
- **Data record max 10 MB** (raised from 1 MB)
- **Retention 24 hours (default) to 365 days** — can replay any time within window
- Data is **immutable** once ingested
- **Multiple independent consumers** can read the same stream

### Capacity Modes

- **On-Demand** — auto-scales shards; pay per throughput used
- **Provisioned** — you manage shard count; cheaper at steady state, but you must scale manually
- **On-Demand Advantage Mode** — warm throughput up to 10 GiB/s, no per-stream-hour charge, 60%+ discount on throughput

### Throughput Limits

| Layer | Limit |
|---|---|
| **Producer per shard** | 1 MB/s or 1,000 records/s |
| **Classic Consumer per shard (shared)** | 2 MB/s + 5 `GetRecords`/s across all classic consumers |
| **Enhanced Fan-Out** per shard, per consumer | 2 MB/s push (≈ 70 ms latency); up to **50 enhanced consumers per stream** (raised from 20) |
| **Default shard quota per account** | 20,000 (up from 500) in major Regions |

`ProvisionedThroughputException` → add shards, switch to on-demand, or improve partition-key distribution.

### Producers & Consumers

**Producers**:
- AWS SDK (simple)
- **KPL (Kinesis Producer Library)** — batching, compression, retries (C++ / Java)
- **Kinesis Agent** — tail log files → Kinesis Data Streams or Firehose

**Consumers**:
- AWS SDK
- **Lambda** (via Event Source Mapping)
- **KCL (Kinesis Client Library)** — checkpointing to DynamoDB, coordinated reads, easy multi-instance scaling

### Use Cases

- App logs, metrics, IoT telemetry, clickstreams
- Real-time big data + analytics
- Replay / re-processing — pause a consumer, replay history

---

## Amazon Data Firehose (formerly Kinesis Firehose)

- Fully managed **near-real-time delivery** (≥ 60s buffer or ≥ 1 MB)
- **No consumers to manage** — Firehose writes for you
- **Destinations** (huge list): S3, Redshift, OpenSearch, Splunk, **Snowflake, Apache Iceberg tables, S3 Tables**, HTTP endpoints, Datadog, New Relic, MongoDB, Dynatrace, Coralogix, Elastic, etc.
- **In-flight transformation** with Lambda
- **Format conversion** (JSON → Parquet / ORC) for analytics-ready S3
- **Compression** + **encryption** at rest
- **Failed records to S3 backup bucket**
- Pay per data ingested

> If real-time flush from Data Streams to S3 is required, use **Lambda** (more expensive but sub-second). Firehose minimum buffer is 60s.

---

## Amazon Managed Service for Apache Flink

- Fully managed, serverless **Apache Flink**
- **Java, Python, Scala, SQL** (Flink SQL)
- Native auto-scaling, **exactly-once** semantics, 40+ source/destination connectors
- Durable application state with checkpointing
- Sources: Kinesis Data Streams, MSK, Kafka, files in S3
- **Use cases**: streaming ETL, continuous metrics, responsive analytics, anomaly detection, real-time dashboards

> **The old "Kinesis Data Analytics for SQL" was discontinued in Jan 2026** — never the right answer on the exam. Current service is **Managed Service for Apache Flink**.

---

## Kinesis Video Streams

- **Securely stream video** from connected devices to AWS
- For ML, analytics, generic video processing
- Use cases: doorbell cameras, drone feeds, industrial CCTV, telemedicine

---

# Amazon MSK (Managed Streaming for Kafka)

> Lift-and-shift Apache Kafka workloads, or run Kafka-API-compatible streaming on AWS.

## Deployment Modes

- **Provisioned** — you size brokers + EBS
- **Serverless** — auto-scale; up to 200 partitions / cluster
- **Express Brokers** — 3× throughput / broker, 20× faster scaling

## KRaft Mode

- Kafka 3.x — **no Zookeeper required**
- Default for new clusters running Kafka 3.7+

## MSK Connect

- Managed **Kafka Connect** workers
- Pre-built connectors (Debezium, S3 Sink, OpenSearch, JDBC, MongoDB)
- Auto-scales on CPU

## MSK Replicator

- **Fully managed cross-Region / cross-cluster replication**
- Replicates topics, partitions, consumer offsets, ACLs
- DR, regional data distribution, cluster migration
- AWS-managed alternative to MirrorMaker 2

## Tiered Storage

- **Hot tier (EBS)** + **Cold tier (S3-backed)** with auto-transition
- Long retention without ballooning EBS cost
- Requires Kafka 2.8.2.tiered+

## Security

- **VPC-only**
- **At rest**: KMS
- **In transit**: TLS (broker ↔ client + broker ↔ broker)
- **Authentication**:
  - **mTLS** via ACM Private CA
  - **SASL/SCRAM** via Secrets Manager
  - **IAM authentication** — recommended
- **Authorization**: Kafka ACLs OR IAM policies
- **PrivateLink** for cross-VPC / cross-account

## Schema Management

- **AWS Glue Schema Registry** — Avro, Protobuf, JSON Schema; compatibility checks; client-side ser/deser

---

## MSK vs Kinesis Data Streams

| Feature | **MSK** | **Kinesis Data Streams** |
|---|---|---|
| Engine | Apache Kafka | AWS-native |
| API | Kafka API | Kinesis API |
| Best for | Existing Kafka, K-native ecosystem | AWS-native streaming, simpler ops |
| Multi-cloud / hybrid | Yes | No |
| Auto-scaling | Express + Serverless | On-demand |
| Ordering | Per-partition | Per-shard |
| Retention | As long as you want | 24 h – 365 d |
| Consumer scaling | Unlimited consumer groups | Enhanced Fan-Out push |
| Setup complexity | Higher (Kafka concepts) | Lower (simpler API) |

> "Already have Kafka" → MSK. "Brand-new AWS-native streaming" → Kinesis.

---

## Exam Tips

- "Real-time streaming ingestion" → **Kinesis Data Streams**
- "Near real-time delivery to S3 / Redshift / OpenSearch with no code" → **Amazon Data Firehose** (NOT Data Streams)
- "Real-time stream processing with windowing / aggregations / joins" → **Managed Service for Apache Flink**
- "Multiple consumers reading the same stream independently" → **Enhanced Fan-Out** on Kinesis Data Streams
- "Replay / re-process streaming data" → **Kinesis Data Streams** (Firehose CANNOT replay)
- "Video from cameras / IoT for ML" → **Kinesis Video Streams**
- "Ordering matters within a stream" → **Kinesis Data Streams** (per shard via partition key)
- "Unpredictable / bursty traffic" → **On-demand mode** (Data Streams) or **Serverless** (MSK)
- "Migrate on-prem Kafka to AWS" → **MSK**
- "Cross-Region Kafka DR" → **MSK Replicator**
- "Managed Kafka Connect" → **MSK Connect**
- "Long Kafka retention without EBS cost" → **MSK Tiered Storage**
- "Kafka auth with AWS IAM" → **MSK IAM authentication**

---

## Exam Traps

- **Data Streams vs Firehose**: Streams = real-time (~200 ms), custom consumers, persistent data. Firehose = near-real-time (60 s+), managed delivery, no replay
- **Kinesis vs SQS**: Kinesis = streaming (order, replay, many consumers). SQS = message queue (decouple, one consumer per message, no replay)
- **Kinesis vs MSK**: MSK = Kafka API; Kinesis = AWS-native API. Existing Kafka clients don't work on Kinesis
- **"Kinesis Data Analytics for SQL" is DISCONTINUED (Jan 2026)** — replaced by **Managed Service for Apache Flink**
- **"Kinesis Firehose" is the OLD name** — current name is **Amazon Data Firehose**; can now ingest directly (not just from Data Streams)
- **Firehose is NOT real-time** — minimum 60-second buffer
- **Record size**: Kinesis Data Streams now supports up to **10 MB** records; Firehose single record still 1 MB
- **Enhanced Fan-Out** = dedicated 2 MB/s push per consumer (low latency); Classic = shared 2 MB/s
- **Shard splitting does NOT auto-happen in Provisioned mode** — manual or scaling policy required
- **Firehose can source from Data Streams OR directly** — don't assume Data Streams is always in front
- **MSK Serverless 200 partition / cluster limit** — for more use Provisioned or Express
- **MSK IAM auth requires AWS IAM SDK in Kafka client** — not all Kafka clients support it
- **MSK Replicator is the managed alternative to MirrorMaker 2** — never propose MirrorMaker 2 for new designs
- **MSK Tiered Storage requires Kafka 2.8.2.tiered+** — older clusters can't use it
