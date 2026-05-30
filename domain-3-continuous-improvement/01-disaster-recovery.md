# Disaster Recovery on AWS

> **Four DR strategies on a cost-recovery spectrum: Backup & Restore (hours), Pilot Light (minutes/hours), Warm Standby (seconds/minutes), Multi-Region Active-Active (near-zero). Pick the least expensive strategy that still meets the stated RPO / RTO. Key services: AWS Elastic Disaster Recovery (DRS), AWS Backup, Aurora Global Database, DynamoDB Global Tables, S3 CRR, Route 53 failover.**

Maps to: **Domain 1.3 — Reliable / resilient architectures**, **Domain 2.2 — Business continuity**, **Domain 3.1 — Improve reliability**

---

## RPO vs RTO

| Term | Meaning |
|---|---|
| **RPO (Recovery Point Objective)** | Max **data loss** tolerated (in time). Disaster at 3 PM with RPO 2 h → must recover to ≥ 1 PM |
| **RTO (Recovery Time Objective)** | Max **downtime** tolerated. Disaster at 3 PM with RTO 4 h → must be back by 7 PM |

---

## Four DR Strategies

| Strategy | RPO | RTO | What's running in DR |
|---|---|---|---|
| **Backup & Restore** | Hours | 24 h or less | Backups in DR Region; nothing else |
| **Pilot Light** | Minutes | Hours | Data replication + core infrastructure (DBs always on; apps stopped/loaded) |
| **Warm Standby** | Seconds | Minutes | Scaled-down but fully functional copy always on |
| **Multi-Region Active-Active** | Near-zero | Near-zero | Fully scaled, serving traffic from multiple Regions |

### Backup & Restore

- Periodic backups (AWS Backup, snapshots) replicated to DR Region
- Restore on demand
- Cheapest; slowest

### Pilot Light

- Data and core infra always replicated/running
- Application servers loaded with code but **stopped** (not serving traffic)
- Used by **AWS DRS** internally

### Warm Standby

- Scaled-down but fully functional copy always running
- Scale up on failover
- Lower RTO than Pilot Light; higher cost
- Fully scaled-up = **Hot Standby**

### Multi-Region Active-Active

- Workload serves traffic from multiple Regions simultaneously
- Requires data synchronization + conflict resolution
- Route 53 / Global Accelerator route traffic to healthy Regions
- Highest cost; lowest RPO/RTO
- **Caveat**: replication does NOT protect against corruption — pair with point-in-time recovery

---

## Key AWS Services for DR

| Service | DR Role |
|---|---|
| **AWS Elastic Disaster Recovery (DRS)** | Agent-based block-level replication for **server-based** workloads (on-prem, other clouds, EC2). Pilot Light internally. RPO seconds, RTO minutes. Supports failback. **Does NOT replicate managed services (RDS, DynamoDB, S3).** |
| **AWS Backup** | Centralized, policy-based backups across services. Cross-Region + cross-account copies. Critical for Backup & Restore. |
| **Aurora Global Database** | Cross-Region replication < 1 s. RPO < 1 min, RTO ~1 min. Supports managed switchover (planned) + failover (unplanned). Warm Standby / Active-Active database tier. |
| **DynamoDB Global Tables** | Multi-Region, multi-active with automatic replication. Active-active reads + writes. **MRSC** for strongly consistent reads across 3 Regions. |
| **S3 Cross-Region Replication (CRR)** | Async object replication to another Region; cross-account supported. |
| **RDS Cross-Region Read Replicas** | Async replication; promote on DR. Pilot Light / Warm Standby database. |
| **Route 53** | DNS-based failover with health checks; active-passive or active-active. |
| **AWS Global Accelerator** | TCP/UDP failover + anycast IPs across Regions. |
| **CloudFormation / IaC** | Rapid infra recreation in DR Region (critical for Pilot Light / Warm Standby). |

---

## Cost vs Recovery Trade-off

```
Cost ◄────────────────────────────────────────►
       Backup & Restore  →  Pilot Light  →  Warm Standby  →  Active-Active
       Slowest                                                  Fastest
       Cheapest                                                 Most expensive
```

The exam frequently tests picking the **least expensive strategy that still meets the stated RPO/RTO**.

---

## DR Testing

- Well-Architected best practice → **regular DR drills**, **Game Days**
- **AWS FIS (Fault Injection Service)** — inject failures (instance termination, AZ outage simulation) to validate DR
- **DRS recovery drills** — non-disruptive test launches in DR Region
- **Aurora Global Database failover testing** — `aws rds failover-global-cluster`
- **Route 53 health check failover testing** — manually flip health checks
- **MRSC Global Tables + FIS** — resiliency testing for DynamoDB

---

## Exam Tips

- DR questions almost always state RPO/RTO — **match strategy to numbers**:
  - **Hours** → Backup & Restore
  - **Minutes** → Pilot Light
  - **Seconds/minutes** → Warm Standby
  - **Near-zero** → Active-Active
- "Minimize cost" + RPO in hours → **Backup & Restore** (don't over-engineer)
- "Server-based workload protection / migration with low RPO" → **AWS DRS**
- "Cross-Region database DR with sub-second replication" → **Aurora Global Database**
- "Multi-Region NoSQL writes" → **DynamoDB Global Tables**
- "Multi-Region strong consistency for NoSQL" → **DynamoDB MRSC Global Tables**
- "DNS-based failover" → **Route 53 failover routing** with health checks
- "TCP/UDP cross-Region failover with static IPs" → **Global Accelerator**
- "Cross-account + cross-Region backups" → **AWS Backup** (supports both)
- "Least operational overhead for DR" → managed services (DRS, Aurora Global DB, DynamoDB Global Tables) over manual replication
- **CloudEndure Disaster Recovery is now AWS DRS** — old material referring to CloudEndure should be replaced

---

## Exam Traps

- **Don't confuse Pilot Light with Warm Standby** — Pilot Light keeps only data replication + core infra running (app servers OFF); Warm Standby keeps a scaled-down but fully functional copy running
- **DRS does NOT protect managed services** — RDS / DynamoDB / S3 need service-native replication (Aurora Global DB, Global Tables, S3 CRR)
- **Active-Active ≠ just deploying in two Regions** — must handle data conflicts, synchronization, and routing
- **Backup & Restore is NOT zero-downtime** — never pick it when near-zero RTO is required
- **Multi-Region replication does NOT protect against data corruption** — pair with point-in-time recovery / versioning
- **MRSC Global Tables require exactly 3 Regions** — 2 is not enough for strong consistency across Regions
- **DRS is for server workloads only** — not Lambda, not container orchestrators, not managed databases
- **Aurora Global Database** supports **single writer**; for multi-Region writes use **Aurora DSQL** or **DynamoDB Global Tables**
- **Route 53 health checks have ~30-second TTL minimum** — RTO claims under 30 seconds via DNS are usually unrealistic
- **Global Accelerator failover is faster than Route 53** (anycast vs DNS) — pick GA for low-latency TCP/UDP failover

---

## References

- [Disaster Recovery of Workloads on AWS (Whitepaper)](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)
- [Well-Architected Reliability Pillar — Plan for DR](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/plan-for-disaster-recovery-dr.html)
