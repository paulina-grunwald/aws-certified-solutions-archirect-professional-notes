# AWS Systems Manager

> **Unified operations service for AWS and hybrid resources. Patch, configure, automate, audit, run commands, store parameters, and remotely access instances — all from one place.**

Maps to: **Domain 1.2 — Prescribe security controls** (operations management)

---

## Overview

- Manage **EC2 instances**, **on-premises servers** (Hybrid Activations), and **edge devices**
- Free for AWS resources; charges may apply for managed nodes outside AWS, Automation, and Session Manager (data transfer)
- Requires **SSM Agent** installed on managed instances (pre-installed on most modern AMIs)
- Instances need an **IAM instance profile** with `AmazonSSMManagedInstanceCore` policy

---

## Capability Categories

### Operations Management
- **Explorer** — unified ops dashboard
- **OpsCenter** — central place to view, investigate, and resolve operational issues
- **CloudWatch Dashboards integration**
- **AppManager** — view resources grouped by application

### Application Management
- **Application Manager** — group resources by application
- **AppConfig** — feature flag and gradual config rollout with validation, monitoring, automatic rollback on alarm
- **Parameter Store** — secure config + secrets (see dedicated note)

### Change Management
- **Change Manager** — pre-approved change templates with approval workflows
- **Automation** — runbooks for routine ops tasks
- **Change Calendar** — time windows when changes are allowed/blocked
- **Maintenance Windows** — scheduled time windows for patches / ops

### Node Management
- **Fleet Manager** — UI for managing all SSM-managed instances
- **Compliance** — patch and config compliance status
- **Inventory** — application and configuration data collected from managed instances
- **Run Command** — execute commands on instances (SSH alternative)
- **Session Manager** — interactive shell access without SSH / bastion
- **Patch Manager** — automate OS and application patching
- **State Manager** — enforce desired state on instances
- **Distributor** — package distribution
- **Hybrid Activations** — manage on-premises servers

### Shared Resources
- **Documents (SSM Documents)** — JSON / YAML scripts; types: Automation, Command, Session, Policy
- **Parameter Store** (Application Management)

---

## Session Manager (heavily tested)

- **Interactive shell access to EC2 / on-prem servers without SSH, bastion, or public IPs**
- Sessions encrypted end-to-end (TLS); auditable in CloudTrail
- Logs to CloudWatch Logs or S3
- Works **through VPC endpoints** (no internet required for instances)
- **No need for SSH key management, bastion hosts, or open inbound ports**
- Cross-account access via IAM
- **Port forwarding** — tunnel to remote services (e.g., RDS, internal HTTPS)

---

## Patch Manager

- Automate **patch deployment** across instances (Linux + Windows + macOS)
- **Patch baselines** — define which patches to install per OS
- **Patch groups** — categorize instances (e.g., dev, test, prod)
- Integrates with **Maintenance Windows** for scheduled patching
- Compliance reporting — see patch state across fleet
- **Quick Setup** — one-click deployment of patching policies org-wide

---

## Automation

- **Runbooks** (Automation Documents) define multi-step workflows
- AWS-managed runbooks for common tasks (restart EC2, snapshot EBS, create AMI, etc.)
- Custom runbooks via JSON / YAML
- Triggers: scheduled, EventBridge events, manual, Config remediation
- Can call Lambda, run scripts, approve via SNS, branch on conditions
- **Common use**: Config rule remediation (e.g., auto-encrypt unencrypted S3 buckets)

---

## Run Command

- Execute commands on managed instances **at scale**
- No SSH / RDP needed
- Targets: instance IDs, tags, or resource groups
- AWS-managed documents for common tasks (install agent, restart service, etc.)
- Integrates with CloudWatch Logs for output

---

## Inventory + Compliance

- Collect installed apps, OS info, network config, custom inventory
- Aggregate org-wide via **Resource Data Sync** → S3 → Athena queries
- **Compliance** view: patch compliance, association compliance, custom compliance

---

## Hybrid Activations (on-premises)

- Manage **non-AWS servers** (on-prem, other clouds) with Systems Manager
- Requires SSM Agent install + activation code
- Counts toward the "managed nodes outside AWS" pricing tier

---

## Quick Setup

- One-click deployment of common SSM configurations org-wide
- Bundles: SSM Agent installation, IAM role setup, Patch Manager baseline, Inventory association
- Use for new account onboarding via AWS Organizations

---

## Exam Tips

- Systems Manager is the answer for: **"patch fleet", "shell access without SSH", "centralized ops", "automation runbooks", "config compliance remediation"**
- **Session Manager** for shell access — no SSH keys, no bastion, no public IPs; logs every command in CloudTrail
- **Patch Manager + Maintenance Windows** for scheduled OS patching across the fleet
- **Automation runbooks** are the answer for **Config rule remediation**
- **AppConfig** for **feature flags with gradual rollout + automatic rollback** (different from Parameter Store)
- **Hybrid Activations** for managing on-prem servers alongside AWS
- **Quick Setup** for org-wide rollout of SSM standards
- Use **SSM VPC endpoints** for private-only Session Manager / Run Command access (no internet needed)
- Run Command for **fleet-wide ad-hoc commands** at scale

---

## Exam Traps

- Session Manager uses **IAM permissions, not SSH keys** — it is a fundamentally different security model from traditional SSH access
- SSM requires **both SSM Agent installed AND an IAM instance profile** — missing either one means the instance cannot be managed
- Parameter Store and AppConfig are **separate SSM features** — Parameter Store is for secrets/config values; AppConfig is for feature flags with gradual rollout/rollback
- Run Command and Automation are **different capabilities** — Run Command is ad-hoc, single-step execution on instances; Automation is multi-step workflows that can include non-EC2 resources
- SSM is **regional** — instances are managed per Region; for cross-Region visibility use Resource Data Sync
- Patch Manager requires **SSM Agent on every instance** — bare metal or unsupported OS instances need alternative solutions
- Session Manager logs **commands but not output by default** — output logging must be explicitly enabled for full audit trails
- Change Manager and AWS Config serve **different purposes** — Change Manager controls who can make changes and when; Config tracks what changed after the fact
