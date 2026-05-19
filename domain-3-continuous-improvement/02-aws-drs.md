# AWS Elastic Disaster Recovery (DRS)

> **Agent-based, continuous block-level replication for server-based workloads. Replicates source servers (on-prem, other clouds, or EC2) into a lightweight staging area in AWS. RPO seconds, RTO minutes. Pilot Light internally. Supports non-disruptive recovery drills and failback. Replaces CloudEndure Disaster Recovery (CloudEndure DR EOL).**

Maps to: **Domain 1.3 — Reliable / resilient**, **Domain 2.2 — Business continuity**, **Domain 4.2 — Migration**

---

## Overview

- For DR solutions with **RPO in seconds** and **RTO in minutes**
- Continuous **block-level replication** keeps data updated in near real-time → RPO of seconds
- On disaster: launches **fully provisioned EC2 instances** in target Region → RTO of minutes (driven mostly by OS boot time)
- **Pilot Light** strategy internally — staging instances are cheap; on failover, full-sized targets launch
- **CloudEndure Disaster Recovery is now AWS DRS** (CloudEndure EOL)

---

## How It Works

```
Source server (on-prem / other cloud / EC2)
   │ AWS Replication Agent (installed on source)
   ▼
Continuous block-level replication (TLS)
   ▼
Staging area in AWS Region
  ├─ Lightweight replication servers (T3 / similar)
  ├─ Replicated volumes on EBS
  └─ Snapshots for point-in-time recovery
   │
   │ (on failover or drill)
   ▼
Fully provisioned EC2 instances in target Region
```

---

## Key Capabilities

- **Continuous replication** — no scheduled snapshots; data flows continuously
- **Point-in-time recovery** — choose any point within retention window for failover
- **Non-disruptive recovery drills** — launch test instances without affecting production
- **Failback** — replicate back to source after primary is restored
- **Cross-Region** and **cross-account** target options
- **Custom launch settings** — instance type, SG, subnet, IAM role at launch time
- **Replication agent** for Linux + Windows
- **Post-launch automation** — run SSM documents after instance boot

---

## What DRS Replicates (and Doesn't)

| Replicated | NOT Replicated |
|---|---|
| EC2 / on-prem / other-cloud servers | RDS / Aurora (use service-native replication) |
| EBS volumes | DynamoDB (use Global Tables) |
| Linux + Windows OS, applications, files | S3 (use S3 CRR) |
| Full OS state | Lambda, Fargate, managed services |

> DRS is for **server-hosted workloads only**. Managed services need their own replication.

---

## Source Types Supported

- **Physical servers** (bare metal) on-premises
- **Virtual machines** (VMware, Hyper-V, KVM)
- **Cloud workloads** from Azure, GCP, OCI
- **EC2 instances** (cross-Region or cross-account DR)

---

## Pricing

- Pay per **source server per hour** of replication
- Plus storage cost for replicated EBS volumes in staging
- **No DR-Region compute cost** until you launch recovery / drill instances
- Significantly cheaper than running full warm-standby infrastructure

---

## DRS vs Application Migration Service (MGN)

| Feature | **AWS DRS** | **AWS MGN (Application Migration Service)** |
|---|---|---|
| Purpose | Disaster Recovery | Lift-and-shift migration |
| Replication | Continuous | Continuous |
| Cutover | Drill anytime + failover on disaster | One-time cutover to AWS |
| Use case | Ongoing DR protection | Migrate to AWS, then decommission source |
| Lifecycle | Long-term | Short-term until cutover |

> Same underlying replication tech (both descend from CloudEndure); DRS = DR product, MGN = migration product. Choose by intent.

---

## Recovery Workflow

1. Install **AWS Replication Agent** on source servers
2. Configure **replication settings** (target subnet, EBS type, encryption, bandwidth throttle)
3. **Continuous replication** to staging area in DR Region
4. (Periodically) run **recovery drills** to validate the workflow
5. On disaster, initiate **recovery** — DRS launches EC2 in target Region using launch template
6. Verify, switch DNS / traffic (Route 53, Global Accelerator)
7. After primary is restored, configure **failback** replication

---

## Use Cases

- **DR for on-premises servers** — VMware / physical → AWS
- **DR for AWS workloads** — cross-Region EC2 DR (instead of custom AMI replication)
- **Cross-cloud DR** — Azure / GCP / OCI workloads protected to AWS
- **Compliance** — meet regulatory DR requirements with verifiable RPO / RTO
- **Migration via DR** — replicate via DRS, run drills, "fail over" as a planned cutover (or use MGN instead)

---

## Exam Tips

- **"RPO in seconds + RTO in minutes for server workloads"** → **AWS DRS**
- **"Replace CloudEndure Disaster Recovery"** → **AWS DRS** (CloudEndure DR is EOL)
- **"DR for on-prem servers to AWS"** → **AWS DRS**
- **"Cross-Region EC2 DR with continuous replication"** → **AWS DRS**
- **"Non-disruptive DR test"** → **DRS recovery drills**
- **"Failback after primary is restored"** → DRS supports it
- **"Lift-and-shift migration to AWS"** → **AWS MGN** (Application Migration Service), not DRS — same tech, different intent
- **"Custom scripts after recovery instance boots"** → DRS **post-launch automation** with SSM
- DRS pricing: cheap staging (replication servers + EBS) — compute only when you launch recovery instances

---

## Exam Traps

- **DRS does NOT replicate managed services** — RDS / Aurora / DynamoDB / S3 / Lambda need their own replication (Aurora Global DB, DynamoDB Global Tables, S3 CRR)
- **DRS ≠ MGN** — same tech, different lifecycle. MGN = one-time cutover (then decommission); DRS = ongoing DR
- **CloudEndure Disaster Recovery is EOL** — never the right answer; use AWS DRS
- **DRS requires an agent on the source** — agentless replication is NOT supported
- **DRS does NOT replicate to a "hot" running instance** — it stages replicated volumes; recovery launches new instances (Pilot Light pattern)
- **RTO depends on OS boot time** — claims of "instant" RTO are wrong; expect minutes for OS boot + app startup
- **DRS supports cross-account** for target — but you must set up cross-account roles
- **Network bandwidth between source and AWS matters** — initial sync can take days for TB-scale source servers
