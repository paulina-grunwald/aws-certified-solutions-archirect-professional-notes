# Amazon RDS (Relational Database Service)

> **Managed relational database service for MySQL, MariaDB, PostgreSQL, Oracle, SQL Server, and Aurora. AWS owns the OS, patching, backups, HA. You bring schemas and queries. Multi-AZ for HA, Read Replicas for read scaling, Blue/Green for zero-downtime upgrades, RDS Proxy for connection pooling, RDS Custom when you need OS access.**

Maps to: **Domain 2.5 — Database selection**, **Domain 1.3 — Reliable / resilient**, **Domain 3.1 — Improve reliability**, **Domain 4.2 — Migration**

---

## Engines & Core Concepts

- **Supported engines**: MySQL, MariaDB, PostgreSQL, Oracle, SQL Server, **Aurora** (covered separately)
- Runs on virtual machines — **you cannot log into the underlying OS** (except **RDS Custom** for Oracle / SQL Server)
- AWS handles: OS patching, DB engine patching, backups, replication, failover, HA infrastructure
- Not serverless — for serverless relational compute use **Aurora Serverless v2**
- **Independent scaling** of compute and storage
- **Storage Auto Scaling** keeps free space ≥ 10%

> **RDS event categories cover operational events only** (DB instance, parameter group, security group, snapshot). To capture data-modifying events (`INSERT` / `UPDATE` / `DELETE`), use native triggers, stored procedures, or DB-native CDC.

---

## Multi-AZ Deployments (resilience, NOT read scaling)

### Multi-AZ Instance (classic)

- **Synchronous standby** in a different AZ within the same Region
- Standby **cannot serve reads** — only failover target
- DNS record updated on failover → **60–120 seconds**
- Failover triggers: patching, host failure, AZ failure, instance reboot with failover, instance class modification
- Failover event recorded as **`RDS-EVENT-0025`**

### Multi-AZ Cluster (newer, MySQL + PostgreSQL only)

- Primary + **two readable standbys** across three AZs
- **Semi-synchronous** replication with transaction-log-based replication
- Standbys **CAN serve read traffic** (unlike classic Multi-AZ)
- Failover typically **< 35 seconds**
- Cluster endpoint (writer) + reader endpoint + per-instance endpoints

### SQL Server Multi-AZ

- Uses **SQL Server Mirroring** (or **Always On Availability Groups** for newer versions)
- Both primary and secondary share the **same endpoint**

---

## Read Replicas (read scaling, NOT HA)

- Read-only copy via **asynchronous** replication
- Available: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora
- Requires **automated backups enabled** on the source
- Each replica has its own DNS endpoint
- **Up to 5 read replicas** per source DB (15 for Aurora)
- Multi-AZ, cross-Region, or replicas-of-replicas (up to 4 layers deep)
- **Promoting a read replica permanently breaks replication** — it becomes a standalone DB
- No automatic failover — application must update endpoints
- Monitor with CloudWatch metric **`ReplicaLag`**
- **Cross-Region replicas** = cost-effective DR (promote to standalone in DR Region) and reduced latency for global reads
- Creating an Aurora read replica from RDS MySQL / PostgreSQL is a common **migration path to Aurora**

> Read Replicas are for **read scaling and performance**, NOT resilience. Use Multi-AZ for failover.

---

## RDS Proxy

- Fully managed, HA database proxy
- **Pools and shares DB connections** — critical for Lambda and short-lived clients
- Reduces failover times by up to **66%** on Multi-AZ (proxy connects directly to the new primary)
- Enforces **IAM authentication**; credentials in **Secrets Manager**
- Engines: MySQL, PostgreSQL, MariaDB, **SQL Server**
- **Blue/Green Deployment integration** — eliminates DNS propagation delays during switchover
- Must live in the **same VPC** as the database
- **Never publicly accessible** — VPC-only

**When to use:**
- Lambda functions hitting DB connection limits
- Applications with many short-lived connections
- Bursty / unpredictable workloads
- Multi-AZ workloads where faster failover matters

---

## RDS Custom

- Provides **OS-level access** and the ability to install custom patches, third-party agents, modify DB settings beyond standard RDS
- Available for **Oracle and SQL Server** only
- Combines RDS automation with EC2-hosted flexibility
- **Automation Paused** mode lets you make custom changes without RDS reconciling them
- Shared responsibility: RDS manages infrastructure; you manage customizations above
- **Scheduled OS updates for RDS Custom SQL Server**

---

## Blue/Green Deployments

- Creates a **green** staging environment that mirrors **blue** production
- Use for major version upgrades, schema changes, parameter group changes, maintenance updates
- Changes in green do not affect blue
- **Switchover < 1 minute** with built-in rollback
- **Supported**: Aurora (MySQL/PostgreSQL compatible), RDS MySQL, RDS PostgreSQL, RDS MariaDB
- **NOT supported**: RDS Oracle, RDS SQL Server, Multi-AZ cluster deployments
- Uses **logical replication** to keep green in sync with blue
- Integrates with **RDS Proxy** for zero DNS-propagation delay on switchover

---

## Storage

- **gp3** (default for new) — independent IOPS / throughput from storage size
- **gp2** (legacy) — IOPS scale with size, burst credits
- **io1 / io2 (Provisioned IOPS)** — predictable high IOPS; io2 Block Express for the highest tier
- **Magnetic** — legacy, avoid
- **Maximum storage**: 64 TiB for MySQL / PostgreSQL / MariaDB / Oracle; **16 TiB for SQL Server**

### Storage Auto Scaling

- Triggers when free storage drops below **10%** of allocated
- No downtime
- Scales by greater of: 5 GiB, 10% of current allocation, or 7-hour predicted growth
- **Maximum Storage Threshold** caps growth

---

## Encryption & Security

### At rest

- KMS-based for all engines; encrypts storage, automated backups, snapshots, read replicas
- Two-tier envelope encryption (KMS master key → data keys → data)
- **Cannot enable encryption on an existing unencrypted DB** — snapshot → copy with encryption → restore from encrypted snapshot
- Encrypted read replicas need a KMS key in the destination Region (cross-Region)

### In transit

- **SSL / TLS** between client and RDS; per-instance certificate with endpoint as CN
- Force SSL with `rds.force_ssl` parameter (static — requires reboot)
- AWS provides regional + global CA bundles for certificate validation

### IAM Authentication

- **MySQL, PostgreSQL, MariaDB** only — NOT Oracle or SQL Server
- 15-minute auth token via AWS Signature V4
- Traffic auto-encrypted with SSL when using IAM auth
- Best combined with **RDS Proxy** for pooling + IAM auth

---

## Backups & Restore

### Automated Backups

- Retention **1–35 days** (0 disables)
- Transaction logs throughout the day
- Stored in S3 at no additional charge
- I/O may suspend briefly during backup (use Multi-AZ to hide impact)
- **Point-in-time recovery** to 5-minute granularity

### Manual Snapshots

- Persist after DB deletion
- No retention limit

> **Restoring from backup or snapshot always creates a new RDS instance** with a new DNS endpoint. Never restores in place.

---

## Logs & Monitoring

- **Error log** — diagnostic, startup/shutdown
- **General query log** — all SQL, connect/disconnect
- **Slow query log** — queries exceeding threshold
- Publish all logs to **CloudWatch Logs** for alerting + retention
- **Enhanced Monitoring** — OS-level agent metrics in real time
  - Available for all engines
  - Stored in **CloudWatch Logs `RDSOSMetrics`** (30-day default retention, configurable)
  - More accurate than CloudWatch for smaller instance classes (CloudWatch reads from hypervisor; Enhanced Monitoring from on-instance agent)
  - Per-process / per-thread CPU analysis
- **Performance Insights** — query-level performance monitoring; **free** 7-day retention, long-term retention paid

---

## DR Strategies

| Strategy | RPO | RTO | Implementation |
|---|---|---|---|
| **Backup & restore** | Hours | Hours | Automated backups + cross-Region snapshot copy |
| **Pilot light** | Minutes | Minutes | Cross-Region read replica kept in sync, promote on failover |
| **Warm standby** | Seconds | Minutes | Multi-AZ with automated failover |
| **Multi-site active/active** | Near-zero | Near-zero | **Aurora Global Database** (RDS proper can't do this) |

---

## Exam Tips

- "Minimize downtime during major version upgrade" → **Blue/Green Deployments**
- "Serverless app hitting DB connection limits" → **RDS Proxy**
- "Need OS-level access or custom patches on RDS" → **RDS Custom** (Oracle or SQL Server only)
- **Multi-AZ instance** = standby cannot serve reads; **Multi-AZ cluster** = two readable standbys across 3 AZs
- Multi-AZ cluster failover ~35 s vs Multi-AZ instance 60–120 s
- Cross-Region read replica + promotion = cost-effective DR with RPO in minutes
- "Encrypt an existing unencrypted DB" → snapshot → copy snapshot with encryption → restore from encrypted snapshot
- Storage Auto Scaling triggers at < 10% free; always set a Maximum Storage Threshold
- IAM authentication tokens are 15 minutes; pair with **RDS Proxy** for pooling
- Blue/Green switchover < 1 minute; NOT for Oracle / SQL Server / Multi-AZ cluster
- gp3 lets you configure IOPS and throughput independently of size
- Enhanced Monitoring = OS-level agent metrics; **Performance Insights** = query-level

---

## Exam Traps

- **Multi-AZ is for HA / failover only, NOT for read scaling** — exception is Multi-AZ **cluster** which does allow reads from standbys
- **Read replicas use asynchronous replication** — do not confuse with Multi-AZ synchronous replication
- **Promoting a read replica is one-way** — replication breaks permanently
- **RDS Proxy is never publicly accessible** — must be in the same VPC
- **IAM DB authentication works for MySQL / PostgreSQL / MariaDB only** — NOT Oracle or SQL Server
- **Blue/Green NOT supported for Oracle, SQL Server, or Multi-AZ clusters**
- **Restoring from backup creates a new instance with a new endpoint** — applications must update
- **You cannot enable encryption on an existing unencrypted DB directly** — must go through snapshot-copy-restore
- **RDS Custom is Oracle / SQL Server only** — not MySQL or PostgreSQL
- **Vertical scaling (instance class change) causes downtime unless Multi-AZ** is used for transparent failover during the change
- **RDS events cover operational events only** — not data-modifying events; use triggers / native CDC for those
- **RDS does NOT support importing or exporting TDE certificates**
- **MariaDB max table size is 64 TB (InnoDB file-per-table); system tablespace max is 16 TB**
