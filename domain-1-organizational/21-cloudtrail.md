# AWS CloudTrail

> **Records and stores API activity across your AWS account — every action taken in the console, CLI, SDK, or by an AWS service. The canonical "who did what, when, from where" audit log.**

Maps to: **Domain 1.2 — Prescribe security controls** (centralized logging and audit)

---

## Overview

- Logs **every API call** in your account — including who, what, when, where, and result
- Enabled by default (90-day **event history** in the console — free)
- Create **trails** for longer retention, S3 export, multi-Region, multi-account, advanced filtering
- Logs in JSON; deliveries to S3 within ~15 minutes
- Integrates with **CloudWatch Logs**, **Athena** (query via SQL), **EventBridge** (event-driven action)

---

## Event Types

### Management Events (default)
- Control-plane operations (create / update / delete resources, IAM changes, console sign-in)
- Examples: `RunInstances`, `CreateBucket`, `AssumeRole`
- Two categories: **Read** and **Write** — logged separately, can be configured independently
- **Free** for the first copy of management events per trail

### Data Events
- High-volume data-plane operations
- **S3 object-level** (GetObject, PutObject, DeleteObject)
- **Lambda function invocations** (Invoke)
- **DynamoDB item-level**
- **EBS direct APIs**, **CloudTrail Lake queries**, etc.
- **Paid** — charged per 100,000 events

### Insights Events (paid)
- ML-based anomaly detection on API call rates and error rates
- Flags unusual patterns (e.g., sudden spike in `RunInstances`, unusual error rates)

---

## Trail Types

### Single-Region Trail
- Captures events from one Region
- Less common — most use multi-Region

### Multi-Region Trail
- One trail captures **all Regions**' events
- Recommended for compliance / centralized audit
- New Regions added automatically

### Organization Trail
- Created in the **AWS Organizations management account** (or delegated admin)
- Captures events from **all accounts in the org**, automatic across new accounts
- Cannot be modified by member accounts
- Common pattern: trail → centralized S3 bucket in **log archive account** (immutable with Object Lock + Vault Lock)

---

## CloudTrail Lake

- **Managed data lake** for CloudTrail events — queryable with SQL
- **Up to 10-year retention** (vs trails to S3 + Athena which require self-management)
- Event data stores can ingest CloudTrail, AWS Config configuration items, and **non-AWS sources** (e.g., apps, third-party SaaS)
- Pre-built queries for common audit scenarios
- Higher cost than S3 + Athena but operationally simpler

---

## Log Integrity (tamper detection)

- **Log file validation** — CloudTrail computes SHA-256 hash of each log file and chains hashes
- Generated digest file every hour
- Allows detection of: log deletion, modification, gaps in the log chain
- Required for many compliance frameworks (PCI, HIPAA)

---

## Centralized Logging Pattern

1. **Log archive account** in AWS Organizations (dedicated, locked-down)
2. Create **Organization trail** in management or delegated admin
3. Deliver to **S3 bucket in log archive account** (with S3 Object Lock in compliance mode)
4. Optionally enable **S3 access logging** on the central bucket
5. Use **CloudTrail Lake** OR **Athena** for queries
6. Send selected events to **CloudWatch Logs** for real-time alerting
7. Route critical events to **EventBridge** → Lambda for automated response

---

## EventBridge Integration

- CloudTrail emits events to EventBridge as they occur
- Route to Lambda / SNS / SQS / Step Functions for automated response
- **Real-time** (vs S3 / Lake which have minutes of delay)
- Common use: alert on root user login, IAM policy changes, security group changes

---

## CloudWatch Logs Integration

- Stream CloudTrail events to **CloudWatch Logs** for real-time monitoring
- Define **CloudWatch metric filters** + **alarms** on patterns
- Example alarms: unauthorized API calls, root user login, IAM policy changes, security group changes
- Common compliance pattern (CIS AWS Foundations Benchmark)

---

## Security Best Practices

- Enable **multi-Region trail** by default
- Use **Organization trails** for org-wide coverage
- Deliver to **centralized log archive account** with restricted access
- Enable **log file integrity validation**
- Enable **S3 Object Lock** on the log bucket (compliance mode for true immutability)
- Use **separate KMS key** for trail encryption
- Set up **CloudWatch alarms** for critical events (root login, IAM changes, etc.)
- Use **SCPs** to prevent member accounts from disabling CloudTrail

---

## Pricing

- **Event history (90 days)** — free, no setup needed
- **First copy of management events** in a trail — free
- **Data events** — $0.10 per 100,000 events
- **Insights events** — $0.35 per 100,000 events analyzed
- **CloudTrail Lake** — per-GB ingestion + storage; query charges

---

## Exam Tips

- CloudTrail is the answer for **"who did what, when, from where"** — API audit
- **Event history (90 days)** is free; for longer retention create a **trail**
- **Multi-Region trail + Organization trail** for org-wide compliance
- **Log file integrity validation** for tamper detection — required by many frameworks
- For querying CloudTrail logs use **Athena (over S3 trail logs)** OR **CloudTrail Lake (managed)**
- For real-time alerts use **EventBridge** or **CloudWatch Logs + metric filters**
- Centralized log archive account + **S3 Object Lock** = immutable audit trail
- Use **SCPs** to prevent disabling CloudTrail in member accounts
- For DR / ransomware concerns combine with **AWS Backup logically air-gapped vault**

---

## Exam Traps

- CloudTrail and Config are **complementary, not interchangeable** — CloudTrail logs API calls (who did what); Config tracks resource state and compliance
- Data events are **off by default** — S3 object-level or Lambda invoke logging must be explicitly enabled in the trail configuration
- Data events are **charged per 100K events** — high-volume S3 buckets can generate massive bills if enabled wholesale; scope by bucket / prefix
- Trail delivery to S3 has **up to 15-minute latency** — it is not real-time. For real-time use EventBridge
- Organization trails are **centrally managed** — member accounts cannot disable or modify them
- Log file integrity validation is **not enabled by default** — you must explicitly turn it on per trail
- CloudTrail Lake has **separate billing** from trails — it can be cheaper for ad-hoc queries vs maintaining S3 + Athena yourself
- CloudTrail captures **only management events by default** — data-plane calls like `s3:GetObject` are data events and not captured unless enabled
- Cross-Region DR of trail logs is **your responsibility** — cross-region replication of the trail S3 bucket is not automatic
- CloudTrail Lake "event data store" and CloudTrail "trail" are **different products** under the same brand — don't confuse them
