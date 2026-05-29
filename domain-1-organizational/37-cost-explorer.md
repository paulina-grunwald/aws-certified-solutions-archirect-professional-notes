# AWS Cost Explorer

> **Visualize, understand, and forecast AWS spend. Filter / group by dimension (service, account, Region, tag, charge type). Daily / monthly / hourly granularity (12 months back). Forecasting up to 12 months ahead. Savings Plans + Reserved Instance recommendations. Cost Categories for custom grouping. Reports for budget planning. Pairs with AWS Budgets (alerts) and Cost Anomaly Detection (ML).**

Maps to: **Domain 1.5 — Cost optimization + visibility**, **Domain 3.5 — Improve cost**

---

## Overview

- **Free** to use (UI; CSV/PDF exports)
- **Granularity**: hourly (last 14 days) / daily / monthly
- **History**: 12 months by default; can extend
- **Forecast**: up to 12 months ahead with ML
- **API**: programmatic access via `ce` API (charges per request)

## Core Views

- **Cost & Usage** — total spend over time
- **Reservations** — Reserved Instance coverage + utilization
- **Savings Plans** — coverage + utilization + recommendations
- **Right Sizing Recommendations** — surface idle / over-provisioned EC2

## Filter / Group Dimensions

- Service, Linked Account, Region, Availability Zone, Instance Type, Usage Type, Operation, Purchase Option, Tag, **Cost Category**, Charge Type (Usage, Recurring, Refund, Credit, Tax)

## Savings Plans + RI Recommendations

- Analyze past usage; recommend specific Savings Plan commitment
- Show payback period + savings estimate
- Compute Savings Plans vs EC2 Instance Savings Plans vs RIs

## Cost Categories

- Define **logical buckets** for cost grouping (e.g., "Production", "Customer X")
- Rule-based mapping of resources to categories
- Use in Budgets, reports, dashboards

## Right Sizing Recommendations

- Suggest **smaller / right instance types** based on CloudWatch util
- Identify **idle instances** (avg CPU < 1% for 4+ days)
- Estimated monthly savings per recommendation
- **Compute Optimizer** is the deeper ML-based alternative

## Multi-Account & Organizations

- **Management account** sees all linked accounts' costs
- **Cost Explorer access** can be granted to member accounts (org setting)
- **Consolidated billing** pools usage for volume discounts

## Common Patterns

### Monthly cost review

- Group by Service → identify drift
- Group by Tag → cost attribution by team / project
- Compare month-over-month with absolute + percentage delta

### Reserved Instance / Savings Plan strategy

1. Review past 12-month usage in Cost Explorer
2. Generate Savings Plan recommendation
3. Commit per workload classification

### Cost attribution

- Tag resources by `team`, `environment`, `project`
- Group by tag in Cost Explorer
- Build Cost Categories for non-tag rules

## Cost Optimization Hub

- **Aggregated recommendations** from Cost Explorer + Compute Optimizer + Trusted Advisor + RDS
- Single dashboard across accounts
- Estimated savings totaled

## Pricing

- **Free** UI + reports
- **API requests**: $0.01 per request (pre-paid)
- **Hourly granularity + resource-level**: $0.01 per 1,000 line items (opt-in)

## Exam Tips

- "Visualize + forecast AWS spend" → **Cost Explorer**
- "Savings Plan / RI recommendations" → **Cost Explorer**
- "Right-size EC2 / Lambda / EBS / Fargate / RDS" → **Compute Optimizer** (deeper ML); Cost Explorer surfaces basic right-sizing
- "Custom cost grouping" → **Cost Categories**
- "Aggregated cost optimization across recommendations" → **Cost Optimization Hub**
- "Per-tag cost breakdown" → tag resources + group by tag in Cost Explorer
- **Management account** view across all linked accounts
- **API charges** for programmatic access ($0.01/request)
- **Hourly + resource-level granularity is opt-in** ($0.01 per 1,000 line items)

## Exam Traps

- **Cost Explorer is NOT real-time** — daily data appears ~24 hours later
- **Cost Explorer ≠ Cost & Usage Report (CUR)** — CUR is detailed line-item dump to S3 for analysis (Athena / Redshift); Cost Explorer is UI visualization
- **Cost Explorer recommendations are conservative** — Compute Optimizer is more aggressive (ML)
- **Hourly + resource-level granularity is opt-in** and costs extra
- **Cost Explorer API calls are paid** — design caching for dashboards
- **Tags must be activated as cost allocation tags** before they appear in Cost Explorer (activation is per-tag, in Billing Console)
