# Podcast 19 — Relational Databases

**Length target**: 15 min
**Repo references**: `domain-2-new-solutions/09-rds.md`, `10-aurora.md`, `38-documentdb.md`

## Topic & Scope

RDS + Aurora is the most-tested database topic. DocumentDB shares the Aurora architecture — covered briefly here. The exam tests Multi-AZ vs Read Replica, Aurora-specific features, and migration patterns.

## Service Coverage Depth

**Deep**:
- RDS engines (PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, Db2)
- RDS Multi-AZ (HA) vs Read Replicas (scale reads) vs Multi-AZ Cluster (3-AZ, two readable replicas)
- Aurora architecture (storage layer separated, 6× replication, up to 15 replicas)
- Aurora Serverless v2, Aurora Global Database, Aurora Backtrack
- RDS Proxy

**Brief**:
- DocumentDB (MongoDB-compatible, Aurora-like architecture)

## Structured Outline

1. **Open (30s)** — "RDS and Aurora dominate the relational story. Multi-AZ, replicas, Global Database — the levers of every architecture."
2. **RDS Engines + Sizing (3 min)** — managed engines (PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, Db2), instance classes, storage (gp3 / io2), backup retention, performance insights
3. **RDS Multi-AZ vs Read Replicas (3 min)** — Multi-AZ = HA (sync standby, automatic failover); Read Replicas = scale reads (async, up to 15, cross-Region possible); Multi-AZ Cluster (3-AZ, 2 readable standbys, 1 writer)
4. **Aurora Architecture (3 min)** — 6× storage across 3 AZs, separate compute + storage, auto-heal, up to 15 readers, cluster + reader + custom endpoints
5. **Aurora Advanced (3 min)** — Serverless v2 (sub-second scaling, fractions of ACUs), Global Database (1s replication to 5 secondary Regions), Backtrack (rewind in time without restore), Aurora I/O-Optimized pricing
6. **RDS Proxy (1 min)** — connection pooling for Lambda + bursty apps
7. **DocumentDB (30s)** — MongoDB-compatible, Aurora-style architecture
8. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- RDS Multi-AZ: synchronous standby, automatic failover (60–120s typical), same endpoint
- RDS Read Replicas: async, scale reads, can promote to standalone, support cross-Region
- RDS Multi-AZ Cluster: 1 writer + 2 readable standbys across 3 AZs, semi-sync, faster failover
- RDS automatic backups: daily snapshot + transaction logs, retention 1–35 days, point-in-time recovery
- RDS encryption: at rest with KMS (must enable at creation, can't add later directly — must snapshot + restore encrypted)
- Aurora storage: 6 copies across 3 AZs, auto-heals from disk failure
- Aurora compute: separate from storage, up to 15 read replicas (vs RDS 15 max too)
- Aurora endpoints: cluster (writer), reader (round-robin readers), custom (subset), instance endpoints
- Aurora Serverless v2: scales in 0.5 ACU increments, supports MySQL + PostgreSQL, fast scaling
- Aurora Global Database: 1s cross-Region replication, RPO < 1s, RTO < 1 min, up to 5 secondary Regions
- Aurora Backtrack: rewind cluster to a point in time WITHOUT restore (MySQL-compatible only)
- Aurora Parallel Query: push query processing to storage layer (analytics on OLTP)
- RDS Proxy: managed connection pooling, reduces DB connections, integrates with Secrets Manager + IAM
- DocumentDB: MongoDB API compatibility, 6× storage replication, up to 15 replicas, Global Clusters
- Blue/Green Deployments for RDS: managed clone + cutover for major version upgrades

## Must-Mention Exam Traps

- EXAM TRAP: RDS Multi-AZ is HA only — standby is NOT readable (unlike Aurora replicas)
- EXAM TRAP: RDS Multi-AZ Cluster (3-AZ) standbys ARE readable — different from classic Multi-AZ
- EXAM TRAP: Read Replicas don't make a primary HA — they scale reads, not failover automatically
- EXAM TRAP: Aurora is NOT serverless by default — Serverless v2 is a specific provisioning mode
- EXAM TRAP: Aurora Global Database secondaries are read-only — fail over manually for DR
- EXAM TRAP: Aurora Backtrack ≠ point-in-time restore — Backtrack is in-place rewind, MySQL only, configurable window
- EXAM TRAP: Aurora I/O-Optimized vs standard: I/O-Optimized predictable, includes I/O cost; standard pays per request
- EXAM TRAP: RDS encryption can't be added to existing unencrypted DB directly — snapshot + restore with encryption
- EXAM TRAP: RDS Proxy needs Secrets Manager for credentials — IAM auth optional
- EXAM TRAP: DocumentDB is NOT 100% MongoDB-compatible — only certain API versions
- EXAM TRAP: SQL Server Multi-AZ in RDS uses native mirroring or Always On — different from MySQL/PostgreSQL Multi-AZ implementations
- EXAM TRAP: RDS Custom (Oracle, SQL Server) gives OS access — for legacy needs
- EXAM TRAP: Aurora cluster endpoint routes ONLY to writer — reader endpoint round-robins replicas (no consistent reader pinning)
- EXAM TRAP: Aurora replica failover: ~30s typical; classic RDS Multi-AZ: 60–120s

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| HA, automatic failover, managed DB | RDS Multi-AZ |
| Scale read traffic | RDS or Aurora Read Replicas |
| Faster failover + readable standbys | RDS Multi-AZ Cluster (3-AZ) |
| MySQL/PostgreSQL with up to 5× perf | Aurora |
| Variable load, scale to zero between bursts | Aurora Serverless v2 |
| Multi-Region DR, RPO < 1s | Aurora Global Database |
| Rewind DB to recent point in place | Aurora Backtrack (MySQL only) |
| Lambda + RDS connection pooling | RDS Proxy |
| Managed MongoDB-compatible | DocumentDB |
| Need OS-level DB customization | RDS Custom |
| Predictable I/O cost regardless of volume | Aurora I/O-Optimized |

## Tone & Style

- Lead with "Multi-AZ vs Read Replica" — the classic confusion
- Hammer that Aurora replicas ARE readable; RDS classic Multi-AZ standby is NOT
- Mention Global Database SLA numbers explicitly

## Rapid-Fire Closer

"RDS Multi-AZ standby readable?" — "NO (classic)." "Aurora replica readable?" — "YES." "Cross-Region 1s replication?" — "Aurora Global Database." "Rewind without restore?" — "Aurora Backtrack." "MongoDB-compatible?" — "DocumentDB."
