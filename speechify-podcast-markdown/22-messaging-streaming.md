# Podcast 22 — Messaging & Streaming

**Length target**: 15 min
**Repo references**: `domain-2-new-solutions/24-sqs.md`, `25-sns.md`, `26-eventbridge.md`, `27-amazon-mq.md`, `28-kinesis.md`

## Topic & Scope

Decoupling + event-driven + streaming. SQS, SNS, EventBridge, Amazon MQ, the Kinesis family (Data Streams, Firehose, Flink, Video), and MSK. The most-confused four are SQS vs SNS vs EventBridge vs Kinesis.

## Service Coverage Depth

**Deep**:
- SQS — Standard vs FIFO, visibility timeout, DLQ, long polling
- SNS — pub/sub, fan-out, message filtering, FIFO topics
- EventBridge — event bus, rules, schema registry, SaaS partner sources
- Kinesis Data Streams — shards, retention, consumers (KCL, Enhanced Fan-Out)
- Kinesis Data Firehose — managed delivery to S3/Redshift/OpenSearch/Splunk
- Kinesis Managed Service for Apache Flink — real-time stream processing

**Brief**:
- Amazon MQ (managed ActiveMQ + RabbitMQ for migration)
- MSK (managed Kafka — covered in repo with Kinesis)

## Structured Outline

1. **Open (30s)** — "Decouple, fan out, stream, lift-and-shift. Each problem has a specific service."
2. **SQS (3 min)** — Standard (high throughput, at-least-once) vs FIFO (in-order, exactly-once, 300 msg/s per group or higher with high-throughput mode), visibility timeout, DLQ, long polling
3. **SNS (2.5 min)** — pub/sub, fanout to SQS / Lambda / HTTP / email / SMS / mobile push, message filtering, FIFO topics, SNS-SQS fanout pattern
4. **EventBridge (3 min)** — default + custom event buses, rules with pattern matching, SaaS partner sources, archive + replay, schema registry, pipes for source→target transformations
5. **Kinesis Family (4 min)** — Data Streams (sharded log, 24h–365d retention), Firehose (managed delivery to S3/Redshift/OpenSearch/Splunk/HTTP), Managed Service for Apache Flink (real-time processing), Video Streams (camera ingest)
6. **MSK + Amazon MQ (1 min)** — MSK = managed Kafka (KRaft mode, Serverless option); Amazon MQ = managed ActiveMQ / RabbitMQ for migration
7. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- SQS Standard: high throughput, at-least-once, best-effort ordering, no duplication guarantee
- SQS FIFO: in-order, exactly-once, 300 msg/s per group (3,000 with batching), high-throughput mode for 70,000 msg/s
- Visibility timeout: time message hidden after receive; default 30s, max 12h
- DLQ: redrive after N receive failures; useful for poison messages
- Long polling: 20s, reduces empty receives + cost
- SNS pub/sub: one message → many subscribers
- Fanout pattern: SNS topic → multiple SQS queues
- SNS FIFO topic ↔ SQS FIFO queue: ordered fanout
- Message filtering: subscribers filter on attributes
- EventBridge: rule with event pattern matches → invoke target (Lambda, SQS, Step Functions, Kinesis, etc.)
- EventBridge SaaS partners: Datadog, MongoDB, Zendesk, Salesforce, etc. — native integration
- Pipes: source → optional filter → optional enrichment → target (low-code)
- Schema Registry: auto-discover + version event schemas
- Archive + Replay: rewind events for testing / recovery
- Kinesis Data Streams: shards (1 MB/s in, 2 MB/s out per shard), Enhanced Fan-Out (2 MB/s per consumer per shard)
- Kinesis retention: default 24h, up to 365 days
- Firehose: managed batch + delivery, transformation via Lambda, near-real-time (60s minimum buffer)
- Managed Flink: KPL + KCL + SQL + Java/Python for stream processing
- Video Streams: live + on-demand video, HLS playback, integrates with Rekognition Video
- MSK: managed Kafka, MSK Serverless eliminates capacity planning, IAM auth
- Amazon MQ: managed ActiveMQ + RabbitMQ — for migrating apps that use AMQP/MQTT/STOMP/WSS/OpenWire

## Must-Mention Exam Traps

- EXAM TRAP: SQS is PULL-based; SNS is PUSH-based — fundamental difference
- EXAM TRAP: SQS Standard is AT-LEAST-ONCE — duplicates possible
- EXAM TRAP: SQS FIFO has 300 msg/s default — 70k with high-throughput mode, both per-message-group
- EXAM TRAP: SNS without filtering sends to ALL subscribers — use filter policies to subset
- EXAM TRAP: SNS-SQS fanout requires subscription permission on SQS to allow SNS
- EXAM TRAP: EventBridge ≠ SNS — EventBridge filters by event content (pattern), SNS by attributes
- EXAM TRAP: EventBridge has limit ~14 KB event size — for large payloads pass S3 reference
- EXAM TRAP: Pipes is for source-to-target with one transform — not a full workflow (use Step Functions)
- EXAM TRAP: Kinesis Data Streams ≠ Firehose — DS is shard-based real-time with custom consumers; Firehose is managed delivery with built-in destinations
- EXAM TRAP: Firehose minimum buffer is 60s OR 1MB — not true real-time
- EXAM TRAP: Kinesis Enhanced Fan-Out has its own cost — uses HTTP/2 push for low latency
- EXAM TRAP: Managed Service for Apache Flink replaces Kinesis Data Analytics
- EXAM TRAP: MSK is Kafka — for AWS-native streaming use Kinesis Data Streams (lower ops)
- EXAM TRAP: Amazon MQ is for MIGRATION of existing apps — for new apps use SNS/SQS or Kafka

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| Decouple producer + consumer at scale | SQS |
| Ordered exactly-once messaging | SQS FIFO |
| One-to-many notification | SNS |
| Fanout: one event → many systems | SNS + SQS / EventBridge |
| Event routing with content pattern matching | EventBridge |
| Integrate SaaS apps via event | EventBridge SaaS partners |
| Real-time streaming with custom consumers | Kinesis Data Streams |
| Managed delivery to S3 / Redshift / OpenSearch | Kinesis Data Firehose |
| Real-time stream processing (windowing, aggregation) | Managed Flink |
| Live video ingest | Kinesis Video Streams |
| Existing Kafka workload | MSK |
| Lift-and-shift ActiveMQ / RabbitMQ | Amazon MQ |
| Order delivery with deduplication | SQS FIFO with content-based deduplication |

## Tone & Style

- Hammer the SQS vs SNS vs EventBridge vs Kinesis distinction
- Use the "decouple / fanout / route / stream" framing
- Repeat Firehose's 60s minimum (not real-time)

## Rapid-Fire Closer

"PULL or PUSH for SQS?" — "PULL." "Fanout one event many systems?" — "SNS or EventBridge." "Real-time custom consumers?" — "Kinesis Data Streams." "Managed delivery to S3?" — "Firehose." "Migrate RabbitMQ?" — "Amazon MQ." "Kafka?" — "MSK."
