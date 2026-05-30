# Amazon AppFlow

> **Fully managed integration service that transfers data between SaaS apps (Salesforce, ServiceNow, Slack, Zendesk, Google Analytics, etc.) and AWS services (S3, Redshift, EventBridge) — no code required. Bidirectional flows, scheduled or event-triggered. Supports field mapping, filtering, validation, masking. Common SAP-C02 use case: pull data from external SaaS into a data lake without writing custom ETL. Often confused with Glue (Glue is for AWS-native ETL; AppFlow is for SaaS connectors).**

Maps to: **Domain 4.3 — Modernization**, **Domain 4.4 — Modernization opportunities**

---

## Overview

- **No-code data transfer** between SaaS apps and AWS
- 70+ pre-built connectors
- Bidirectional: SaaS → AWS or AWS → SaaS
- Triggered by: **schedule**, **event**, or **on-demand**
- Transformation: field mapping, validation, masking, filtering
- Pay per flow run + data processed

## Supported Sources / Destinations

### Sources (SaaS apps)
- Salesforce, Salesforce Marketing Cloud / Pardot
- Zendesk, ServiceNow, Jira
- Slack, Mailchimp, SendGrid
- Google Analytics, Google Ads
- Marketo, Singular
- SAP OData
- Datadog, Dynatrace, New Relic
- Microsoft Dynamics 365
- LinkedIn Ads
- Trend Micro
- Snowflake (read)

### Destinations
- **Amazon S3** (most common — feeds data lake)
- **Amazon Redshift** (direct load)
- **Amazon EventBridge** (real-time event routing)
- **Snowflake** (write back)
- **Salesforce, Marketo, Zendesk, Slack** (write back)
- **Upsolver, Singular** etc.

## Flow Configuration

1. **Source connector** (e.g., Salesforce account)
2. **Trigger** — run on schedule (cron) / event / on-demand
3. **Destination** (e.g., S3 bucket / Redshift table)
4. **Field mapping** — map source fields → destination fields
5. **Validation rules** — drop / log invalid records
6. **Filters** — only sync records matching criteria
7. **Masking** — mask PII fields

## Key Features

- **Incremental transfer** — sync only changed records (last-modified field)
- **Full transfer** — full dataset (use for first sync or recovery)
- **Event-driven** — Salesforce change → AppFlow → S3 in seconds
- **PrivateLink** — connector traffic stays in AWS network for supported connectors
- **Encryption** — TLS in transit; KMS at rest in destination
- **Up to 100 GB per flow run**

## Triggers in Depth

### On-demand
- Run manually or via API call

### Scheduled
- Cron expression — minutely up to daily
- Use for periodic incremental sync

### Event-driven
- Source app publishes change event → AppFlow triggers
- Sub-minute latency from change to destination
- Available for select connectors (Salesforce, Zendesk, etc.)

## AppFlow vs Glue vs EventBridge

| Tool | When |
|---|---|
| **AppFlow** | SaaS app ↔ AWS, no-code, simple field mapping |
| **Glue** | AWS-native ETL, complex transformations, big data |
| **EventBridge SaaS** | Real-time event-driven integration (no field transformation) |
| **DMS** | Database-to-database migration / CDC |
| **Lambda** | Custom code transformation glue |

**Rule of thumb**: SaaS data into S3 → **AppFlow**. AWS-to-AWS ETL with complex transforms → **Glue**. SaaS events triggering Lambda → **EventBridge SaaS partners** or **AppFlow event-driven**.

## EventBridge Comparison

- **EventBridge** has **SaaS partner sources** (Datadog, MongoDB, Zendesk, etc.)
- For **event routing** with no data transformation — EventBridge is simpler
- For **data sync** with field mapping + transformation — AppFlow

## Pricing

- **$0.001 per flow run** + processing charges
- **$0.30 per GB processed** (varies by connector)
- PrivateLink supported with extra charge

## Common Patterns

### Salesforce → S3 data lake
- Daily scheduled flow: Salesforce accounts + opportunities → S3 (Parquet)
- Athena queries on top
- Downstream: QuickSight dashboards for sales

### Real-time SaaS event → SNS
- Salesforce contact created → AppFlow event-driven → EventBridge → Lambda → SNS
- Sub-minute latency from CRM update to notification

### Bidirectional ServiceNow ↔ AWS
- AWS-generated tickets → ServiceNow via AppFlow
- ServiceNow status updates → AppFlow → DynamoDB

### Marketing data warehouse
- Google Analytics + Salesforce Pardot + Marketo → S3 via AppFlow daily
- Glue catalogs + Redshift Spectrum reads
- Marketing team's single source of truth

## Limitations

- **Not a general ETL engine** — for SaaS connectors specifically
- **Transformations are limited** — basic field map / filter / mask; no complex logic
- For complex transforms: AppFlow → S3 → Glue ETL → final destination

## Exam Tips

- "Transfer Salesforce / Zendesk / Slack data to S3 without code" → **Amazon AppFlow**
- "No-code SaaS-to-AWS integration" → **AppFlow**
- "70+ pre-built SaaS connectors"
- Triggers: **schedule / event-driven / on-demand**
- Destinations: **S3, Redshift, EventBridge, SaaS write-back**
- Supports field mapping, filtering, validation, masking
- PrivateLink for in-network traffic

## Exam Traps

- **AppFlow ≠ Glue** — Glue is AWS-native ETL; AppFlow is SaaS connectors
- **AppFlow ≠ EventBridge SaaS** — EventBridge is event routing without transformation; AppFlow is data sync with mapping
- **Not for AWS-to-AWS data movement** — use DMS / Glue / Step Functions instead
- **100 GB per flow run cap** — large datasets need partitioning across flows
- **Event-driven trigger only on select connectors** — check connector docs
- **Field-level transformations are basic** — for complex logic, land in S3 then Glue
- **No built-in retry for source-app failures** — must monitor + manually re-run
- **Some connectors require OAuth setup** in the SaaS app first
