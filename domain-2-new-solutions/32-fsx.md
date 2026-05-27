# Amazon FSx

> **Managed file systems for high-performance + protocol-specific workloads. Four variants: FSx for Windows File Server (SMB + AD), FSx for Lustre (HPC scratch + persistent), FSx for NetApp ONTAP (NFS + SMB + iSCSI + multi-protocol with snapshots, clones, replication, FlexCache, FlexGroup), FSx for OpenZFS (NFS, snapshots, clones, replication). When EFS (Linux NFS) isn't the answer, FSx variants cover Windows, HPC, multi-protocol, and ZFS use cases.**

Maps to: **Domain 2.5 — Storage selection**, **Domain 4.2 — Migration**, **Domain 1.3 — Reliability**

---

## FSx Variants

| Variant | Protocol | Use case |
|---|---|---|
| **FSx for Windows File Server** | SMB | Windows file shares, AD integration |
| **FSx for Lustre** | Lustre, POSIX | HPC, ML training, video processing — high-throughput parallel |
| **FSx for NetApp ONTAP** | NFS, SMB, iSCSI | Multi-protocol; NetApp features (snapshots, clones, replication, FlexCache, FlexGroup) |
| **FSx for OpenZFS** | NFS | OpenZFS features (snapshots, clones, replication); migrate ZFS workloads |

---

## FSx for Windows File Server

- **SMB protocol** (SMB 2.0–3.1.1)
- **Active Directory integration** (AWS Managed AD or self-managed AD)
- **NTFS** file system semantics + ACLs
- **Microsoft Distributed File System (DFS) Namespaces** for unified namespace
- **SSD** (low latency) or **HDD** (cheaper) storage
- **Multi-AZ** option for HA
- **Native Windows features**: Shadow Copies (VSS snapshots), data deduplication, quotas, audit logging
- **Throughput**: up to 21 GB/s
- Use cases: lift-and-shift Windows file shares to AWS, user home directories, SQL Server SMB shares

---

## FSx for Lustre

- **High-performance Lustre** file system (parallel POSIX)
- **Sub-ms latency**, **hundreds of GB/s throughput**, **millions of IOPS**
- Two deployment options:
  - **Scratch** — short-term, lower cost, no replication
  - **Persistent** — long-term, replicated within AZ, higher durability
- **S3 integration** — link file system to an S3 bucket; objects appear as files; lazy load + write-back
- **Linux only**
- Use cases: ML training, genomics, video rendering, financial simulations, HPC

---

## FSx for NetApp ONTAP

- **Full NetApp ONTAP** experience on AWS managed
- **NFS, SMB, iSCSI** — multi-protocol from one file system
- **Snapshots, clones, SnapMirror replication**
- **FlexCache** — multi-Region caching
- **FlexGroup** — scale-out across many storage nodes
- **Multi-AZ** option
- **Tiering** between SSD (hot) and capacity pool (cold)
- Use cases: lift-and-shift NetApp / NFS / SMB workloads, multi-protocol shares, advanced data management features

---

## FSx for OpenZFS

- **OpenZFS** file system fully managed
- **NFS v3 / v4 / v4.1 / v4.2**
- **ZFS features**: snapshots, clones, compression, replication
- **Single-AZ or Multi-AZ**
- Use cases: migrate on-prem ZFS workloads, NFS file shares with ZFS data management

---

## When EFS vs FSx

| Need | Service |
|---|---|
| Linux NFS, multi-AZ, broad use | **EFS** |
| Windows SMB file shares | **FSx for Windows File Server** |
| HPC / ML training / genomics (parallel high-throughput) | **FSx for Lustre** |
| Multi-protocol NFS + SMB + iSCSI; NetApp features | **FSx for NetApp ONTAP** |
| OpenZFS workloads | **FSx for OpenZFS** |

---

## Backup

- **AWS Backup** integration for all variants
- **Daily automatic backups** (configurable retention)
- **Manual backups** anytime
- **Cross-Region copy** via AWS Backup

---

## Security

- **At rest** (KMS) — enabled at creation
- **In transit** (Kerberos for FSx for Windows; SMB encryption; TLS for OpenZFS / Lustre)
- **Security groups** for network access
- **Active Directory** for FSx for Windows; **IAM** for control plane

---

## Exam Tips

- **EFS = Linux NFS multi-AZ default answer**
- **FSx for Windows** = SMB + AD
- **FSx for Lustre** = HPC / ML training / parallel high-throughput
- **FSx for NetApp ONTAP** = multi-protocol (NFS + SMB + iSCSI), NetApp features, lift-and-shift
- **FSx for OpenZFS** = NFS + ZFS features, migration from on-prem ZFS
- **Lustre + S3** integration — files in S3 appear as POSIX files; lazy load + write-back
- **FSx for Windows** integrates with AD for user-level permissions
- **NetApp ONTAP** is the answer for "multi-protocol" file system requirements

---

## Exam Traps

- **EFS is NFS Linux only** — Windows needs FSx for Windows File Server
- **FSx for Lustre Scratch is NOT replicated** — for durability use Persistent
- **NetApp ONTAP and OpenZFS support multi-AZ** — but check storage class
- **FSx encryption at rest is enabled at creation only** — cannot retrofit
- **FSx for Windows requires AD** — either AWS Managed AD or self-managed
- **FSx for Lustre's S3 integration** is lazy — files appear instantly but data loads on first access
- **FSx integrates with AWS Backup** — manual or automatic
