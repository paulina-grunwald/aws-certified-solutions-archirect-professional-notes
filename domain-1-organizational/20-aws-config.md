# AWS Config

> **Continuously monitors and records AWS resource configurations and evaluates them against desired configurations. Think "security camera" for AWS resources.**

Maps to: **Domain 1.2 — Prescribe security controls** (centralized audit and compliance)

---

## Overview

- Assess, audit, and evaluate configurations of AWS resources
- Continuously records configurations + changes over time
- Automated compliance checking against rules
- Sends alerts via **SNS** / **EventBridge** for non-compliance
- **Per-Region** service; aggregate across Regions and accounts
- Stores configuration history in **S3** (queryable via Athena)
- 75+ AWS-managed rules; custom rules via Lambda
- **Pricing**: $0.003 per configuration item recorded per Region, $0.001 per rule evaluation per Region (no free tier)

### Questions Config can answer
- Is there unrestricted SSH access to my security groups?
- Do my buckets have public access?
- How has my ALB configuration changed over time?
- Are my EBS volumes encrypted?

---

## Config Rules

A rule represents the desired state for resources. Rules can be triggered:
- On each config change
- At regular intervals (periodic)

### AWS Managed Rules
- Pre-built and managed by AWS
- Configure parameters and enable

### Customer Managed Rules
- Custom logic via **AWS Lambda function** invoked by the rule
- Function executes in your account

---

## Config Rule Remediation

- Automate remediation of non-compliant resources using **SSM Automation Documents**
- Use AWS-managed automation docs OR create custom ones
- **Remediation Retries** — retry if resource is still non-compliant after auto-remediation

---

## Config Rule Notifications

- Use **EventBridge** to trigger notifications when resources are non-compliant
- Route to: Lambda, SNS, SQS, Step Functions for downstream action
- **SNS** can deliver config change + compliance state notifications (all events; use SNS filtering or client-side)

---

## Conformance Packs

- Collection of Config rules + remediation actions, deployed as a **single unit**
- Defined via **YAML template** with managed or custom rules + remediation actions (Lambda)
- Deploy at account level or **across an entire AWS Organization**
- **Recommended successor to AWS Audit Manager** (which is being EOL'd for new customers April 30, 2026)

---

## Aggregator

- Aggregator collects Config data from multiple accounts and Regions into a central account
- **Rules are created in each source account** (aggregator only consolidates results)
- Sources:
  - Multiple accounts + multiple Regions
  - Single account + multiple Regions
  - An entire AWS Organization (no per-account authorization needed)
- Useful for org-wide compliance dashboards

---

## Service-Linked Recording

- AWS services can record configuration data on your behalf without explicit Config recorder setup
- Used by Security Hub, Config Rules across the org
- Reduces setup friction for downstream services needing configuration data

---

## Permissions

Config requires an IAM role with:
- **Read-only permissions** to recorded resources
- **Write access** to the S3 logging bucket
- **Publish access** to SNS

---

## Use Cases

- Audit IAM policies
- Detect if CloudTrail has been disabled
- Detect if EC2 instances use unapproved AMIs
- Detect if security groups are open to the public
- Detect if Internet Gateway is added to an unauthorized VPC
- Detect if EBS volumes are encrypted
- Detect if RDS databases are public

---

## Exam Tips

- Config is the answer for **"track configuration changes over time,"** **"audit compliance against rules,"** **"detect drift from desired configuration"**
- Config is **per-Region** — enable in every Region you want to track
- Use **Aggregator** for org-wide or multi-Region compliance view; aggregator collects, but **rules run in each source account**
- **Conformance Packs** = YAML template bundling Config rules + remediations; deploy as one unit org-wide
- **Remediation** uses **SSM Automation Documents** (AWS-managed or custom)
- **EventBridge** for non-compliance notifications; can route to Lambda, SNS, SQS, Step Functions
- Combine Config + Security Hub: Config detects, Security Hub aggregates findings across the org
- For deploying Config rules to many accounts: use **CloudFormation StackSets with delegated administrator**
- **Conformance Packs are the recommended successor** to AWS Audit Manager (EOL for new customers April 30, 2026)

---

## Exam Traps

- Config is **regional** — to track across all Regions, you must enable Config in each Region and use an Aggregator to consolidate
- Aggregator **only consolidates results** — it does not run rules; rules must be created in each source account
- Config **only detects** non-compliant resources — it does not prevent their creation. For prevention use SCPs, CloudFormation hooks, or Control Tower proactive controls
- Config and CloudTrail serve **different purposes** — CloudTrail logs API calls (who did what); Config tracks resource state and compliance (current configuration and changes)
- Config rules evaluate **after the fact** — SCPs filter API calls before they execute; they are not interchangeable
- Config rule evaluations are **not always real-time** — they happen on a schedule or at config change, which can introduce delay
- Audit Manager is **being EOL'd** for new customers (April 30, 2026) — Conformance Packs is the replacement
