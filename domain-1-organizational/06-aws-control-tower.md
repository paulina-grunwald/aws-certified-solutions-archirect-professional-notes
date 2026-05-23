# AWS Control Tower

> **Easy way to set up and govern a secure, compliant multi-account AWS environment based on best practices. Runs on top of AWS Organizations.**

Maps to: **Domain 1.4 — Design a multi-account AWS environment**

---

## Overview

- Runs on top of **AWS Organizations** — automatically sets up Organizations to organize accounts and implement SCPs
- Automates setup of well-architected multi-account environment in a few clicks
- Automates ongoing policy management using **controls** (formerly called guardrails)
- Provides a dashboard for visibility into compliance status across all accounts

---

## Landing Zone

The well-architected, multi-account environment that Control Tower sets up for you.

### Landing Zone 4.0
- **Optional Service Integrations** — choose which integrations to enable (AWS Config, CloudTrail, SecurityRoles, AWS Backup); can create a landing zone with no integrations at all
- **Dedicated Resources** — AWS Config and CloudTrail use separate dedicated S3 buckets and SNS topics instead of shared resources
- **Flexible Organization Structure** — removes previous mandatory OU structure requirements
- **ConfigBaseline** — new baseline type for detective controls support without requiring the full AWSControlTowerBaseline

Available in **European Sovereign Cloud** (Jan 2026).

---

## Control Only Experience

- Enables faster governance setup for **established multi-account environments**
- Provides access to AWS managed controls **without requiring a full landing zone** deployment
- Ideal when you already have a mature Organizations setup and just need controls / guardrails

---

## Account Factory

- Automates account provisioning and deployments
- Enables you to create pre-approved baselines and configuration options for AWS accounts in your organization (e.g., VPC default configuration, subnets, region)
- Uses **AWS Service Catalog** to provision new AWS accounts

### Account Factory Customization (AFC)
- Define account blueprints (as Service Catalog products) to customize accounts during provisioning
- Blueprints stored in a hub account

### Account Factory for Terraform (AFT)
- Account provisioning and customization using **Terraform**
- Uses a pipeline with CodeCommit, CodeBuild, and CodePipeline under the hood

Supports **automatic enrollment** of accounts when moved into a new OU (LZ 3.1+).

---

## Controls (formerly Guardrails)

Provide ongoing governance for AWS accounts managed by Control Tower.

### Three behavior types
- **Preventive** — using SCPs, RCPs, and declarative policies (e.g., Disallow Creation of Access Keys for the Root User). Status: *enforced* or *not enabled*
- **Detective** — using AWS Config rules (e.g., Detect if MFA for root user is enabled). Status: *clear*, *in violation*, or *not enabled*
- **Proactive** — using AWS CloudFormation hooks; scans resources **before provisioning** and blocks non-compliant resources. Status: *PASS*, *FAIL*, or *SKIP*

### Three levels
- **Mandatory** — automatically enabled and enforced by Control Tower (e.g., disallow public read access for log archive account)
- **Strongly Recommended** — based on AWS best practices (optional; e.g., enable encryption for EBS volumes attached to EC2)
- **Elective** — commonly used by enterprises (optional; e.g., disallow delete actions without MFA in S3 buckets)

**Best practice**: use all three behavior types for **defense-in-depth** (preventive for baselines, proactive for pre-deployment, detective for continuous monitoring).

---

## Exam Tips

- Control Tower runs **on top of AWS Organizations** — if a question mentions multi-account governance with automated setup, think Control Tower first
- Know the three control types: **Preventive** (SCPs — block actions), **Detective** (Config rules — detect violations), **Proactive** (CloudFormation hooks — prevent non-compliant resource creation)
- **Account Factory** uses **AWS Service Catalog** under the hood; **AFT** uses **Terraform** — know which to pick based on IaC tooling in the scenario
- Control Tower creates a **Log Archive** account and an **Audit** account automatically — these are shared accounts
- **Control Only Experience** = you already have Organizations set up and just want governance controls without a full landing zone
- Landing Zone 4.0 lets you **opt out of service integrations** (Config, CloudTrail, etc.) — useful for orgs that manage these independently
- Control Tower is **regional** — it operates in supported AWS Regions only
- If the question mentions **drift detection**, Control Tower detects drift when changes are made to member accounts outside of Control Tower (e.g., directly editing SCPs in Organizations)

---

## Exam Traps

- Control Tower **builds on top of Organizations** — it adds automated landing zone setup, controls, and Account Factory; Organizations is the underlying service, not a synonym
- Preventive and proactive controls are **different mechanisms** — Preventive controls (SCPs) block API actions; Proactive controls (CloudFormation hooks) block non-compliant resource configurations before provisioning
- "Guardrails" and "Controls" are **the same thing** — AWS renamed guardrails to controls; both terms may appear on the exam
- Control Tower **uses Config rules for detective controls but is not a replacement for Config** — it orchestrates Config across accounts as part of its governance model
- AFC and AFT use **different IaC tooling** — AFC uses Service Catalog blueprints for account customization; AFT uses Terraform. Pick based on the IaC preference in the scenario
