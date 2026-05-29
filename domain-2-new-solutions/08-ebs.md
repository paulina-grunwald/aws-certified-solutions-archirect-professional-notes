# Amazon EBS (Elastic Block Store)

> **Persistent block storage attached to EC2 over the network. AZ-scoped, replicated within the AZ, snapshotted to S3 (cross-Region copyable). Six volume types (gp3 default for new, io2 Block Express for highest performance). Multi-Attach for io1/io2 cluster apps. Encrypted by default.**

Maps to: **Domain 2.5 — Storage selection**, **Domain 1.3 — Reliability**, **Domain 3.5 — Cost optimization**

---

## Fundamentals

- **Block storage** — works like a virtual disk attached over the network
- **AZ-scoped** — each volume lives in exactly one AZ; replicated within that AZ for durability
- **Attached to one EC2 instance** by default; **Multi-Attach** allows io1/io2 to attach to up to 16 instances in the same AZ
- Multiple volumes can be attached to one EC2 instance
- **Snapshots live in S3** (managed; you don't see the bucket); **incremental** — only changed blocks since the last snapshot
- **Persistent across stop/reboot** (unlike instance store)
- **gp3 is the default** for new general-purpose volumes
- **Encrypted by default** at account+region level (AWS-owned key by default; customer-managed KMS optional)

## IOPS vs Throughput vs Bandwidth

| Term | Meaning |
|---|---|
| **IOPS** | Input/Output operations per second — speed of small random reads/writes |
| **Throughput** | Data transfer rate in MiB/s — large sequential reads/writes |
| **Bandwidth** | Total network pipe between EC2 instance and EBS (instance-type-dependent) |

Bandwidth = pipe; Throughput = water; IOPS = drips per second.

---

## Volume Types (current)

| Type | Class | Max IOPS | Max throughput | Size | Best for |
|---|---|---|---|---|---|
| **gp3** | SSD (default) | 16,000 | 1,000 MiB/s | 1 GiB – 16 TiB | General workloads; independent IOPS / throughput |
| **gp2** | SSD (legacy) | 16,000 | 250 MiB/s | 1 GiB – 16 TiB | Legacy; IOPS tied to size (3/GiB, burst credits) |
| **io2 Block Express** | SSD (highest perf) | **256,000** | 4,000 MiB/s | 4 GiB – **64 TiB** | Mission-critical, sub-ms latency: SAP HANA, SQL Server, Oracle |
| **io1** | SSD (legacy PIOPS) | 64,000 | 1,000 MiB/s | 4 GiB – 16 TiB | Replaced by io2 |
| **st1** | HDD (throughput) | 500 | 500 MiB/s | 125 GiB – 16 TiB | Cheap sequential: big data, logs, data warehouse |
| **sc1** | HDD (cold) | 250 | 250 MiB/s | 125 GiB – 16 TiB | Lowest-cost block: infrequent access |

- HDD (st1, sc1) **cannot be boot volumes**
- **gp3** is ~20% cheaper than gp2 and lets you provision IOPS/throughput independently of size
- **io2 has 99.999% durability** (vs 99.8–99.9% for other types)

---

## Multi-Attach (io1 / io2)

- Attach the same volume to **up to 16 EC2 instances in the same AZ**
- Each instance has full read/write access
- Application must coordinate writes (cluster-aware filesystem like GFS2, OCFS2 — NOT XFS / ext4)
- Use case: clustered Linux apps, Teradata, Oracle RAC-style HA

---

## Snapshots

- **Incremental** — only changed blocks since the last snapshot
- **Initial snapshot** of a volume takes longest; subsequent are fast
- Can be taken **while the instance is running** (quiesce app or freeze filesystem for consistency)
- **Multi-volume snapshots** — point-in-time across all EBS volumes attached to an EC2 instance
- Stored in **S3** (managed; not visible as a bucket)
- **Cross-Region copy** + **cross-account share** supported
- **Snapshot copy** can re-encrypt with a new KMS key
- **Marketplace product codes** propagate to snapshots
- **Pending snapshot limits**: 1 pending per st1/sc1, 5 pending per other types
- **Volumes restored from snapshots load lazily** — first-touch reads slow; use **Fast Snapshot Restore (FSR)** to pre-warm blocks (per-AZ, per-snapshot cost)
- **Snapshot Archive** — ~75% cheaper for long-term retention (90-day minimum, restore 24–72h)
- **Recycle Bin** — recover accidentally deleted snapshots / AMIs within a configurable retention window
- **EBS Direct APIs** — read snapshot blocks directly (forensics, backup vendors); read-only

---

## Data Lifecycle Manager (DLM) vs AWS Backup

- **DLM** — automate creation/retention/deletion of EBS snapshots + AMIs (free, EBS-only)
- **AWS Backup** — centralized backup across many services (EBS, RDS, DynamoDB, EFS, FSx, ...) with reporting, cross-Region/cross-account copy, Vault Lock, audit

---

## Encryption

- **Account-level setting** — enable "encryption by default" per Region; cannot be disabled per volume
- **At rest**: KMS envelope encryption; **symmetric keys only** (no asymmetric CMKs)
- **In transit**: between EC2 and EBS (always encrypted in transit on Nitro instances)
- **Encrypting an unencrypted volume**: snapshot → copy snapshot with encryption → create new volume from encrypted snapshot
- **Cannot change the KMS key on an existing encrypted volume or snapshot** — re-encrypt during snapshot copy
- **AWS SMS**: do NOT enable encryption by default during migration; enable AMI encryption when creating the replication job

---

## Deletion Behavior

- **`DeleteOnTermination` defaults**: `true` for root, `false` for additional volumes
- Configurable at launch (Console / CLI) or on running instances (CLI only)
- **Deleting a volume zero-overwrites the physical block storage** before reuse — no manual wiping required

---

## RAID

- **RAID 0** (striping) — combine volumes for higher IOPS/throughput; no redundancy
- **RAID 1** (mirroring) — redundancy; halves usable size
- **RAID 5 / 6 NOT recommended** on EBS (parity overhead negates benefits; EBS already replicates within AZ)
- Volume failure on a RAID 0 still loses data; use snapshots as the durability layer

---

## Moving EBS Across AZ / Region

- **Same AZ**: detach → re-attach to a different instance
- **Different AZ**: snapshot → create new volume from snapshot in target AZ
- **Different Region**: snapshot → copy snapshot to target Region → create new volume there

---

## Anti-Patterns

- Temporary scratch → **Instance Store** (ephemeral, free, faster, AZ-local)
- Multi-instance shared writes → **EFS** (NFS) or **FSx** (SMB / Lustre); EBS Multi-Attach only with cluster-aware FS
- 99.999% durability outside an AZ → snapshot or replicate at app level

---

## Exam Tips

- **gp3 is the default** for general-purpose; provision IOPS / throughput independently of size
- **io2 Block Express** for highest IOPS / sub-ms / 99.999% durability (SAP HANA, SQL Server, Oracle)
- **HDD (st1, sc1) cannot be boot volumes**
- **Multi-Attach** is io1 / io2 only, up to 16 instances in the same AZ, requires cluster-aware filesystem
- **Encrypting an existing unencrypted volume**: snapshot → copy with encryption → restore
- **Cross-Region copy re-encrypts with the target Region's KMS key**
- **FSR** for predictable first-touch performance after restore (per-AZ, per-snapshot cost)
- **Snapshot Archive** for ~75% cheaper long-term retention (90-day min, 24–72h restore)
- **Recycle Bin** protects against accidental snapshot / AMI deletion
- **DLM** automates EBS snapshot lifecycle for free; **AWS Backup** does it across many services
- **Default `DeleteOnTermination` = true for root, false for additional volumes**
- **EBS volumes are AZ-scoped** — for cross-AZ HA, use snapshots + cross-AZ restore or replicate at app level
- **Initialization (lazy loading)** can impact performance up to 50% on the first read of each block — pre-read or use FSR

---

## Exam Traps

- **EBS volumes are AZ-scoped** — they cannot span AZs; an AZ failure makes the volume unreachable
- **Multi-Attach is single-AZ only** — does not solve multi-AZ HA
- **Multi-Attach requires a cluster-aware filesystem** — XFS / ext4 will corrupt; use GFS2 / OCFS2
- **Cannot change the KMS key on an existing encrypted volume or snapshot** — re-encrypt by copying the snapshot
- **Encryption by default is per-Region** — must be enabled separately in each Region
- **Asymmetric KMS keys are NOT supported** for EBS encryption
- **HDD volumes (st1, sc1) cannot be boot volumes**
- **Snapshots restored to a new volume load lazily** — first-touch reads are slow unless FSR or pre-read
- **RAID 5/6 are NOT recommended** on EBS — use RAID 0 for performance, RAID 1 for redundancy
- **Snapshot Archive minimum retention is 90 days** — early deletion billed for the full 90 days
- **gp2 has burst credits**; gp3 does not (gp3 gives baseline 3,000 IOPS / 125 MiB/s without bursting)
- **Initial snapshot takes the longest** — subsequent snapshots are incremental
- **Cannot snapshot a hibernated instance** or create snapshots from hibernation-enabled instances
