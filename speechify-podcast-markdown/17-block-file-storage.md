# Podcast 17 — Block & File Storage

**Length target**: 15 min
**Repo references**: `domain-2-new-solutions/08-ebs.md`, `31-efs.md`, `32-fsx.md`, `domain-4-migration/06-storage-gateway.md`

## Topic & Scope

EBS (block), EFS (NFS), FSx (Windows / Lustre / NetApp ONTAP / OpenZFS), Storage Gateway (hybrid). The most-asked storage decision is "which file system" — get the distinctions right.

## Service Coverage Depth

**Deep**:
- EBS volume types (gp2/gp3, io1/io2, st1, sc1)
- EFS — NFS, multi-AZ, performance modes, storage classes
- FSx four flavors (each has a clear use case)
- Storage Gateway three modes (File / Volume / Tape)

**Brief**:
- EBS snapshots cross-region + cross-account
- Instance Store (mention as ephemeral)

## Structured Outline

1. **Open (30s)** — "Block, file, hybrid — get the right storage for the workload pattern."
2. **EBS (3.5 min)** — block, single-AZ (single-EBS-volume), volume types (gp3 default, io2 for IOPS, st1 / sc1 throughput, EBS Multi-Attach for io1/io2), snapshots to S3, encryption
3. **EFS (3 min)** — managed NFS, multi-AZ, performance modes (general / max I/O), throughput modes (bursting / provisioned / elastic), storage classes (Standard / IA / Archive)
4. **FSx Four Flavors (4 min)** — Windows (SMB + AD), Lustre (HPC, GBs/s, scratch + persistent), NetApp ONTAP (multi-protocol NFS/SMB/iSCSI + dedup), OpenZFS (NFS, snapshots, cloning)
5. **Storage Gateway (2.5 min)** — File Gateway (SMB/NFS backed by S3), Volume Gateway (iSCSI backed by EBS snapshots, cached vs stored), Tape Gateway (VTL → S3 Glacier)
6. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- EBS gp3: 3,000 IOPS / 125 MB/s baseline free, scale IOPS and throughput independently — default choice
- EBS gp2: legacy, IOPS scales with size (3 per GB) — migrate to gp3
- EBS io2 Block Express: up to 256k IOPS, sub-ms latency, durable; use for SAP HANA, Oracle
- EBS st1: throughput-optimized HDD, large sequential reads
- EBS sc1: cold HDD, lowest cost, infrequent access
- EBS Multi-Attach: attach io1/io2 to multiple EC2 in same AZ (clustered apps)
- EBS snapshots are stored in S3, can be copied cross-region + cross-account
- EFS: NFSv4, multi-AZ, scales to PBs, mounts to many EC2 / Lambda / Fargate
- EFS performance: General (lower latency) vs Max I/O (higher throughput, higher latency)
- EFS throughput: Bursting (default), Provisioned (paid baseline), Elastic (auto-scale, recommended)
- EFS storage classes: Standard, IA (cost-optimized), Archive — lifecycle policy moves files
- FSx for Windows: SMB protocol, AD-integrated, supports DFS, Multi-AZ
- FSx for Lustre: parallel file system, GBs/s throughput, integrate with S3 (scratch / persistent)
- FSx for NetApp ONTAP: multi-protocol (NFS/SMB/iSCSI), snapshots, dedup, FlexClone, Multi-AZ
- FSx for OpenZFS: NFS, snapshots, cloning, lower cost than ONTAP for many cases
- Storage Gateway: 3 types — File / Volume / Tape Gateway
- File Gateway: SMB/NFS share backed by S3, local cache, integrates with Backup
- Volume Gateway: iSCSI, cached (hot in local, all in S3) or stored (all local, snapshot to S3)
- Tape Gateway: VTL replacing physical tape backups, goes to S3 Glacier

## Must-Mention Exam Traps

- EXAM TRAP: EBS is SINGLE-AZ (per volume) — for multi-AZ you replicate or use a multi-AZ file system
- EXAM TRAP: EBS gp2 IOPS scale with size — for small disks, IOPS is limited
- EXAM TRAP: EFS is MULTI-AZ by default (Regional file system)
- EXAM TRAP: EFS One Zone class is single-AZ — cheaper but no Multi-AZ
- EXAM TRAP: FSx Windows requires Directory Service (Managed AD or AD Connector) — not standalone
- EXAM TRAP: FSx Lustre scratch is temporary (no replication) — persistent has replication option
- EXAM TRAP: FSx ONTAP supports NFS + SMB + iSCSI; FSx Windows supports SMB only; FSx OpenZFS supports NFS only
- EXAM TRAP: Storage Gateway File Gateway ≠ DataSync — File Gateway is for ongoing share; DataSync is for one-time bulk migration
- EXAM TRAP: Volume Gateway cached vs stored — cached = primary in S3 with local cache; stored = primary local with S3 backup
- EXAM TRAP: Tape Gateway uses iSCSI VTL — appears as virtual tape library to existing backup software
- EXAM TRAP: EBS snapshots are incremental — first one is full, subsequent are deltas
- EXAM TRAP: Fast Snapshot Restore (FSR) on EBS removes lazy-load latency — costs $0.75/hr/AZ per snapshot
- EXAM TRAP: EFS lacks fine-grained per-file IOPS controls — for high-perf low-latency, EBS or FSx Lustre

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| Single EC2 boot volume, general purpose | EBS gp3 |
| High IOPS DB on single EC2 | EBS io2 (Block Express for extreme) |
| Shared file system across many EC2 / Lambda | EFS |
| Windows file share for users + apps | FSx for Windows |
| HPC parallel I/O | FSx for Lustre |
| Multi-protocol (NFS + SMB + iSCSI) + snapshots | FSx for NetApp ONTAP |
| NFS + low-cost snapshots | FSx for OpenZFS |
| On-prem SMB/NFS share to S3 | Storage Gateway File Gateway |
| On-prem iSCSI to AWS | Storage Gateway Volume Gateway |
| Replace physical tape backup | Storage Gateway Tape Gateway |
| One-time large file migration | DataSync (podcast 27) |

## Tone & Style

- Frame: "Block (EBS), File (EFS / FSx), Object (S3 — next podcast), Hybrid (Storage Gateway)"
- For FSx, drill the four flavors with a memorable hook each
- Repeat "EBS = single AZ" since it's a common trap

## Rapid-Fire Closer

"EBS single-AZ?" — "YES." "EFS multi-AZ?" — "YES." "SMB Windows share?" — "FSx Windows." "HPC parallel?" — "FSx Lustre." "Multi-protocol + dedup?" — "FSx ONTAP." "On-prem to S3 share?" — "File Gateway."
