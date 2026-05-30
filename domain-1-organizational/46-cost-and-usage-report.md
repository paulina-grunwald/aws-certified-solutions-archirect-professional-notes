# AWS Cost and Usage Report (CUR)

> **The most granular billing data AWS provides. Delivers detailed CSV/Parquet line-item reports to S3, refreshed up to 3x/day. Each row is one usage record per resource, per usage type, per hour (or per day/month). Foundation for custom cost analytics — query with Athena, Redshift Spectrum, QuickSight, or third-party FinOps tools. CUR 2.0 ships via Data Exports with simplified schema. Use when Cost Explorer's grouping isn't granular enough or when you need to integrate billing data into your own data warehouse.**

Maps to: **Domain 1.5 — Cost optimization + visibility**, **Domain 3.5 — Improve cost**

---

## Overview

- **Most detailed billing data** AWS publishes
- Each line = one resource × one usage type × one time period
- Delivered to **S3** as CSV / Parquet / Apache ORC
- **Refreshed up to 3× per day**
- **Free to generate** (you pay S3 storage + any query costs)
- Configure once; runs forever

## Two Versions

### CUR Legacy (still supported)
- Configure via Billing console → Cost & Usage Reports
- Schema with ~150+ columns
- Includes manifest + folder structure per billing period

### CUR 2.0 (Data Exports)
- Configured via **Data Exports** in Billing console
- **Simplified schema** (~50 columns)
- SQL-defined customization — filter / aggregate at source
- Recommended for **new** setups
- Parquet by default

## Schema Highlights

Key columns (legacy CUR):

- `bill_payer_account_id` — management account
- `line_item_usage_account_id` — account that incurred usage
- `line_item_product_code` — service (e.g., `AmazonEC2`)
- `line_item_usage_type` — granular usage (e.g., `BoxUsage:m5.large`)
- `line_item_resource_id` — ARN or resource ID
- `line_item_blended_cost` / `line_item_unblended_cost` — $ before / after RI sharing
- `pricing_term` — `OnDemand` / `Reserved`
- `reservation_*` — RI / SP attribution
- `savings_plan_*` — SP attribution
- `resource_tags_user_*` — your cost allocation tags

## Where CUR is Stored

- An S3 bucket you own
- AWS writes hourly or daily roll-ups to `s3://bucket/prefix/year=YYYY/month=MM/`
- **Manifest JSON** describes the partitions
- Lifecycle: typically transition to Glacier after 90 days

## Querying CUR

### Amazon Athena (most common)
- Use CloudFormation template AWS provides → creates external table over CUR
- SQL queries on top of CUR data
- Pay per query (Athena pricing)

### Amazon Redshift / Redshift Spectrum
- Load into Redshift, or query S3 directly via Spectrum
- Best when you have other data warehouse use cases

### Amazon QuickSight
- Native CUR data source connector
- Dashboards on top of Athena queries
- Most-used SAP-C02 "build a cost dashboard" answer

### Third-Party Tools
- CloudHealth, Cloudability, Vantage, Apptio — all ingest CUR
- Common for enterprise FinOps

## CUR vs Cost Explorer vs Budgets

| Need | Tool |
|---|---|
| Custom dashboards / data-warehouse integration | **CUR** (raw data) |
| Interactive analysis, charts, recommendations | **Cost Explorer** |
| Threshold alerts + automated actions | **Budgets** |
| ML-based anomaly detection | **Cost Anomaly Detection** |
| All four together | Yes — they complement |

## Cost Allocation Tags

- Tags do NOT appear in CUR by default — must **activate** them in Billing → Cost Allocation Tags
- Once activated, appear as `resource_tags_user_<tagkey>` columns
- Takes up to 24h to populate after activation
- **Tag everything** in advance — historical data without tags is lost forever

## CUR for Multi-Account Orgs

- Set up CUR in the **management account**
- Includes all linked-account usage
- Use `line_item_usage_account_id` to split costs per account
- Combine with **AWS Organizations OUs** for hierarchical reporting

## Common Patterns

### Per-team chargeback
- Activate `Team` tag
- Athena query: `SUM(unblended_cost) GROUP BY resource_tags_user_team`
- QuickSight dashboard → shared with finance

### Untagged-resource detection
- Query CUR for rows missing the required `Team` tag
- Surface untagged resources back to teams to remediate

### Daily cost trend with anomaly detection
- Athena query on daily aggregates
- Pipe to QuickSight for sparklines
- Combine with Cost Anomaly Detection for alerts

### RI / SP utilization deep dive
- Cost Explorer summarizes; CUR has line-item attribution
- Identify which workloads consume the SP / RI commitment
- Detect under-utilized RIs by linked account

### FinOps showback
- CUR data + tags → cost per product/team/environment
- Monthly showback reports per team

## Setting Up CUR

1. Create / select S3 bucket (bucket policy must allow AWS Billing to write)
2. Billing console → Data Exports → Create report
3. Choose schema (CUR Legacy or CUR 2.0)
4. Include resource IDs + split cost allocation data (recommended)
5. Choose delivery frequency (hourly or daily)
6. Wait up to 24h for first delivery
7. Run Athena setup CloudFormation template → external table ready

## Pricing

- **CUR generation is free**
- **S3 storage** for the reports (~MBs–GBs)
- **Athena queries** at $5 per TB scanned (partition + Parquet to keep cheap)
- **QuickSight** per-user / per-session

## Exam Tips

- "Most detailed billing data" → **CUR**
- "Custom cost dashboards in QuickSight" → **CUR → Athena → QuickSight**
- "Per-resource, per-hour billing detail" → **CUR**
- "Refreshed 3× per day"
- **Cost Allocation Tags must be activated** to appear in CUR
- Stored in S3 as CSV / Parquet / ORC
- **CUR 2.0 via Data Exports** is the modern path

## Exam Traps

- **CUR ≠ Cost Explorer** — CUR is raw data; Cost Explorer is the UI/analysis tool
- **Tags don't appear in CUR unless activated** in Billing — easy to miss
- **Historical data has no tags** if you activate late — only forward-looking data carries tags
- **CUR is generated from the management account** — linked accounts can't configure their own CUR (they get summary in their console)
- **CUR is per-billing-period** — files are appended/updated throughout the month
- **Querying without partition pruning is expensive** — always filter by year/month in Athena
- **CUR 2.0 schema is simpler but different** — migration from legacy schema needs query rewriting
- **Don't store CUR in same bucket as data** — separate bucket for clean lifecycle / IAM
- **CUR delivery delay** — up to 24h for first report after setup
