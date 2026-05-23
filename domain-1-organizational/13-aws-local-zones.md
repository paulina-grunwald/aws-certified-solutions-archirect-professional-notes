# AWS Local Zones

> **Extension of an AWS Region into a specific metro area for latency-sensitive applications. AWS-owned, AWS-managed. Single-digit-millisecond latency to end users in that city.**

Maps to: **Domain 1.1 — Architect network connectivity strategies** (edge placement)

---

## Overview

- Infrastructure deployments that place AWS compute, storage, database, and select services **closer to end users in specific metro areas**
- An **extension of a parent AWS Region** — a Local Zone is associated with one Region and behaves like an extra AZ for compatible workloads
- AWS-owned and managed (unlike **Outposts**, which are on customer premises)
- Extend an existing VPC into a Local Zone by adding a subnet there
- Compatible services: **EC2, EBS, RDS, ECS, EKS, ElastiCache, FSx, ELB, Direct Connect** (subset of full Region capability — varies by Zone)

---

## When to Use

- **Low-latency at the edge** — real-time gaming, live streaming, AR/VR, virtual workstations, financial trading interfaces
- **Hybrid migrations** — move workloads close to on-prem users while keeping cloud benefits
- **Data residency** — meet state / local data residency rules (healthcare, finance, iGaming, government)
- **Media production** — video editing, render farms close to creators

---

## Architecture

```
        ┌────────────── Parent Region (e.g., us-east-1) ──────────────┐
        │                                                              │
        │   AZ1   AZ2   AZ3                                            │
        │    │     │     │                                             │
        │    └─────┴─────┘                                             │
        │           │                                                  │
        │      Region VPC ────────── extended into ──────►             │
        │                                                              │
        └──────────────────────────────────────────────────────────────┘
                                                              │
                                                              ▼
                                                    ┌──────────────────┐
                                                    │   Local Zone     │
                                                    │   (e.g., Boston) │
                                                    │   subnet + EC2   │
                                                    └──────────────────┘
```

- Single VPC spans the parent Region AND its Local Zones
- Subnets in Local Zones are like additional AZs with `<region>-<zone>-1a` naming (e.g., `us-east-1-bos-1a`)
- Data plane traffic between parent Region and Local Zone uses the AWS backbone

---

## Networking and Connectivity

- **Subnets in Local Zones** support route tables, security groups, NACLs like Regional subnets
- **Internet Gateway** routes Local Zone traffic — **local internet egress** (lower latency than hopping back to the Region)
- **Local Zone-specific Elastic IPs** (different IP range than parent Region)
- **Direct Connect** can terminate near a Local Zone for low-latency on-prem connectivity
- **Cross-Local-Zone traffic** goes via the parent Region
- **Route 53** can use Local Zones as targets for latency-based or geolocation routing

---

## Local Zones vs Wavelength vs Outposts vs Global Accelerator

| Feature | **Local Zones** | **Wavelength Zones** | **Outposts** | **Global Accelerator** |
|---|---|---|---|---|
| Location | AWS-owned metro datacenter | **Inside telco 5G networks** | **Customer premises** | AWS edge POPs (anycast) |
| Use case | Low-latency to end users in a city | Ultra-low-latency 5G mobile apps | Data residency, low-latency to on-prem | Static anycast IP, global TCP/UDP traffic |
| Owner / operator | AWS | AWS + telco | AWS hardware, customer site | AWS |
| Services available | EC2, EBS, RDS, ECS, EKS, ElastiCache, ELB | EC2, EBS, ECS, EKS | EC2, EBS, S3, RDS, ECS, EKS, Lambda | Front-end for ALB / NLB / EC2 / EIP |
| Connectivity | Public + Direct Connect | Carrier gateway in 5G network | Connected back to parent Region | Anycast routing over AWS backbone |
| Internet egress | Local | Through carrier | Through parent Region | N/A (passes traffic to backend) |

---

## Pricing

- EC2, EBS, RDS, etc. in Local Zones are priced **slightly higher** than the parent Region
- **Data transfer OUT** of a Local Zone follows standard internet egress pricing
- **No additional fee for the Local Zone itself** — pay per resource

---

## Limits and Caveats

- **Subset of AWS services** — not every Regional service is available; check per-Zone service list
- **Some instance families only** — not all EC2 families available
- **Single AZ within a Local Zone** — for HA across Zones, deploy in multiple Local Zones OR Local Zone + parent Region AZ
- **EBS volumes are tied to a Local Zone** — cannot move snapshots cross-zone directly (use S3 backup)
- **Some integrations live in the parent Region** — e.g., S3, DynamoDB calls hop back to the parent Region

---

## 2025–2026 Expansion

- New Local Zones added in metros across **North America, Europe, Asia-Pacific, Latin America**
- More services added per Zone over time
- Increased instance family coverage including **Graviton**

---

## Exam Tips

- **Latency-sensitive, single metro / city** = **Local Zone**
- **Latency-sensitive, on-prem / data residency** = **Outposts**
- **Ultra-low-latency to 5G mobile users** = **Wavelength Zone**
- **Global TCP/UDP traffic acceleration, static anycast IPs** = **Global Accelerator** (NOT Local Zones)
- A Local Zone subnet **extends an existing VPC** — same VPC, new subnet in a different physical location
- Local Zones have **local internet egress** — public traffic doesn't hop back to the parent Region
- Use **Direct Connect** to land hybrid traffic close to a Local Zone for lowest hybrid latency
- **Route 53 latency-based / geolocation routing** can target endpoints in Local Zones
- Local Zones are **AWS-owned** infrastructure — not on customer premises (that's Outposts)

---

## Exam Traps

- **Local Zones ≠ Outposts** — Local Zones are AWS-owned metro locations; Outposts are on customer premises
- **Local Zones ≠ Wavelength** — Wavelength is in telco 5G networks for mobile carriers
- **Local Zones ≠ Global Accelerator** — Global Accelerator gives global anycast IPs in front of backends; Local Zones are physical infrastructure for running workloads close to users
- **Not all services are available in every Local Zone** — verify per-Zone service availability
- **Single AZ within a Local Zone** — for HA, deploy across multiple Local Zones or fall back to the parent Region
- **A Local Zone is associated with one parent Region** — cross-Region failover requires resources in the other Region's Local Zones
- **Local Zone egress uses standard egress pricing** — Local Zones don't lower egress costs
