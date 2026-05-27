# AWS Snow Family

> **Physical devices for offline data transfer when network bandwidth is insufficient. Move TB to PB of data without depending on networks. Current devices: Snowcone (small / edge), Snowball Edge Storage Optimized (80 TB HDD), Snowball Edge Compute Optimized (42 TB HDD + GPU option). AWS Snowmobile (semi-truck, 100 PB) was RETIRED December 2024. Manage devices with AWS OpsHub (GUI) instead of CLI.**

Maps to: **Domain 4.2 — Optimal migration approach**, **Domain 2.5 — Storage selection**

---

## Overview

- **Physical** data-transport solution: move **TBs or PBs** of data into / out of AWS
- Alternative to moving data over the network (avoid network fees + slow transfer)
- Pay **per data transfer job**
- Provide **block storage** + **S3-compatible object storage** while in-field
- Some devices support **edge compute** (run EC2, Lambda, ML inference locally)

> **AWS Snowmobile (semi-truck, 100 PB / device) was RETIRED December 2024** — never the right answer.

---

## Current Devices

### Snowcone (small / edge)

| Variant | Storage | Use |
|---|---|---|
| **Snowcone** | 8 TB HDD | Light edge / data transfer in space-constrained sites |
| **Snowcone SSD** | 14 TB SSD | Faster edge / data transfer |

- Light (2.1 kg), rugged, portable
- Bring your own battery + cables
- Can ship back offline OR connect to internet + use **AWS DataSync** to push data
- **DataSync agent preinstalled** on Snowcone

### Snowball Edge Storage Optimized

- **80 TB usable HDD** for block + S3-compatible object storage
- 40 vCPUs, 80 GB RAM
- For large data migrations, DC decommissions, DR seeding

### Snowball Edge Compute Optimized

- **42 TB HDD** OR **28 TB NVMe SSD**
- **104 vCPUs, 416 GiB RAM**
- **Optional NVIDIA GPU** (video processing, ML inference at the edge)
- For edge analytics, video / sensor pre-processing, disconnected compute

---

## Workflow

```mermaid
flowchart LR
    A[1. Request device in AWS Console] --> B[2. AWS ships device]
    B --> C[3. Install OpsHub / Snowball client]
    C --> D[4. Copy data to device]
    D --> E[5. Ship device back to AWS]
    E --> F[6. Data loaded into S3]
    F --> G[7. Device wiped]
```

---

## Edge Computing on Snow Family

Snowball Edge variants can run:

- **EC2 instances** (sbe1 / sbe-c / sbe-g families)
- **AWS Lambda** functions
- **AWS IoT Greengrass** for IoT workloads
- **S3-compatible** object store + EBS-compatible block storage

Use cases: disconnected sites (oil rigs, ships, military), edge ML inference, manufacturing floor analytics.

---

## AWS OpsHub

- Desktop GUI to manage Snow Family devices without CLI
- Unlock + configure single or clustered devices
- Transfer files
- Launch + manage EC2 instances on device
- Monitor metrics (storage capacity, active instances)
- Launch DataSync / NFS / S3 endpoints on device

---

## Improving Transfer Performance

Most impactful to least:

1. **Parallel writes** from multiple terminals
2. **Batch small files into archives** (≥ 1 MB each)
3. **Don't run other operations** on the source during transfer
4. **Reduce local network use** (dedicated NIC if possible)
5. **Direct connection** between client and device

> **File interface**: 25–40 MB/s. **S3 Adapter for Snowball**: 250–400 MB/s. Use the S3 Adapter for fastest transfer.

---

## Cost Comparison

| Data Volume | Recommended Path |
|---|---|
| < 10 TB | **DataSync** over the network |
| 10 TB – 80 TB | **Snowball Edge Storage Optimized** |
| 80 TB – multi-PB | Multiple Snowball Edge devices in parallel |
| > 100 PB | Multiple Snowball Edge devices in waves + Direct Connect for incremental updates (Snowmobile retired) |

---

## Security

- **Encryption at rest** with 256-bit AES; keys managed in KMS
- **Tamper-resistant + tamper-evident** enclosure
- **Trusted Platform Module (TPM)** for chain-of-custody
- Devices are **NIST 800-88 wiped** after data ingest

---

## Exam Tips

- "Migrate PB+ data without saturating network" → **Snow Family** (multiple Snowball Edge devices)
- "Offline edge compute + data collection at remote site" → **Snowball Edge Compute Optimized** or **Snowcone**
- "ML inference at the edge with GPU" → **Snowball Edge Compute Optimized with optional GPU**
- "Tiny portable device for field site" → **Snowcone** (2.1 kg, edge)
- "GUI to manage Snow devices (no CLI)" → **AWS OpsHub**
- "Snowcone has DataSync agent preinstalled" — push data via DataSync once back online
- "Multi-PB migration when Snowmobile is mentioned" → Snowmobile is **retired**; use multiple Snowball Edge devices in parallel

---

## Exam Traps

- **AWS Snowmobile is RETIRED (2024)** — never the right answer; old material lists it as the answer for 10+ PB
- **Snow Family is NOT for ongoing replication** — one-time bulk transfer
- **Snow Family does NOT support real-time data** — multi-day shipping latency
- **Snowball Edge ≠ Snowball** — legacy "Snowball" (50/80 TB without compute) is gone; current devices are all "Snowball Edge" variants
- **Snowball Edge cluster mode** = fault tolerance + capacity (5–10 nodes), NOT performance multiplication for single transfers
- **Snowcone is the only device you supply power / cables for** — Snowball Edge devices have their own
- **Don't pick Snow Family for small (< 10 TB) migrations** — DataSync is cheaper and faster end-to-end
- **File interface ≈ 25–40 MB/s** — use the **S3 Adapter for Snowball** for 250–400 MB/s
