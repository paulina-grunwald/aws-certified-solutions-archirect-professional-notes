# Amazon CloudWatch

> **AWS's unified observability platform: metrics, logs, alarms, events (via EventBridge), traces (via X-Ray), and curated higher-level features. The answer for "monitor / detect / alert on operational state". Complementary to CloudTrail (audit) and AWS Config (resource state history).**

Maps to: **Domain 1.2 — Prescribe security controls** (centralized monitoring, alerting, observability)

---

## Overview

The three pillars of observability — **metrics, logs, traces** — are all in CloudWatch:

- **Metrics**: numeric time series (CPU, latency, request count, ...)
- **Logs**: structured / unstructured log events
- **Traces**: distributed traces via X-Ray (surfaced in CloudWatch ServiceLens)
- **Events**: via EventBridge (formerly CloudWatch Events)
- **Curated solutions**: Application Insights, Container Insights, Lambda Insights, Synthetics, ServiceLens, Internet Monitor, Network Monitor, GenAI Observability

> **CloudWatch is about performance.** CloudTrail is about API audit. AWS Config records resource state.

---

## CloudWatch Metrics

- **Standard metrics** — published automatically by AWS services
- **Custom metrics** — your application / scripts push metrics via `PutMetricData`
- **High-resolution metrics** — granularity of **1s, 5s, 10s, 30s** (vs default 60s) — for sub-minute alarms / Auto Scaling reactions
- **Detailed monitoring** for EC2 — sends metrics every **1 minute** (vs default 5 min). Required for fine-grained Auto Scaling
- **Dimensions** — up to 30 per metric; metrics keyed by name + dimensions
- **Metric Math** — derive new metrics from existing ones (sums, rates, anomalies)
- **Metric Streams** — continuously stream metrics to Kinesis Firehose → S3 / Splunk / Datadog / 3rd-party (lower latency than polling APIs)

### Anomaly Detection
- ML-based dynamic thresholds adapting to seasonality and trends
- Alarm fires when metric is outside the predicted band
- Use case: alarming on traffic / latency / queue depth without manual thresholds

---

## CloudWatch Logs

- **Log Group**: container; configure retention (1 day to 10 years or never expire) + KMS encryption per group
- **Log Stream**: sequence from one source (e.g., one EC2 instance, one Lambda invocation)
- **Sources**: SDK / CLI `PutLogEvents`, CloudWatch Agent on EC2 / on-prem, Lambda, ECS, EKS, ELB access logs, Route 53, VPC Flow Logs, CloudTrail, RDS, API Gateway, IoT
- **Encryption**: SSE-S3 default; **KMS CMK** at the log-group level

### Metric Filters
- Pattern (literal / JSON / structured) against logs → publishes a metric → alarm
- **No regex** — only supported pattern syntax
- Common: filter `403`, `5xx`, `Unauthorized` → metric → alarm → SNS

### Subscriptions
- Real-time stream to **Kinesis Data Streams**, **Kinesis Data Firehose**, **Lambda**, or **OpenSearch**
- Filters select which events flow
- Use case: centralized aggregation, SIEM ingestion, real-time alerting

### Export to S3
- Batch export for archival / Athena queries
- **Up to 12 hours** for export to complete
- **SSE-S3 only** (no KMS at export)
- Triggered via `CreateExportTask`

### Logs Insights
- SQL-like query language
- Fields prefixed with `@` (e.g., `@timestamp`, `@message`, `@logStream`)
- **15-min query timeout** per query
- Up to **20 log groups** queried simultaneously
- Pricing: per GB scanned

### Cross-Account Observability
- One account = **monitoring account**
- Multiple **source accounts** share metrics, logs, traces, Application Insights, Container Insights
- Eliminates log-shipping pipelines for cross-account visibility
- Works with **AWS Organizations**

### Data Protection Policies
- Detect and redact (mask) **sensitive data** (PII, secrets, credit cards) in log events
- Built-in and custom data identifiers
- Optional audit event when sensitive data is detected
- Use for HIPAA / PCI / GDPR

---

## CloudWatch Alarms

- **States**: `OK`, `ALARM`, `INSUFFICIENT_DATA`
- **Period** + **Evaluation Periods** + **Datapoints to Alarm** — `M of N` evaluation logic
- **Missing data treatment**: `notBreaching`, `breaching`, `ignore`, `missing`
- Actions: SNS, Auto Scaling, EC2 (recover, reboot, stop, terminate), Systems Manager, EventBridge
- **EC2 Instance Recovery** — auto recover failing EC2 via alarm action; preserves instance ID, private IP, EIP, metadata
- Test with `aws cloudwatch set-alarm-state`

### Composite Alarms
- Aggregate multiple alarms with **AND / OR** boolean logic
- Reduces noise; alert only when correlated conditions fire
- Example: page on-call only if both `CPU > 80%` AND `RequestErrors > 5%`

---

## CloudWatch Dashboards

- Cross-Region, cross-account widgets (with Cross-Account Observability)
- Widgets: metrics, logs, alarms, text, anomaly detection bands
- Auto-refresh
- **Sharing**: public, share with specific accounts, or embedded IAM
- Pricing: first 3 free; charged after

---

## CloudWatch Agent (Unified Agent)

- One agent for **EC2** AND **on-prem** servers (Linux + Windows)
- Collects **metrics** (CPU, memory, disk, swap, custom procstat, StatsD, collectd) AND **logs**
- Configured via JSON file or **SSM Parameter Store**
- Requires IAM role on instance with `CloudWatchAgentServerPolicy`
- **procstat plugin** — process-level metrics (CPU/memory per process)
- Install via **SSM Distributor** or Run Command for fleet-wide deployment

> Standard EC2 metrics do **NOT** include memory or disk — install the CloudWatch Agent for these.

---

## CloudWatch Synthetics (Canaries)

- Synthetic scripted HTTP monitoring — simulate user flows from outside your VPC
- Canaries in **Node.js or Python** using Puppeteer / Selenium-like APIs
- Detect: broken links, slow loads, login failures, certificate problems, transactional failures
- Alarms on success rate, duration, HTTP status
- For proactive synthetic SLO monitoring

---

## CloudWatch ServiceLens (with X-Ray)

- Combines metrics, logs, and traces in a single view
- Service topology graph; click a node → metrics + traces + logs
- Built on **AWS X-Ray** + CloudWatch

---

## CloudWatch Application Insights

- Curated monitoring for **.NET, SQL Server, MySQL, Postgres, web servers, SAP HANA, Active Directory**
- Auto-discovers resources in a CloudFormation stack / Resource Group
- Pre-built dashboards, alarms, ML anomaly detection
- Sets up CloudWatch Agent, metric filters, alarms automatically

---

## CloudWatch Container Insights

- Curated monitoring for **ECS, EKS, Kubernetes on EC2, Fargate**
- Auto-instruments containers via CloudWatch Agent or ADOT
- Pre-built dashboards: CPU, memory, network, disk, pod restarts
- **Enhanced observability** — container-level metrics for EKS without manual ADOT setup
- Logs via FluentBit / Fluentd integrations

---

## CloudWatch Lambda Insights

- Curated monitoring for **AWS Lambda**
- System metrics: CPU, memory, init duration, network, fd usage
- Enabled via **Lambda Insights Layer**
- Pre-built dashboard; integrates with X-Ray and ServiceLens

---

## CloudWatch Contributor Insights

- Top-N analytics over CloudWatch Logs
- Find top **IPs, users, API calls, errors** — "top talkers" leaderboard
- Define rules with patterns + key fields
- Use case: hot keys, abusive callers, top 5xx producers

---

## CloudWatch Internet Monitor

- Measures **internet performance** between AWS workloads and end users
- Identifies BGP / ISP outages / performance degradation impact
- Generates **health events** with affected geographies + performance/availability scores
- Integrates with CloudWatch Alarms and EventBridge
- Use case: SLO monitoring for user-facing internet-served workloads

---

## CloudWatch Network Monitor / Network Flow Monitor

- **Network Monitor** — synthetic probing of network paths between on-prem and AWS (DX / VPN)
- **Network Flow Monitor** — passive flow analytics from VPC traffic; detect packet loss, latency, jitter without flow logs
- Use case: pinpoint hybrid network issues, route problems, AZ-level impairments

---

## CloudWatch GenAI Observability

- Metrics + traces for **Amazon Bedrock**, foundation models, agents, knowledge bases
- Latency, token usage, model invocations, error rates
- X-Ray integration for end-to-end GenAI traces

---

## EventBridge (formerly CloudWatch Events)

- "CloudWatch Events" is now **Amazon EventBridge** (same service, newer brand; still under CloudWatch in the console)
- Event-driven routing: AWS service events, custom events, SaaS partner events → targets (Lambda, SNS, SQS, Step Functions, Kinesis, ECS, ...)
- **Schedules**: rate / cron
- **Patterns** match events
- Heavily used for security automation (GuardDuty / Security Hub findings → remediation)

---

## Cost Optimization

- **Logs retention** — set per log group; default is forever (expensive)
- **Logs Insights queries** — billed per GB scanned; narrow scope + time range
- **Custom metrics** — billed per metric; high-cardinality dimensions multiply cost
- **Detailed monitoring** on EC2 — only enable where needed
- **Dashboards** — first 3 free; charged after
- **Metric Streams** > polling APIs for high-volume export

---

## Exam Tips

- **CloudWatch vs CloudTrail vs Config**: CloudWatch = performance/ops; CloudTrail = API audit; Config = resource state history
- **High-resolution metrics** (1s) for sub-minute alarms / Auto Scaling reactions
- **Detailed monitoring on EC2** = 1-min metrics; default basic = 5-min
- **Metric Filters do NOT support regex** — only literal / JSON / structured patterns
- **Logs Insights**: SQL-like, 15-min timeout, 20 log groups, `@`-prefixed fields
- **Subscriptions** = real-time fan-out (Kinesis / Firehose / Lambda / OpenSearch); **S3 Export** = batch archival
- **Cross-Account Observability** shares metrics/logs/traces with a monitoring account via Organizations
- **Data Protection Policies** mask PII / secrets in logs
- **Composite Alarms** = boolean logic across alarms (reduce noise)
- **EC2 Instance Recovery** is a CloudWatch Alarm action that preserves IP/EIP/metadata
- **Container Insights / Lambda Insights / Application Insights** = curated dashboards for ECS/EKS/Fargate, Lambda, common app stacks
- **Synthetics canaries** = outside-in synthetic monitoring (broken links, login flows, certs)
- **ServiceLens** = CloudWatch + X-Ray topology view
- **Contributor Insights** = top-N analytics on logs
- **Anomaly Detection** = ML-based dynamic alarm thresholds
- **Internet Monitor** = AWS↔end-user internet paths
- **Network Monitor / Flow Monitor** = hybrid + VPC network observability
- **EventBridge ≡ CloudWatch Events**
- **Metric Streams** > polling APIs for high-volume export
- **CloudWatch Agent (Unified)** is required for host-level metrics (memory, disk) from EC2

---

## Exam Traps

- **CloudWatch is NOT a security service** — for threat detection use GuardDuty; vulnerability scans use Inspector; resource posture use Security Hub / Config
- **Standard EC2 metrics do NOT include memory or disk** — install the CloudWatch Agent
- **Metric Filters publish metrics, not findings** — for findings use GuardDuty / Security Hub
- **Logs Insights timeout is 15 min** — large queries should be narrowed or sent to Athena
- **Logs Subscriptions support a limited target set** — Kinesis / Firehose / Lambda / OpenSearch. NOT S3 directly (use Firehose or Export)
- **Export to S3 is batch (up to 12h)** — NOT real-time. Real-time = Subscription
- **Export to S3 uses SSE-S3 only** — KMS not applied at export
- **Cross-Account Observability requires opt-in** in both monitoring and source accounts
- **EventBridge ≠ SNS / SQS** — EventBridge routes events with schema and filtering; SNS / SQS are pub-sub / queue
- **Alarm `INSUFFICIENT_DATA` is not `ALARM`** — verify missing-data treatment explicitly
- **Custom metrics cost per metric**; high-cardinality dimensions multiply billing
- **Logs KMS encryption is per log group** — KMS key must be in the same Region
- **Container Insights for EKS requires CloudWatch Agent or ADOT** — does not auto-instrument
- **Internet Monitor ≠ Network Monitor** — Internet Monitor measures end-user-to-AWS internet paths; Network Monitor probes VPN / DX hybrid paths
