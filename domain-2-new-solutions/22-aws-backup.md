# AWS Backup

> **Centralized, fully managed backup service. Single place to configure, schedule, audit, and monitor backups across AWS storage, databases, file systems, and select hybrid workloads.**

Maps to: **Domain 2.2 — Design solutions for business continuity** (backup & restore strategies)

---

## Overview

- **Centralized** — one service, one set of policies, many AWS storage / database services
- **Backup Plans** — policy expressions defining when and how to back up
- **Cross-Region** and **cross-account** backups supported
- **PITR (Point-in-Time Recovery)** for supported services (RDS non-Aurora, Aurora, DynamoDB, S3)
- **On-demand** and **scheduled** backups
- **Tag-based resource selection** for backup plans (back up everything with tag `Backup=true`)
- **KMS-integrated encryption** for backup vaults
- Helps meet compliance: PCI, HIPAA, SOC, FedRAMP, etc.

---

## Supported AWS Services

### Storage and file systems
- **Amazon EBS** (snapshots)
- **Amazon EFS**
- **Amazon FSx** (Windows File Server, Lustre, NetApp ONTAP, OpenZFS)
- **AWS Storage Gateway** (Volume Gateway snapshots)
- **Amazon S3**

### Databases
- **Amazon RDS** (all engines, including continuous backup with PITR)
- **Amazon Aurora** (continuous backup)
- **Amazon DynamoDB** (snapshot + PITR via continuous backup)
- **Amazon DocumentDB**
- **Amazon Neptune**
- **Amazon Timestream**
- **Amazon Redshift** (provisioned clusters)
- **SAP HANA on EC2**

### Compute
- **Amazon EC2** (instance-level backup via AMIs + EBS snapshots)

### Hybrid / Third-party
- **VMware workloads** (on-premises + VMware Cloud on AWS)
- **AWS Outposts** resources

---

## Backup Plans

A backup plan defines **when** and **how** AWS Backup backs up resources.

### Components
- **Backup Rule(s)**:
  - **Frequency**: cron expression, or every 12 hours / daily / weekly / monthly
  - **Backup window**: when the backup can start (preferred window)
  - **Lifecycle**: optional transition to **cold storage** (days / months); only EFS and Storage Gateway support cold storage transitions
  - **Retention**: how long to keep the backup (days / weeks / months / years / forever)
  - **Copy actions**: optionally copy to another Region and / or another account
  - **Vault**: which backup vault stores the recovery points
- **Resource assignments**:
  - By **resource ID**
  - By **tag** (e.g., `Backup=true`, `Environment=prod`) — the most scalable approach
  - By **service type**

> 💡 **Tag-based selection** is the exam-recommended approach for backing up many resources with consistent policies.

---

## Continuous Backup vs Snapshot Backup

### Snapshot backup (default for most services)
- Point-in-time snapshot taken at scheduled intervals
- Restore to the snapshot time (not in between)
- All AWS Backup-supported services support snapshot backups

### Continuous backup (PITR)
- Captures every change continuously (write-ahead log shipping)
- Restore to **any second** within the retention window (typically last 35 days)
- Supported for: **Amazon RDS (non-Aurora), Aurora, DynamoDB, S3**
- Required for sub-day RPO scenarios

### Incremental backups
- All services EXCEPT: **DynamoDB, Aurora, DocumentDB, Neptune** (those take full snapshots each time)
- Reduces storage cost dramatically over time

---

## Backup Vault Lock (WORM)

Enforce **Write Once, Read Many (WORM)** on backups stored in a vault.

### Two retention modes (similar to S3 Object Lock)
- **Governance mode** — IAM principals with `backup:DisableBackupVaultLockCompliance` can remove the lock; useful for non-regulated buckets
- **Compliance mode** — **no one (including root) can delete backups or shorten retention** until the retention period expires. Locked state is immutable after the cooling-off period.

### Cooling-off period
- After enabling lock, there is a **grace period** (3 to 72 hours; default 3 days = 72 hr) during which you can disable the lock
- After cooling-off ends, **lock becomes permanent** for compliance mode

### Use cases
- Protection against **ransomware** (malicious deletes)
- Compliance: SEC Rule 17a-4(f), FINRA, HIPAA, financial archives
- Insider threat protection (rogue admin scenarios)

> ⚠️ **Exam pattern**: scenario says "immutable backup that cannot be deleted by anyone, even with root credentials" → **Backup Vault Lock in Compliance mode**.

---

## Logically Air-Gapped Vault

**Ransomware-resistant backup vault** — stores immutable backup copies in an **AWS-owned account**, isolated from your account.

### Properties
- Backups are **immutable** (cannot be modified or deleted before retention expires)
- **Vault Lock automatically enforced** in compliance mode
- Encrypted with AWS-owned keys by default; customer-managed KMS keys also supported
- **Cross-account / cross-Region restore** without restoring to source account first
- **Direct backup** to air-gapped vault supported (previously had to copy from a regular vault first)

### Supported services (growing list)
EBS, RDS, DynamoDB, EFS, Aurora, S3, FSx

### Use cases
- **Ransomware recovery** — attackers in your account cannot delete or encrypt the backups
- **Insider threat** protection
- **Compliance** requiring isolation between primary data and backup

> 💡 **Exam pattern**: scenario says "protect backups from ransomware attack that compromises the AWS account" → **AWS Backup logically air-gapped vault**.

---

## Restore Testing

**Automated validation that your backups actually work** — schedules test restores on a recurring basis to validate recoverability.

### Features
- Define **restore testing plans** with frequency (daily, weekly, monthly)
- Pick a **subset of recovery points** (latest, random within window, by tag)
- Restores to a **test environment**; validates restore succeeds
- **Auto-cleanup** of test resources after validation (configurable retention)
- Optional **resource-specific validation** scripts via Lambda
- Supports **cross-account** restore testing, including from logically air-gapped vaults
- Generates **CloudWatch metrics** and **CloudTrail events** for test outcomes

**Supported types**: EC2, EBS, RDS, Aurora, S3, DynamoDB, EFS, FSx (varies by Region)

**Why it matters for the exam**: a backup is only as good as a successful restore. Restore testing is the answer when scenario requires "verify backups are actually recoverable on an ongoing basis."

---

## AWS Backup Audit Manager

Tracks backup activity, compliance, and policy adherence.

### Features
- **Frameworks** — pre-built (e.g., backup frequency, retention, encryption) or custom controls
- **Reports** — daily / weekly compliance reports delivered to S3
- **Alerts** — flag non-compliant resources (e.g., resources missing backup plans, unencrypted backups)
- Integrates with **AWS Config** under the hood
- Useful for: regulatory audit evidence, internal compliance posture

> 💡 Often paired with **AWS Audit Manager** (a different service): Backup Audit Manager tracks backup-specific compliance; AWS Audit Manager tracks broader compliance frameworks.

---

## AWS Organizations Integration + Backup Policies

**Centralized backup management** across an AWS Organization.

- **Backup policies** — Organizations management policy type (like SCPs but for backups)
- Define backup plans **at the OU or account level**; child OUs / accounts inherit
- Plans **merge** (additive) when inherited (unlike SCPs which intersect)
- **Delegated administrator** — designate a member account as backup admin so the management account doesn't run workloads

### Pattern
1. Enable AWS Backup in management account
2. Enable cross-account management feature
3. Create org-wide backup policy
4. Apply to OUs / accounts
5. Cross-account copy to centralized backup account (often paired with logically air-gapped vault)

**Use case**: enforce minimum backup hygiene (daily backups, 30-day retention, encryption) across every account in the org.

---

## Cross-Region and Cross-Account Backup

### Cross-Region
- Backup plan can include a **copy action** to another Region
- Use for **disaster recovery** — primary Region failure
- Charges for storage in destination Region + data transfer

### Cross-Account
- Copy backups to a vault in a **different AWS account**
- Common pattern: centralized backup account, isolated from workload accounts
- Workload account can't delete backups in the central account (defense in depth)
- Often combined with **logically air-gapped vault** in the central account for maximum protection

Both can be combined: copy backup → another Region AND another account simultaneously.

---

## Encryption

- **Vault-level encryption** — each backup vault has a KMS key (AWS-managed or customer-managed)
- Backup data is **always encrypted at rest** with the vault's key
- **Cross-Region / cross-account copies** can use a different KMS key in the destination
- **Logically air-gapped vaults**: AWS-owned key by default; customer-managed KMS keys also supported
- Encryption keys are NOT replicated automatically — you must grant the destination key access

---

## Service-Specific Notes

- **DynamoDB, Aurora, DocumentDB, Neptune** — full snapshots (no incremental); cost more for frequent backups
- **DynamoDB** — PITR is separate from AWS Backup; AWS Backup can leverage PITR for granular restore
- **Aurora** — continuous backup (PITR) is native; AWS Backup integrates with it
- **RDS (non-Aurora)** — supports both snapshot and continuous backup via AWS Backup
- **EFS** — only service supporting **item-level restore** via AWS Backup (restore individual files)
- **EFS, Storage Gateway** — only services supporting **cold storage lifecycle** transition
- **S3** — backup includes object versions; supports continuous backup (PITR) for the bucket
- **FSx** — backup is a point-in-time snapshot; restore creates a new file system

---

## Pricing

- **Per-GB-month** for backup storage (varies by service and storage class)
- **Cold storage** (EFS, Storage Gateway only) is cheaper per GB
- **Restore** charges per GB for some services (e.g., DynamoDB)
- **Cross-Region copy** charges for inter-Region data transfer
- **Cross-account copy** is free for transfer (storage charged in destination account)
- **Logically air-gapped vault** charged separately (premium tier)
- **Restore testing** uses standard backup storage + restore charges

---

## AWS Backup vs Native Service Backups

When to use AWS Backup vs native (e.g., RDS automated backups, EBS snapshot via Data Lifecycle Manager)?

**AWS Backup wins when**:
- Multiple services need consistent backup policy
- Cross-account / cross-Region centralized backup pattern
- WORM / immutability (Vault Lock, logically air-gapped vault)
- Compliance audit (Backup Audit Manager)
- Tag-based resource selection across accounts
- Restore testing automation

**Native wins when**:
- Single-service, simple use case
- You need service-specific features not exposed in AWS Backup (rare)
- Cost minimization — native may be slightly cheaper for single service

---

## Common Architecture Patterns

### Pattern 1: Centralized backup account
- Workload accounts: backup plans copy to central account vault
- Central account: logically air-gapped vault with Vault Lock in compliance mode
- Restore: from central account back to workload account or to a forensics account

### Pattern 2: Multi-Region DR
- Backup plan in primary Region with copy-to-secondary-Region
- On Region failure, restore in secondary Region
- RPO = backup frequency (e.g., 1 hour for hourly snapshots)

### Pattern 3: Ransomware protection
- Primary backup → regular vault (frequent, low cost)
- Daily / weekly copy → logically air-gapped vault (immutable)
- Restore testing weekly to validate recoverability

### Pattern 4: Compliance
- Backup policies at Organizations level enforce minimum hygiene
- Backup Audit Manager generates evidence for auditors
- Vault Lock compliance mode for required retention periods

---

## Exam Tips

- **AWS Backup** is the answer for "centralized backup across many services with consistent policy"
- **Backup Plans** with **tag-based resource selection** scale better than resource IDs
- **Backup Vault Lock Compliance mode** is the answer for "immutable backup that even root cannot delete"
- **Logically air-gapped vault** is the answer for "protect backups from ransomware / account compromise"
- **Restore Testing** is the answer for "verify backups are actually recoverable on an ongoing basis"
- **AWS Backup Audit Manager** is the answer for "compliance evidence about backup coverage"
- **AWS Organizations backup policies** for org-wide enforcement (additive across OUs, unlike SCPs which intersect)
- **Cross-Region + cross-account copies** for defense in depth
- **Continuous backup / PITR** for sub-day RPO (RDS non-Aurora, Aurora, DynamoDB, S3)
- **EFS is the only service** supporting **item-level restore**
- **Cold storage lifecycle** is only for EFS and Storage Gateway
- **DynamoDB / Aurora / DocumentDB / Neptune** take full snapshots (no incremental) — budget accordingly
- **VMware on-premises** workloads can be backed up to AWS Backup
- For **Bedrock / SageMaker workload backup**: data is in S3 → use AWS Backup for S3
- Backup Vault Lock has a **cooling-off period** (3-72 hours) before lock becomes permanent in compliance mode

---

## Exam Traps

- Backup Vault Lock and S3 Object Lock are **separate services with separate APIs** — they share similar WORM concepts (Governance / Compliance modes) but are not interchangeable
- Logically air-gapped vaults are stored in an **AWS-owned account** — they are not in your account; the isolation is what provides ransomware protection
- Backup Vault Lock Governance mode is **not truly immutable** — users with the right IAM permission can remove the lock; only Compliance mode is truly immutable
- Cold storage transitions are supported **only for EFS and Storage Gateway** — other services do not support cold storage lifecycle in AWS Backup
- Item-level restore is supported **only for EFS** — other services restore at the full resource level
- Continuous backup (PITR) is supported **only for RDS (non-Aurora), Aurora, DynamoDB, and S3** — not all services
- AWS Backup **centralizes management but does not replace native service backups** — native backups still work independently
- Backup policies and SCPs are **different Organizations policy types** — backup policies merge additively across OUs, while SCPs intersect
- Cross-account copy uses **AWS Backup APIs with no networking changes** — VPC peering or Transit Gateway is not required
- Restore testing must be **explicitly configured** — you must define and enable restore testing plans; it is not automatic for all backups
- Vault Lock compliance mode is **permanent after the cooling-off period** (3-72 hours) — it cannot be removed or shortened once locked
- Backup vaults are **Region-scoped** — for cross-Region protection you must copy to a vault in the destination Region
- AWS Backup Audit Manager and AWS Audit Manager are **different services** — Backup Audit Manager is specific to backup compliance; AWS Audit Manager covers broader frameworks (and is being EOL'd for new customers April 30, 2026)
- Mixing services in a single backup plan is allowed, but **service-specific options apply only where supported** — cold storage, PITR, and item-level restore only work for their respective services
- Logically air-gapped vaults are a **premium tier with separate pricing** — they are not free

---

## Quick Reference

| Need | Feature |
|---|---|
| Centralized backup across services | AWS Backup |
| Policy-driven backup at scale | Backup Plans + tag-based selection |
| Immutable backup, governance mode | Backup Vault Lock (Governance) |
| Truly immutable backup (even root can't delete) | Backup Vault Lock (Compliance) |
| Ransomware-resistant backup | Logically air-gapped vault |
| Validate backups recoverable | Restore Testing |
| Compliance evidence | AWS Backup Audit Manager |
| Org-wide policy enforcement | Backup policies in AWS Organizations |
| Cross-Region DR | Cross-Region copy action |
| Cross-account isolation | Cross-account copy to backup account |
| Sub-day RPO | Continuous backup (PITR) — RDS / Aurora / DynamoDB / S3 only |
| Restore single file | EFS item-level restore (only service supporting this) |
| Cold storage backup tier | EFS, Storage Gateway only |
| VMware / SAP HANA backup | AWS Backup with respective integration |
