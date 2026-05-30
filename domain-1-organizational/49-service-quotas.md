# AWS Service Quotas

> **Centralized view + management of AWS service limits ("quotas") across all services + Regions + accounts. Replaces opening individual support tickets for each limit increase. Request quota increases programmatically, set CloudWatch alarms when approaching limits, and manage org-wide via Service Quotas + Organizations. Critical for SAP-C02 scenarios about scaling, capacity planning, and accidental outages caused by hitting service limits.**

Maps to: **Domain 3.1 — Reliability**, **Domain 3.6 — Operational excellence**, **Domain 1.4 — Multi-account governance**

---

## Overview

- **One place** to view + manage AWS service quotas (formerly "service limits")
- Per service, per Region, per account
- Two quota types:
  - **Adjustable** — can request increase via Service Quotas
  - **Non-adjustable** — hard caps (rare; e.g., EC2 instances per ENI, S3 bucket name uniqueness)
- Request increases via console / API / CLI — most are auto-approved for reasonable values

## Quota vs Limit

- AWS rebranded **"service limits"** as **"quotas"** when launching Service Quotas service
- Same concept: caps on resources you can create or consume

## Common Quota Examples

| Service | Quota example | Default |
|---|---|---|
| EC2 | Running On-Demand instances per family | Varies (often 5 vCPU starting) |
| VPC | VPCs per Region | 5 |
| Lambda | Concurrent executions per account | 1,000 |
| S3 | Buckets per account | 100 (request up to 1000) |
| RDS | DB instances per Region | 40 |
| IAM | Roles per account | 1,000 |
| CloudFormation | Stacks per account | 2,000 |
| API Gateway | Throttle rate per account | 10,000 RPS |

## Requesting Increases

- Service Quotas console / API
- Choose service → quota → request new value
- AWS reviews — most reasonable requests auto-approve in minutes; large requests can take days
- **Plan ahead** for known scaling needs (Black Friday, product launch, migration)

## CloudWatch Integration

- Service Quotas publishes metrics for current usage / quota
- **Set CloudWatch alarms** at 80% of quota
- Get notified BEFORE hitting the wall
- Critical for production systems where hitting a limit = outage

## Trusted Advisor Integration

- Trusted Advisor has **Service Limits checks** (Basic plan, free)
- Shows usage % vs quota for common quotas
- Less granular than Service Quotas but easier overview

## Organization-Wide Quota Management

- **Quota request templates** — apply standard quota increases to new accounts in Org
- Use case: every new prod account auto-requests EC2 vCPU quota = 5,000, Lambda concurrency = 5,000, etc.
- Saves manual ticket-opening per new account

## Common Patterns

### Pre-launch capacity planning
- Identify expected peak load (e.g., 50,000 Lambda concurrent invocations)
- Check current quota (1,000 default)
- Request increase to 100,000 a week before launch
- Verify approval; load-test against approved limit

### Production safety alarms
- Service Quotas + CloudWatch alarm at 80% for: EC2 vCPUs, Lambda concurrency, VPC endpoints, RDS instances
- Alarm → SNS → page on-call
- Pre-empt outages from quota exhaustion

### New account auto-bootstrap
- Quota request template applied to OU
- New account in OU automatically gets standard quota adjustments
- Combined with Control Tower / Account Factory for full automation

### Migration capacity check
- Pre-migration: list all required services / quantities
- Service Quotas check for any limits below need
- Bulk-request increases via API

## Service Quotas vs Limits That Don't Show

Not every cap is in Service Quotas. Some examples:
- **DynamoDB** throughput is per-table (configured at table level)
- **Aurora** I/O is not a quota
- **API Gateway** throttling can be account-level (quota) or stage-level (config)
- **CloudFront** distribution count is in Service Quotas; cache behaviors per distribution are not

## Pricing

- **Free** — Service Quotas itself
- CloudWatch alarms cost standard CloudWatch pricing

## Exam Tips

- "Centralized view of AWS service limits" → **Service Quotas**
- "Alarm before hitting a quota" → **Service Quotas + CloudWatch alarm at 80%**
- "Auto-apply quota increases to new accounts" → **Quota request templates** (Org integration)
- "Increase EC2 vCPU limit for Black Friday" → **Service Quotas increase request**
- Free service
- Replaces support ticket for most quota increases
- Some quotas are **non-adjustable** (hard caps)

## Exam Traps

- **Not every limit is in Service Quotas** — some are per-resource (DynamoDB throughput, Aurora I/O)
- **Non-adjustable quotas can't be raised** — must architect around (e.g., split workload across accounts)
- **Quota increases take time** — large increases can take days; plan ahead
- **Quotas are per-Region** — increase in us-east-1 doesn't increase eu-west-1
- **Trusted Advisor service limits ≠ Service Quotas** — TA is read-only summary; Service Quotas is manage + request
- **CloudWatch metrics may lag** — alarm at 80% gives buffer; don't set at 95%
- **Bursty workloads can trip soft quotas quickly** — combine with retry/backoff at app layer
- **Account-level vs API-level throttling** — API Gateway has both; tune the right one
- **Quotas reset when account closed + reopened** — not preserved
