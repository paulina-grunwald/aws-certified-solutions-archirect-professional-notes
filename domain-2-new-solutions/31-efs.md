# Amazon EFS (Elastic File System)

> **Fully managed, scalable, elastic NFS v4.1 file system. Multi-AZ by default (Standard). Auto-scales to petabytes. Mount on many EC2 / Lambda / ECS / EKS / Fargate simultaneously. Storage classes: Standard, Standard-IA, One Zone, One Zone-IA. Intelligent-Tiering with lifecycle policies. Cross-Region replication. Access Points for app-specific entry. Encrypted at rest + in transit. POSIX permissions.**

Maps to: **Domain 2.5 — Storage selection**, **Domain 1.3 — Reliability**, **Domain 2.4 — Shared state**

---

## Overview

- **NFS v4.1** managed file system
- **Linux only** (POSIX-compatible)
- **Auto-scales** to petabytes; pay per GB stored
- **Multi-AZ by default** (Standard storage classes) — accessible from all subnets in the VPC
- **Thousands of concurrent NFS connections**
- **Read-after-write consistency**
- Mount targets (ENIs) — **one per AZ** in the VPC
- **Security groups** control access; **IAM** + POSIX permissions

---

## Storage Classes

| Class | AZ scope | Use case |
|---|---|---|
| **EFS Standard** | Multi-AZ | Hot data; default |
| **EFS Standard-IA** | Multi-AZ | Infrequently accessed; ~92% cheaper than Standard |
| **EFS One Zone** | Single-AZ | Lower cost; less resilient |
| **EFS One Zone-IA** | Single-AZ | Lowest cost; less resilient |

> Files < 128 KiB are NOT eligible for IA transitions.

### Lifecycle Management + Intelligent-Tiering

- **Lifecycle policy** ages files into IA after configurable days (7, 14, 30, 60, 90)
- **EFS Intelligent-Tiering** — auto-tier between Standard ↔ IA based on access pattern
- IA storage savings up to **92%** vs Standard
- Files transition back to Standard on access (no unbounded IA charges)

---

## Performance Modes

| Mode | Use case |
|---|---|
| **General Purpose** (default) | Latency-sensitive (web servers, CMS) |
| **Max I/O** | Massively parallel I/O (big data, media processing); higher latency per op |

> Max I/O is harder to back out of — choose carefully at creation.

---

## Throughput Modes

| Mode | Behavior |
|---|---|
| **Elastic** (default) | Auto-scales to workload; pay per usage |
| **Bursting** | Throughput scales with storage size; burst credits |
| **Provisioned** | Set throughput independently of size |

> **Elastic Throughput** is the new default — simpler, no burst credit management.

---

## Access Points

- **Application-specific entry point** to an EFS file system
- Enforces:
  - **POSIX user/group** for all access through the AP
  - **Root directory** (mounts to a subdir, not file system root)
- Use case: multi-tenant apps where each tenant gets its own AP with its own POSIX identity + path

---

## Cross-Region Replication

- **Replicate EFS** to another Region for DR
- **One-way replication**; promote replica for failover
- **RPO < 1 minute** typical
- Supports all storage classes
- **Async** replication

---

## Encryption

- **At rest** (KMS) — enabled at file system creation; cannot change later
- **In transit** (TLS) via the EFS mount helper (`amazon-efs-utils`)

---

## Backup

- **Automatic backups** via AWS Backup (default ON for new file systems)
- **One Zone file systems** are auto-backed up by default
- Configurable schedule, retention, cross-Region copy

---

## Access Patterns

### EC2

- Install **NFS client** (or `amazon-efs-utils`)
- Mount via DNS: `<file-system-id>.efs.<region>.amazonaws.com:/`

### Lambda

- Mount via **Access Point** + VPC integration
- Files persist across invocations
- **Not compatible with SnapStart**

### ECS / Fargate

- Mount in task definition as a volume

### EKS

- **EFS CSI driver** + Persistent Volume

### On-Premises

- Mount over **Direct Connect** or **VPN**
- Use **DataSync** for one-time / scheduled bulk transfers

---

## EFS vs EBS vs S3 vs FSx

| Feature | **EFS** | **EBS** | **S3** | **FSx** |
|---|---|---|---|---|
| Protocol | NFS (Linux) | Block | HTTPS object | NFS / SMB / Lustre |
| Concurrent mounts | Many | 1 (or 16 multi-attach io1/io2) | API | Many |
| Multi-AZ | Yes (Standard) | No (AZ-scoped) | Yes | Some variants |
| Use case | Shared Linux files | Boot/data volumes | Objects, web hosting | Windows shares, HPC, NetApp ONTAP, OpenZFS |

---

## Monitoring

- **CloudWatch metrics**: `TotalIOBytes`, `ReadIOBytes`, `WriteIOBytes`, `MetadataIOBytes`, `ClientConnections`, `PercentIOLimit`, `BurstCreditBalance`
- **CloudTrail** for control plane

---

## Exam Tips

- "Shared Linux file system across many EC2/containers" → **EFS**
- "POSIX-compliant, NFS-mountable, multi-AZ" → **EFS**
- **Linux-only** — for Windows shares use **FSx for Windows File Server**
- **Multi-AZ by default** (Standard) — automatically replicated across AZs
- **Storage classes**: Standard / Standard-IA / One Zone / One Zone-IA
- **Intelligent-Tiering** for automatic cost optimization
- **Elastic Throughput** is the modern default
- **Access Points** for app-specific entry + POSIX enforcement
- **Cross-Region replication** for DR
- **EFS + Lambda** via Access Points (VPC required; not compatible with SnapStart)
- **Auto-backup via AWS Backup** by default for new file systems
- **Encrypted at rest at creation only** — cannot enable later
- **EFS mount helper** for in-transit TLS encryption

---

## Exam Traps

- **EFS is Linux only** — for Windows use **FSx for Windows File Server**
- **EFS encryption-at-rest is enabled at creation only** — cannot retrofit
- **One Zone classes are single-AZ** — less resilient
- **Files < 128 KiB are NOT eligible for IA transitions**
- **EFS Max I/O has higher per-op latency** — only choose if you really need extreme parallelism
- **EFS does NOT work with SnapStart** in Lambda
- **Cross-Region replication is one-way** — promote replica for failover
- **EFS mount target is per AZ in a VPC** — one ENI per AZ
- **EFS does NOT support cross-VPC mounts directly** — use TGW / VPC peering OR DataSync to copy
