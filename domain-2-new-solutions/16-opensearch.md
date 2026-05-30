# Amazon OpenSearch Service

> **Managed OpenSearch (Elasticsearch fork) for search, log analytics, security analytics, observability, and vector search. Two flavors: OpenSearch Service (provisioned clusters) and OpenSearch Serverless (auto-scale). OpenSearch Ingestion for managed Data Prepper pipelines. Vector search + ML for RAG / semantic search. Zero-ETL with DynamoDB, Aurora, S3. UltraWarm + Cold Storage for cost optimization.**

Maps to: **Domain 2.5 — Analytics / search**, **Domain 3.5 — Cost optimization**, **Domain 4.3 — Modernization (search, GenAI)**

---

## Naming

- **OpenSearch** = open-source project (fork of Elasticsearch + Kibana)
- **OpenSearch Dashboards** = open-source Kibana fork
- **Amazon OpenSearch Service** = AWS-managed OpenSearch + Dashboards (formerly Amazon Elasticsearch Service)

> Don't confuse with **Elastic Inc.'s** Elastic Cloud on AWS — different product.

---

## Two Deployment Flavors

### OpenSearch Service (provisioned)

- Managed clusters with chosen node types + counts
- You configure: instance types, count, AZs (Multi-AZ for HA), EBS volumes, version
- Supports **OpenSearch 1.x / 2.x / 3.x** and legacy Elasticsearch 7.x (frozen)
- **Reserved Instances** for steady-state cost savings
- **OR1 instances** — OpenSearch-optimized, segment replication, ~30% lower cost for indexing-heavy workloads
- **Service Software Updates** (managed; you trigger) for patches + features

### OpenSearch Serverless

- **No cluster management** — AWS scales compute (OCU = OpenSearch Compute Unit) on demand
- Collection types:
  - **Time series** — logs / observability, append-heavy + time-based queries
  - **Search** — full-text + vector
  - **Vector search** — dedicated vector collection
- Pay per OCU-hour for indexing + search + storage (S3-backed managed storage)
- Native VPC + KMS support; no node sizing
- **Cold start latency** on first query after idle period

---

## Storage Tiers (Provisioned)

| Tier | Latency | Cost | Best for |
|---|---|---|---|
| **Hot** | Sub-second | Highest | Active indexing + frequent queries |
| **UltraWarm** | Seconds | ~80% cheaper than hot | Recently aged data, infrequent queries |
| **Cold Storage** | Minutes | ~99% cheaper than hot | Archival; must re-attach before query |

- **Index State Management (ISM)** policies automate transitions
- **Snapshot to S3** for ultimate archive + DR

---

## OpenSearch Ingestion

- Managed **Data Prepper** pipelines — no infrastructure to run
- **Sources**: OpenTelemetry, S3, Kinesis, MSK, Kafka, CloudWatch Logs
- **Sinks**: OpenSearch (Service or Serverless), S3
- Built-in transforms: enrichment, filtering, anomaly detection, persistent buffering
- Alternative to Kinesis Firehose / Lambda for OpenSearch ingestion

---

## Vector Search & GenAI

- **k-NN plugin** for vector similarity search
- Use cases:
  - **Semantic search** — embeddings of text/images, find by meaning
  - **RAG (Retrieval-Augmented Generation)** — knowledge base for Bedrock / SageMaker
  - **Multimodal search** — search images by text descriptions and vice versa
  - **Recommendation systems** — match users to items via embeddings
  - **Fraud detection** — similarity to known fraud patterns
- **Bedrock Knowledge Bases** can use **OpenSearch Serverless (vector collection)** as the vector store
- Direct integration with **SageMaker** for embedding generation

---

## ML Features

- **Anomaly Detection** — built-in unsupervised ML for time-series anomalies
- **Learning to Rank (LTR)** — ML-based relevance ranking
- **Trace Analytics** — observability traces (OpenTelemetry)
- **Security Analytics** — pre-built detectors for security incidents
- **Alerting** — destinations (Slack, SNS, webhook, Lambda)

---

## Direct Query & Zero-ETL

- **Direct Query for S3** — query S3 logs directly from OpenSearch Dashboards without ingesting (Spark-based query engine)
- **Direct Query for CloudWatch Logs** — same pattern; no need to subscribe-and-ingest
- **Zero-ETL integrations**:
  - **DynamoDB → OpenSearch** — full-text + vector search on operational data
  - **Aurora PostgreSQL → OpenSearch** — search-optimized analytics
  - **S3 → OpenSearch** — log analytics without ETL pipelines

---

## Security

- **VPC-native** — OpenSearch domain deployed inside your VPC
- **Encryption at rest** (KMS) + **in transit** (TLS) + node-to-node encryption
- **Fine-Grained Access Control (FGAC)** — IAM, internal user DB, SAML / Cognito SSO
- **Document-level + field-level security** — restrict who sees which docs / fields
- **CloudTrail** logs all OpenSearch control-plane API calls

---

## ELK Pattern (AWS-Managed)

```
Producers → OpenSearch Ingestion / Logstash / Fluent Bit → OpenSearch Service → OpenSearch Dashboards
```

Alternative to CloudWatch Logs Insights + Dashboards — more advanced queries, longer retention control, but more ops.

---

## Comparison with Related Services

| Service | Role |
|---|---|
| **OpenSearch Service / Serverless** | Search-heavy unstructured data, large-scale vector search, log analytics |
| **Aurora PostgreSQL with pgvector** | Relational vector search (smaller scale, OLTP integrated) |
| **DynamoDB** | Key-value / document (no native search; use Zero-ETL to OpenSearch for search) |
| **Neptune** | Graph DB for relationship-focused use cases |
| **CloudWatch Logs Insights** | AWS-native log queries; lower ops; less flexible |
| **Athena** | Serverless SQL on S3 logs; cheaper for ad-hoc; not real-time |
| **Kinesis Firehose** | Stream ingestion (use OpenSearch Ingestion as alternative) |

---

## Exam Tips

- "Log analytics + dashboards beyond CloudWatch" → **OpenSearch Service**
- "Full-text search on application data" → **OpenSearch Service or Serverless**
- "Vector search / semantic search / RAG knowledge base" → **OpenSearch Serverless (vector collection)** or OpenSearch Service with k-NN plugin
- "No cluster management, auto-scale" → **OpenSearch Serverless**
- "Reduce OpenSearch cost for aged data" → **UltraWarm + Cold Storage** via ISM policies
- "Indexing-heavy workload at lower cost" → **OR1 instances**
- "Query S3 logs without ingesting into OpenSearch" → **Direct Query for S3**
- "Search on DynamoDB / Aurora data" → **Zero-ETL to OpenSearch**
- "RAG with Bedrock" → **OpenSearch Serverless vector collection** as the vector store
- "Anomaly detection in time-series logs" → **OpenSearch ML / Anomaly Detection**
- "Managed Data Prepper ingestion" → **OpenSearch Ingestion**

---

## Exam Traps

- **OpenSearch Service is NOT serverless** — use OpenSearch Serverless if cluster management isn't desired
- **OpenSearch Serverless has cold-start latency** — not ideal for sub-second sporadic queries
- **OpenSearch Service domains live in a VPC** — public endpoints exist but are discouraged
- **Cold Storage requires attaching the index to a node before query** — not instant; minutes of warmup
- **k-NN plugin must be enabled** on the cluster for vector search on OpenSearch Service
- **Direct Query is a query-time integration**, not ingestion — slower for high-frequency queries
- **OpenSearch Ingestion is NOT Kinesis Firehose** — it uses Data Prepper, supports more advanced transforms
- **Reserved Instances** are for OpenSearch Service only — Serverless has no RIs
- **CloudWatch Logs subscription → OpenSearch** still works but Direct Query and OpenSearch Ingestion are cheaper / more flexible
- **Don't confuse "OpenSearch Serverless" with "Zero-ETL"** — Serverless is a deployment model; Zero-ETL is a data integration pattern; the two can be combined
- **Multi-AZ with standby (3-AZ)** is required for production HA — single-AZ is a dev pattern
- **OpenSearch 2.x / 3.x is current** — Elasticsearch 7.x is legacy; AWS does not support newer Elasticsearch versions (Elastic.co's commercial product)
