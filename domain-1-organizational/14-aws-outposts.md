# AWS Outposts

> **Fully managed service that extends AWS infrastructure, services, APIs, and tools to virtually any on-premises or edge location for a consistent hybrid experience. AWS owns, operates, and maintains the hardware; you own the workloads.**

Maps to: **Domain 1.1 — Architect network connectivity strategies** (hybrid connectivity)

---

## Overview

- **Hybrid cloud**: combine on-premises infrastructure with AWS cloud using the same AWS services, APIs, and tools
- AWS sets up and manages physical infrastructure (compute, storage, networking) at your location
- You access Outposts capacity via the **same AWS console / CLI / SDK** as the parent Region
- **You are responsible for the Outposts physical security** at your location (power, cooling, locked space)
- Outposts hardware is **owned and operated by AWS** — you cannot administer the OS underneath the AWS services
- Billed monthly, pay-as-you-go (3-year terms with upfront / partial / no-upfront options)

---

## Outposts Family — Form Factors

### 1. Outposts Rack (42U industry-standard)
- Full 42U rack with AWS compute, storage, networking
- Scale from a single rack to up to **96 racks** at one site
- Capacity options: a range of EC2 instance types + EBS volumes
- Includes **Local Gateway (LGW)** for on-prem network connectivity
- Best for: data center deployments needing large local capacity, low-latency to on-prem systems, data residency

### 2. Outposts Servers (1U and 2U)
- Small form-factor servers for **space-constrained or edge locations**
- **1U server**: Graviton2-based (`c6gd`), 64 vCPUs, 128 GB RAM, 4 TB local NVMe
- **2U server**: Intel Xeon (`c6id`), 128 vCPUs, 256 GB RAM, 8 TB local NVMe
- Single server unit (no multi-server clustering like Rack)
- Uses **Local Network Interface (LNI)** for on-prem connectivity (no Local Gateway)
- Best for: branch offices, retail stores, factory floors, telco edge, remote sites

> 💡 **Decision**: large data center with significant local compute needs → **Outposts Rack**. Small edge / branch / remote site → **Outposts Servers**.

---

## Supported Services

### On Outposts Rack (broader service catalog)
- **EC2** (broad instance family selection, including Graviton)
- **EBS** (gp2, io2)
- **S3 on Outposts** (object storage with S3 APIs)
- **ECS, EKS** (containers)
- **EMR** (big data)
- **RDS** (PostgreSQL, MySQL — limited engine support)
- **ElastiCache** (Redis, Memcached)
- **Application Load Balancer (ALB)**
- **AWS App Mesh, AWS Service Mesh**
- **Route 53 Resolver** (DNS)
- **AWS Outposts API Gateway**

### On Outposts Servers (limited)
- **EC2** (only specific instance types based on the server model)
- **ECS** (containers)
- **VPC** networking
- **AWS IoT Greengrass** (edge processing)
- **SageMaker Edge Manager** (edge ML inference)
- No EBS, no RDS, no S3 on Outposts — local NVMe storage only

> ⚠️ **Many AWS services run only in the parent Region**, not on Outposts. Outposts depends on a service link to its parent Region for management and for accessing those services.

---

## Networking — Service Link, Local Gateway, LNI

### Service Link
An encrypted set of VPN tunnels from the Outpost back to its **parent AWS Region**.
- Required for management plane (control plane, AWS APIs)
- Goes over public internet or AWS Direct Connect (recommended for production)
- Minimum **1 Gbps**; AWS recommends redundant 10 Gbps for production
- If service link is lost: **EC2 and existing workloads continue running**, but you can't launch new resources or use AWS APIs

### Local Gateway (LGW) — Outposts Rack only
- Logical interconnect virtual router that enables communication between the Outpost VPC subnet and your **on-premises network**
- Used as a route target in VPC route tables associated with Outpost subnets
- Supports **customer-owned IP (CoIP) pools** — assign on-prem IPs to Outpost resources
- Configurable BGP peering with your on-prem routers for dynamic routing

### Local Network Interface (LNI) — Outposts Servers only
- ENI-like interface that connects an Outpost server directly to your on-prem LAN
- No Local Gateway; servers participate in the on-prem network directly

### Outposts Subnet
- A normal VPC subnet that "lives" on the Outpost rack / server
- Same VPC as Region subnets; same routing, security groups, NACLs
- Cross-AZ patterns work between Outpost subnet and Region AZs via service link

---

## S3 on Outposts

- Use **S3 APIs** to store and retrieve data **locally on the Outpost** (Outposts Rack only)
- S3 Storage Class: **`OUTPOSTS`**
- Default encryption with **SSE-S3** (SSE-KMS not supported on Outposts S3)
- Capacity options: 26 TB, 48 TB, 96 TB, 240 TB, 380 TB
- Use cases: keep data close to on-prem apps, reduce transfer to AWS Regions, data residency
- Access via **S3 access points** — required for Outposts S3 (not direct bucket endpoints)
- Replication to Region S3 supported (Cross-Region Replication style)

---

## EKS Local Clusters on Outposts

- Run a full **Kubernetes control plane locally** on Outposts Rack (not just worker nodes)
- **Survives temporary service link disconnection** — EKS cluster keeps operating during Region disconnection
- Self-managed control plane (no Region dependency for control plane)
- Useful for **air-gapped or unreliable network** environments
- Different from "EKS on Outposts" with Region-managed control plane (Region control plane + Outpost worker nodes)

---

## Use Cases

- **Low-latency on-prem apps**: manufacturing IoT, telco, financial trading, healthcare imaging
- **Data residency / sovereignty**: regulatory requirement that data stays in a specific country / building
- **Edge processing**: real-time analytics on data that can't (or shouldn't) leave the local site
- **Migration staging**: lift-and-shift workloads to AWS hardware on-prem, then migrate to Region later
- **Local hardware refresh**: replace aging on-prem data centers with AWS-managed hardware
- **Air-gapped / disconnected operations**: branch banks, retail, military, oil rigs (with EKS local clusters or Outposts Servers)

---

## Order and Installation Process

1. **Choose** form factor (Rack vs Server) and capacity configuration in AWS Console
2. **Site survey**: AWS reviews your facility (power, cooling, network, physical space, weight limits)
3. **Site requirements**:
   - Outposts Rack: ~10 kW typical power, 2,000 lbs weight, redundant power, climate-controlled, sufficient floor space
   - Outposts Servers: standard rack space, much less power
4. **AWS delivers and installs** the hardware
5. **You activate** via console; capacity becomes available as EC2 / EBS / etc.
6. **AWS manages** patching, hardware replacement, capacity health

**Terms**: 3-year commitment (1-year limited availability); upfront / partial / no-upfront pricing options

---

## Disaster Recovery Considerations

- **Outposts is single-site** — no automatic HA across Outposts
- For DR, pair an Outpost with the parent AWS Region OR with another Outpost
- Common patterns:
  - **Outpost + Region pairing**: replicate data and run failover stack in Region
  - **Two Outposts in different sites**: cross-Outpost replication via Direct Connect / VPN
- AWS Backup supports backup of Outpost resources to a Region-resident S3 bucket
- EBS snapshots from Outpost-resident volumes can be stored in the parent Region (with service link)

---

## Outposts vs Local Zones vs Wavelength — Decision Matrix

| Service | Who manages | Where it sits | Latency target | Primary use case |
|---|---|---|---|---|
| **Outposts (Rack / Servers)** | AWS-managed in **your** location | Your data center / branch | Sub-millisecond local | Hybrid, data residency, low-latency local |
| **Local Zones** | AWS-managed in **AWS-owned** metro location | Major metro city (NYC, LA, etc.) | <10 ms to end users in that metro | Latency-sensitive consumer apps near population centers |
| **Wavelength** | AWS-managed inside **telco 5G networks** | Telecom carrier site | <10 ms over 5G | Mobile / 5G edge apps (gaming, AR/VR, V2X) |
| **Region** | AWS in a Region | AWS data center | 10–100 ms+ | General cloud workloads |

**Exam pattern**: "data must stay in customer facility" → **Outposts**. "Low-latency to end users in a specific city" → **Local Zones**. "5G edge app over carrier network" → **Wavelength**.

---

## Pricing

- **Pay for the Outposts capacity** (compute + storage configuration) on a **3-year commitment**
- Options: **all-upfront, partial-upfront, no-upfront** monthly billing
- **Data transfer**: data sent to the parent Region over the service link is charged at standard egress rates
- **Data residency benefit**: local data and processing avoid Region transfer costs entirely
- **AWS responsible**: hardware, patching, maintenance, replacements
- **You responsible**: physical site (power, cooling, network), OS / app management within EC2 instances

---

## Exam Tips

- **AWS Outposts is the answer** when scenario requires: "AWS services on-prem," "data must stay in customer facility," "low single-digit ms latency to on-prem systems," "hybrid with consistent AWS APIs"
- **Outposts Rack** = large data center deployments (42U, up to 96 racks); supports broader service catalog
- **Outposts Servers** = 1U/2U for edge / branch / space-constrained locations; limited services (EC2, ECS, VPC, IoT Greengrass, SageMaker Edge)
- **Local Gateway (LGW)** is Rack-only — connects Outpost VPC subnet to on-prem network
- **LNI (Local Network Interface)** is Servers-only — direct on-prem LAN attachment
- **Service link** to parent Region required for management; loss of service link means existing workloads keep running but no new launches / APIs
- **EKS Local Clusters** on Outposts have a **locally-hosted K8s control plane** — survives service link disconnection
- **S3 on Outposts** uses the **`OUTPOSTS` storage class**, requires **S3 Access Points**, default SSE-S3 encryption (no SSE-KMS)
- **Customer responsible for physical security** of the Outposts hardware
- **3-year commitment**: upfront / partial-upfront / no-upfront pricing
- For DR: **Outpost + Region** pairing or **Outpost + Outpost** pairing
- **Latency comparison**: Outposts (sub-ms local) < Local Zones (<10 ms metro) < Wavelength (<10 ms 5G edge) < Region (10–100 ms+)
- **Data residency**: Outposts keeps data on-prem; Local Zones and Wavelength still in AWS-owned property

---

## Exam Traps

- Outposts hardware is **managed by AWS** — you use standard AWS service APIs only, with no OS-level access to the underlying servers
- Outposts Servers (1U / 2U) use **LNI (Local Network Interface)**, not Local Gateway — if the scenario specifies LGW, it must be Outposts Rack
- Many AWS services run **only in the parent Region** — Outposts supports a subset. The service link provides access to Region-only services from Outpost workloads
- S3 on Outposts is **Rack-only** — Outposts Servers have local NVMe only, with no managed S3 or EBS
- Outposts is **single-site** with no automatic cross-Outpost HA — DR must be designed explicitly
- Running workloads **continue during service link loss** — but you cannot launch new resources or use AWS APIs until the link is restored
- S3 on Outposts defaults to **SSE-S3 encryption** — SSE-KMS is not supported on Outposts S3
- Outposts is **permanent AWS infrastructure** at your location; Snowball is data transfer / temporary edge compute — they serve different purposes
- Outposts is in **your location**; Local Zones and Wavelength are in **AWS-managed edge facilities** — they are distinct deployment models
- Relocating an Outpost requires **physical hardware decommission** and reordering from AWS — you cannot simply move it
- Outposts requires a **3-year commitment** — it is not pay-as-you-go like Region services
- EBS on Outposts supports **gp2 and io2 only** — gp3, st1, sc1, and io2 Block Express are not available
- When a scenario requires data to **never leave the customer building**, the answer is Outposts — Local Zones and Regions do not satisfy that requirement
- AWS Backup copies from Outposts are stored **in the Region** (not on-prem) — verify whether the scenario's data residency requirements allow that
