# Amazon Aurora

> **AWS's cloud-native relational database. MySQL- or PostgreSQL-compatible. 6 copies across 3 AZs, self-healing storage, auto-scaling to 256 TiB. Up to 5× MySQL perf / 3× PostgreSQL perf. Serverless v2 for variable workloads. Global Database for cross-Region active reads + write forwarding + sub-second RPO + sub-minute RTO. Aurora DSQL for active-active multi-Region with strong consistency.**

Maps to: **Domain 2.5 — Database selection**, **Domain 1.3 — Reliable / resilient**, **Domain 3.1 — Improve reliability**, **Domain 4.3 — Modernization**

---

## Overview

- Fully managed; auto-scaling with Aurora Serverless v2
- **Compatible** with **MySQL** and **PostgreSQL** — drop-in replacement at the wire protocol level
- Cluster volume grows automatically up to **128 TiB** (now **256 TiB** with recent storage enhancements)
- Stores **6 copies of data across 3 AZs**; 4/6 needed for writes, 3/6 for reads; self-healing
- Up to **15 Aurora replicas** in a cluster, each with configurable failover priority
- **5× MySQL** / **3× PostgreSQL** performance vs the open-source baselines
- Auto-failover under **30 seconds** (vs RDS Multi-AZ 60–120 s)
- Storage is striped across hundreds of volumes, fault-tolerant, self-healing
- No crash-recovery or cache-rebuild needed — storage handles it

---

## Cluster Architecture

```
                  ┌──── Aurora cluster ────┐
                  │   shared storage (6×)  │
                  └────────┬───────────────┘
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        Writer node     Reader 1     Reader N (up to 15)
              ▲            ▲            ▲
              │            │            │
        Writer endpoint  Reader endpoint (round-robin)
              │
        Custom endpoints (subset of nodes for specific workloads)
        Instance endpoints (specific node for diagnosis)
```

- **Master / writer is the only instance that can write**
- Writer + up to 15 replicas serve reads
- Aurora **Auto Scaling** dynamically adjusts replica count based on CloudWatch (CPU, connections)

---

## Endpoints

| Endpoint | Purpose |
|---|---|
| **Cluster (Writer)** | Current primary; all writes |
| **Reader** | Round-robin load balancing across replicas (connection-level, not query-level) |
| **Custom** | A subset of instances you pick (different sizes, ML-heavy, batch) |
| **Instance** | A specific instance (diagnostics, tuning) |

---

## Aurora Serverless

### v1 (legacy — end of life)

- Pause/resume capability with cold-start latency
- Does NOT support: read replicas, Multi-AZ, Global Database, Performance Insights
- **Don't propose for new architectures**

### v2 (current)

- Scales in increments of **0.5 ACU** (Aurora Capacity Units), from 0.5 ACU minimum to **256 ACU**
- **No pause in processing** during scale — much faster than v1
- Supports **read replicas, Multi-AZ, Aurora Global Database**, Performance Insights, Enhanced Monitoring, RDS Proxy
- Can **mix provisioned + Serverless v2** instances in the same cluster
- Pay per ACU-hour consumed; no capacity planning
- Ideal for variable, intermittent, or unpredictable workloads

---

## Aurora Global Database

- **1 primary Region** (read/write) + up to **5 secondary Regions** (read-only)
- Storage-level replication, lag typically **< 1 second**
- Each secondary Region can have up to **16 read replicas**
- **RTO < 1 minute** on cross-Region failover, **RPO typically < 1 second** (managed RPO available on Aurora PostgreSQL)
- **Write forwarding** — secondary-Region instances forward writes to the primary, enabling local read/write semantics for global apps
- **Blue/Green for Aurora Global Database** — upgrade primary + secondary Regions with minimal downtime
- The textbook answer for **multi-Region active/read, fast DR**

### Aurora DSQL

- Distributed SQL, **active-active multi-Region with strong consistency** (PostgreSQL-compatible)
- Different from Global Database (which is single-writer, read-only secondaries)
- Use case: applications that need writes in multiple Regions simultaneously with serialisable isolation

---

## Aurora Cloning

- **Copy-on-write** — clone shares storage with source; only modified pages cost extra
- Creating a clone is near-instantaneous regardless of DB size
- Use cases: test / staging copies, analytical queries off prod, Blue/Green orchestration
- **Cannot clone across accounts directly** — use snapshot sharing

---

## Blue/Green Deployments

- Green = staging environment that mirrors blue (production)
- **Logical replication** keeps green in sync with blue
- Switchover typically **< 1 minute** with built-in guardrails (replication-lag check)
- Ideal for: major/minor engine version upgrades, parameter changes, schema changes
- Supports Aurora MySQL and Aurora PostgreSQL
- **Aurora Global Database support**
- **RDS Proxy integration** — zero DNS-propagation delay on switchover

---

## Aurora I/O-Optimized

- Storage configuration option
- **Predictable pricing — eliminates I/O charges entirely**
- Up to **40% cost savings** for I/O-intensive workloads (when I/O costs exceed ~25% of total Aurora bill)
- Higher storage rate but $0 for read/write I/O
- Can switch between **Standard** and **I/O-Optimized** every 30 days

---

## Zero-ETL Integration with Redshift

- Near real-time analytics on transactional data without ETL pipelines
- Data replicates automatically from Aurora → Redshift
- Supports Aurora MySQL and Aurora PostgreSQL
- Enables ML / BI workloads on fresh transactional data in Redshift

---

## Aurora Multi-Master (Deprecated)

- Allowed multiple writer instances in a single Aurora cluster (MySQL only)
- **Discontinued by AWS** — never the right answer
- Replacement: **Aurora Global Database with write forwarding** for cross-Region writes, or single-master with fast failover (< 30 s)

---

## Logs & Monitoring

- **Error log, Slow query log, General log, Audit log** — download or publish to CloudWatch Logs
- **Performance Insights** — query-level wait, top SQL, hosts, users
- **Enhanced Monitoring** — host-level / per-process metrics every 1 s
- **CloudWatch metrics** — CPU, memory, swap, replica lag

---

## Security

- All Aurora instances live in a **VPC**
- **SSL/TLS** (AES-256) for data in transit
- **KMS** encryption at rest (storage, backups, snapshots, replicas)
- **Cannot encrypt an existing unencrypted DB** — snapshot → copy snapshot with encryption → restore
- **IAM database authentication** — token-based, no password
- **Secrets Manager** for automatic credential rotation
- **RDS Proxy** supports Aurora — connection pooling, IAM auth, Lambda-friendly

---

## Migrating from RDS to Aurora

- **Option 1**: Create an Aurora **Read Replica** from RDS MySQL / PostgreSQL, then promote it (minimal downtime)
- **Option 2**: Snapshot RDS → restore as an Aurora cluster (more downtime)
- Both involve some downtime — plan accordingly

---

## Exam Tips

- **Cross-Region DR with RPO < 1 s and RTO < 1 min** → **Aurora Global Database**
- **Active-active multi-Region writes** → **Aurora DSQL**
- **Variable / unpredictable workloads needing full Aurora features** → **Aurora Serverless v2** (v1 is end-of-life)
- **Read-heavy I/O-intensive workload, want predictable cost** → **Aurora I/O-Optimized**
- **Write-forwarding** in Aurora Global Database lets secondary-Region apps write without connection-string changes
- **Blue/Green** is the go-to for safe major version upgrades; switchover < 1 minute
- **Near-real-time analytics on transactional data, no ETL** → **Aurora zero-ETL → Redshift**
- **Aurora Cloning** (copy-on-write) is the fastest way to spin up a test/staging copy of prod
- **RDS Proxy** is the answer when Lambda / many short-lived clients exhaust connections
- **Auto-scaling replicas** (0–15) on CloudWatch metrics is separate from Serverless v2 ACU scaling
- **Encrypting an unencrypted Aurora DB**: snapshot → copy snapshot with encryption → restore
- **6 copies across 3 AZs**, 4/6 for writes, 3/6 for reads, self-healing storage
- **Custom endpoints** route queries to specific instance subsets (different sizes, analytics nodes)
- **Aurora storage = up to 128 TiB (256 TiB with recent updates)** — do not confuse with RDS limits (64 TiB)

---

## Exam Traps

- **Aurora Multi-Master is deprecated** — never the right answer for multi-writer scenarios; use Global Database with write forwarding (single Region writes per region) or **Aurora DSQL** (active-active strong consistency)
- **Aurora Serverless v1 is end-of-life** — v1 limitations (no read replicas, no Global DB) make it a distractor
- **Aurora reader endpoint uses connection-level load balancing, not query-level** — does not distribute individual queries
- **Aurora Global Database replication lag < 1 s** refers to storage-level, not logical
- **Blue/Green switchover is NOT zero-downtime** — brief interruption (< 1 minute); don't confuse with truly zero-downtime patterns
- **Aurora Cloning ≠ Snapshot Restore** — Cloning = instant, copy-on-write, same account; Snapshot Restore = slower, can cross accounts
- **Aurora I/O-Optimized is NOT always cheaper** — only saves money when I/O > ~25% of total Aurora bill
- **Cannot create an Aurora Global Database with Aurora Serverless v1** — but CAN with Serverless v2 (common trap)
- **Aurora storage = 128 TiB (256 TiB)** — do not confuse with RDS limits (64 TiB max)
- **RDS Proxy does NOT support Aurora Multi-Master** — another reason Multi-Master is never the answer
- **You cannot directly encrypt an existing unencrypted Aurora DB** — snapshot + encrypted copy + restore
- **Aurora ≠ RDS for Aurora** — Aurora is its own engine family in RDS but with different limits, architecture, and features (cluster volume, endpoints, replicas, Serverless v2)
