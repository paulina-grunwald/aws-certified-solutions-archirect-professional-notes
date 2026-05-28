# AWS Audit Manager

> **Continuously assess the risk and compliance of your AWS workloads. Audit Manager automates evidence collection across CloudTrail, Config, Security Hub, and License Manager, organizes it against compliance frameworks, and generates audit-ready reports.**

Maps to: **Domain 1.2 — Prescribe security controls** (centralized audit and compliance)

---

> 🔥 **AWS Audit Manager will no longer be open to new customers starting April 30, 2026.** Customers seeking compliance management solutions are encouraged to explore **Conformance Packs in AWS Config**. Conformance Packs provide a means to deploy and monitor detective controls (Config rules) across multiple accounts and regions, with pre-built templates for common frameworks (HIPAA, NIST, PCI-DSS) and custom rule support. Conformance Packs can be managed at the account level or org-wide via AWS Organizations.

---

## Overview

- **Assess the risk and compliance** of your AWS workloads
- **Continuously audit** AWS services usage and prepare audits
- Generates **assessment reports** with evidence folders — for non-compliant items, includes action items to resolve
- Continuous evidence collection, not one-shot
- Existing customers can continue using it; new customers cannot sign up after April 30, 2026

---

## Prebuilt Frameworks

- CIS AWS Foundations Benchmark 1.2.0 & 1.3.0
- General Data Protection Regulation (GDPR)
- Health Insurance Portability and Accountability Act (HIPAA)
- Payment Card Industry Data Security Standard (PCI DSS) V4.0
- Service Organization Control 2 (SOC 2)
- NIST 800-53 (Rev 5)
- FedRAMP Moderate Baseline
- GxP 21 CFR Part 11
- **AWS Generative AI Best Practices Framework v2** — 110 controls for **Amazon Bedrock** and **Amazon SageMaker AI** covering accuracy, fairness, privacy, resilience, responsible AI, safety, security, sustainability

### Custom Frameworks
You can create custom controls and custom frameworks from scratch, or customize existing prebuilt frameworks.

---

## Evidence Finder

Search and filter across thousands of collected evidence items using groupings and filters to identify trends and cross-reference issues.

---

## Evidence Sources

Audit Manager collects automated evidence from four data source types:

- **AWS CloudTrail** — captures user activity logs continuously
- **AWS Config** — compliance check evidence based on Config rule triggers
- **AWS Security Hub** — imports Security Hub CSPM compliance check findings
- **AWS License Manager** — license compliance data

---

## Multi-Account Support

- Assessments can run across multiple accounts in **AWS Organizations**
- Evidence is consolidated into a **delegated administrator** account
- Common pattern: dedicated audit/compliance account as the delegated admin

---

## Common Controls Library

- Pre-defined, pre-mapped reusable controls for GRC (Governance, Risk, Compliance) teams
- Controls automatically inherit improvements as Audit Manager updates
- Eliminates duplicate mapping work across multiple compliance frameworks
- Map enterprise controls **once**, reuse across frameworks

---

## Exam Tips

- Audit Manager is being **sunset for new customers** on April 30, 2026 — the recommended replacement is **AWS Config Conformance Packs**
- Know the evidence source chain: Audit Manager pulls from **CloudTrail** (user activity), **Config** (resource compliance), **Security Hub** (security findings), and **License Manager** (license compliance)
- Audit Manager provides the **third-line assurance** function (independent audit), while Config Conformance Packs provide **first-line** (risk management)
- Multi-account auditing uses **AWS Organizations** with a **delegated administrator** — a common exam pattern
- Audit Manager generates **assessment reports** with evidence folders — it does not remediate; it only collects and organizes evidence
- Conformance Packs do NOT currently support all frameworks Audit Manager covers (e.g., no SOC 2 or GDPR templates in Conformance Packs)
- **Generative AI Best Practices Framework v2** is the answer for "audit Bedrock or SageMaker AI workloads against responsible-AI controls"
- **Common Controls Library** is the answer when scenario says "map enterprise controls once and reuse across multiple compliance frameworks"

---

## Exam Traps

- Audit Manager **aggregates evidence** from Config, CloudTrail, Security Hub, and License Manager into audit-ready reports — Config alone only evaluates resource compliance against rules
- AWS Artifact provides **AWS's own** compliance reports/certifications (SOC, ISO) — Audit Manager helps you audit your workloads, not AWS's
- Audit Manager is **evidence collection only** — it does not remediate. For auto-remediation, use Config rules with remediation actions or Systems Manager
- Conformance Packs collect **less exhaustive evidence** (only Config rule evaluations, no CloudTrail logs or Security Hub findings) — they are not a full replacement despite being the recommended successor
- Security Hub is a **security findings aggregator** (CSPM) — Audit Manager packages evidence for formal audits. They serve different purposes
- Audit Manager is **sunset for new customers** after April 30, 2026 — don't pick it as the answer for a brand-new compliance program; use Conformance Packs instead
