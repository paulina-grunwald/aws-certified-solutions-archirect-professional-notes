# AWS Artifact

> **Self-service portal for AWS compliance documentation and agreements. Download AWS's SOC, ISO, PCI, FedRAMP reports and accept BAA / HIPAA / Customer Account Addenda. Free, console-based.**

Maps to: **Domain 1.2 — Prescribe security controls** (compliance documentation)

---

## Overview

**AWS Artifact** is the canonical answer for:
- "Where do I download AWS's **SOC reports / ISO certs / PCI certs / HIPAA BAAs**?"
- "How do I prove AWS's compliance posture to **my regulators or auditors**?"
- "How do I accept industry-specific agreements (BAA, HIPAA) with AWS?"

Available in the AWS Console. **Free** of charge.

---

## Two Main Areas

### Artifact Reports
On-demand download of AWS's third-party audit reports and certifications:
- **ISO 9001 / 27001 / 27017 / 27018 / 27701**
- **SOC 1, SOC 2, SOC 3** reports
- **PCI DSS Attestation of Compliance (AoC)**
- **FedRAMP**, **DoD CC SRG**, **CCCS**, **C5**, **ENS**, **IRAP**, **K-ISMS**, **MTCS**, **OSPAR**
- **HIPAA, HITRUST** documentation

> Documents are **access-controlled** via IAM. Some require accepting an NDA before download.

### Artifact Agreements
Review, accept, and track AWS agreements for your account OR your AWS Organization:
- **Business Associate Addendum (BAA)** — required for HIPAA workloads
- **Customer Account Addendum** for various international agreements

> For AWS Organizations: management account can accept on behalf of all member accounts. Withdraw when no longer needed.

---

## Third-Party Reports

- AWS Artifact now hosts reports from **AWS Partner Network (APN) partners** with software running on AWS — independent ISV audit reports
- Provides **end-to-end** compliance evidence (AWS infrastructure + partner workloads) in one place
- Simplifies due diligence for SaaS purchases

---

## Use Cases

- **Compliance demonstration** to regulators / auditors / security teams
- **Internal audit** — evaluate your cloud architecture against industry frameworks
- **Vendor risk assessment** — share AWS's SOC 2 / ISO certs with clients evaluating your platform
- **HIPAA workload provisioning** — accept the BAA before storing PHI in AWS
- **PCI scope reduction** — leverage AWS's PCI AoC for the underlying infrastructure

---

## Differences From Related Services

| Service | Role |
|---|---|
| **AWS Artifact** | Download AWS's **third-party audit reports** and accept AWS agreements |
| **AWS Audit Manager** | Generate **your own audit evidence packages** for compliance frameworks (PCI / SOC / HIPAA) using your AWS resources |
| **AWS Security Hub** | Run **continuous control checks** against standards (FSBP, CIS, PCI, NIST) on your AWS resources |
| **AWS Config** | Track **resource configuration history** and evaluate against rules |

> Artifact = AWS's compliance proof to **you**.
> Audit Manager = **your** compliance proof to your auditor.

---

## Access Control

- IAM permissions: `artifact:Get*`, `artifact:DownloadAgreement`, `artifact:AcceptAgreement`, etc.
- Best practice: restrict downloads to specific groups (security / compliance teams)
- All downloads are logged in **CloudTrail**

---

## Exam Tips

- **AWS Artifact = AWS's compliance documentation portal** (SOC, ISO, PCI, FedRAMP, HIPAA BAA, ...)
- **Free**, console-based
- **HIPAA workloads** require accepting the **BAA via Artifact Agreements** before storing PHI
- **PCI compliance**: download AWS's **PCI DSS AoC** to demonstrate the underlying infrastructure is compliant
- **Third-party reports** from partners — useful for SaaS due diligence
- **Restrict downloads via IAM** — sensitive documents often under NDA
- **CloudTrail records all Artifact downloads** — useful for compliance audit trails
- For **your own evidence packages** for an external audit, use **Audit Manager** (not Artifact)

---

## Exam Traps

- **Artifact ≠ Audit Manager**. Artifact = AWS's compliance docs to you. Audit Manager = your evidence packages to your auditor
- **Artifact ≠ Security Hub**. Security Hub runs continuous control checks; Artifact just hosts AWS's compliance documents
- **Some Artifact documents require an NDA** — pre-accept before download
- **Artifact Agreements are per-account OR per-Organization** — if managing multiple accounts, accept org-wide from the management account
- **Artifact does NOT make YOUR workload compliant** — it provides evidence of AWS's underlying infrastructure compliance; you must still design, configure, and document your workload's compliance
- **HIPAA in AWS requires both**: (1) accept the BAA via Artifact, AND (2) only use HIPAA-eligible services for PHI
