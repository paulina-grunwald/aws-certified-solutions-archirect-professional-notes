# AWS Budgets

> **Set custom cost, usage, RI/Savings Plans utilization, or RI/SP coverage budgets with alert thresholds. Different from Cost Explorer (analysis tool) — Budgets is for proactive alerting + automated actions. Supports forecasted alerts (predicted to exceed) + actual alerts. Budget Actions can auto-apply IAM policies / SCPs / stop EC2/RDS when threshold breached. Integrates with EventBridge + SNS for downstream automation. Use with Cost Anomaly Detection (anomaly-based) for full cost-control toolkit.**

Maps to: **Domain 1.5 — Cost optimization**, **Domain 3.5 — Improve cost**

---

## Overview

- **Set cost / usage / RI utilization / RI coverage / Savings Plans utilization / Savings Plans coverage budgets**
- Alert on **forecasted** (predicted to exceed) or **actual** (already exceeded) thresholds
- Per-account or **consolidated across Organization**
- **Budget Actions** — auto-remediation when budget breached
- **Free** for first 2 budgets; **$0.02 per budget per day** after

## Budget Types

| Type | Tracks |
|---|---|
| **Cost budget** | $ spend per period |
| **Usage budget** | Quantity (e.g., GB-month S3, EC2 instance-hours) |
| **RI utilization budget** | % of RI capacity actually used |
| **RI coverage budget** | % of eligible usage covered by RIs |
| **Savings Plans utilization budget** | % of committed SP spend used |
| **Savings Plans coverage budget** | % of eligible usage covered by SPs |

## Budget Periods

- Daily / Monthly / Quarterly / Annually
- Custom start/end dates supported
- **Monthly is most common** — aligns to billing cycle

## Filters & Scope

- Filter by **service, linked account, tag, region, instance type, purchase option, charge type**
- Combine filters: e.g., "EC2 cost, prod tag, us-east-1, On-Demand only"
- **Cost Categories** can be used as filters for advanced grouping

## Alerts

- Up to **5 alerts per budget**
- Trigger: actual ≥ X% **OR** forecasted ≥ X%
- Notification: **SNS topic** or **email** (up to 10 recipients)
- Common pattern: 50% / 80% / 100% / 110% thresholds with escalating recipients

## Budget Actions (Auto-Remediation)

When threshold breached, Budgets can **automatically execute** one of:

- **Apply IAM policy** to a user / group / role (e.g., deny EC2 launch)
- **Apply SCP** to an OU (e.g., deny new resource creation)
- **Stop / restart EC2 / RDS instances** (by tag or ID)
- **Send to EventBridge** (Lambda / Step Functions for custom action)

Two execution modes:
- **Automatic** — runs immediately on breach
- **Manual approval** — pending until approver acts

This is the **key differentiator vs Cost Anomaly Detection** (which only alerts).

## EventBridge Integration

- Budget events published to default bus
- Detail type: `AWS Budgets Alarm`
- Build downstream: notify Slack, page on-call, freeze IAM policies

## Multi-Account / Organization

- Create budgets in **management account** scoped to specific member accounts
- Use **linked account** filter to drill into individual account spend
- Use **Cost Categories** for logical grouping (per team, per product)

## Common Patterns

### Dev account spend cap
- Cost budget: $5,000/month on dev account
- Action at 90%: Apply SCP that denies new EC2/RDS launches
- Forces dev teams to clean up before adding more

### RI / Savings Plans utilization tracking
- SP utilization budget: alert if < 80% utilized
- Indicates over-committed SP — adjust next purchase

### Forecasted overrun for slow-leak
- Forecasted alert at 100% triggers mid-month
- Catches slow cost creep before it hits the budget ceiling

### Tag-based product cost tracking
- Budget per product tag (`Product=Search`)
- Each product team owns their budget + alerts

## Budgets Reports

- Schedule **email reports** (daily / weekly / monthly)
- Summary of budget status sent to recipients
- Useful for exec dashboards without giving Cost Explorer access

## Pricing

- **First 2 budgets free**
- **$0.02 per budget per day** after
- Budget Actions: **$0.10 per action triggered** (after first 100/month)
- SNS / Lambda / SCP costs separate

## Budgets vs Cost Explorer vs Cost Anomaly Detection

| Need | Tool |
|---|---|
| **Set threshold, alert + auto-remediate** when exceeded | **Budgets** |
| **Analyze historical cost trends, group by service/tag/account** | **Cost Explorer** |
| **Detect unusual spend patterns** (ML-based) | **Cost Anomaly Detection** |
| **Forecast future spend** | Cost Explorer (forecast feature) or Budgets (forecasted alerts) |

All three are commonly tested together — know which to pick.

## Exam Tips

- "Alert when EC2 spend exceeds $X" → **AWS Budgets cost budget**
- "Auto-stop EC2 instances when monthly budget breached" → **Budgets + Budget Action (stop EC2)**
- "Alert before spend exceeds budget" → **Budgets forecasted alert**
- "Track RI / Savings Plans utilization" → **Budgets RI/SP utilization budget**
- Up to **5 alerts per budget**; SNS or email
- **First 2 budgets free**; $0.02/day after
- Budget Actions = IAM policy / SCP / stop EC2/RDS / EventBridge

## Exam Traps

- **Budgets ≠ Cost Explorer** — Budgets is proactive (alerting + action); Cost Explorer is retrospective (analysis)
- **Budgets ≠ Cost Anomaly Detection** — Budgets uses fixed thresholds; CAD uses ML to detect anomalies regardless of budget
- **Budgets actions execute with the permissions you give the action role** — scope tightly (least privilege)
- **Forecasted alerts can fire repeatedly** — fix the forecast logic if you get noise
- **SP/RI utilization budget tracks USAGE of committed capacity** — not whether you bought enough; coverage budget is for that
- **Tag-based budgets need tag activation** — activate cost allocation tags in Billing console first
- **Member accounts need management account permission** to create org-wide budgets
- **Budget Actions delay** — can take up to a few hours to apply after threshold breach
