# AWS Global Infrastructure

> **Foundation of every architecture. Regions, AZs, Local Zones, Wavelength, Outposts, Edge Locations, Regional Edge Caches, CloudFront, Lambda@Edge, Global Accelerator. Picking the right footprint is half of SAP-C02.**

Maps to: **Domain 1.1 — Architect network connectivity strategies** (foundational)

---

## Overview

**36 Regions** (and growing), **114+ Availability Zones**, **500+ edge locations** (Points of Presence), plus specialised footprint like Local Zones, Wavelength Zones, Outposts, and the AWS GovCloud / China partitions.

---

## Regions

- A **physical geographic area** containing multiple, isolated **Availability Zones**
- **Independent** (separate fault domain, separate billing, separate service availability)
- Choose for: proximity to users, data residency / compliance, service availability, cost (prices vary)
- **Not all services are available in every Region** — newer services launch first in commercial Regions
- Codes like `us-east-1`, `eu-west-2`, `ap-southeast-2`
- **Cross-Region traffic** uses the AWS global backbone

### Partitions

AWS Regions are grouped into **partitions** — independent administrative boundaries:

- `aws` — standard commercial Regions
- `aws-cn` — China (Beijing, Ningxia), under Chinese law via local partners; **separate IAM, separate accounts**
- `aws-us-gov` — AWS GovCloud (US-East, US-West) — isolated, US-government workloads, FedRAMP High; **separate accounts**

ARNs include the partition: `arn:aws:s3:::bucket` vs `arn:aws-us-gov:s3:::bucket`.

---

## Availability Zones (AZs)

- One or more **discrete data centres** with redundant power, networking, and connectivity in a Region
- **Physically isolated** from other AZs (separate buildings, power feeds, network)
- Connected via **low-latency, high-throughput, redundant private fiber**
- Each Region has **at least 3 AZs** (most have 3–6)
- AZ names like `us-east-1a` are **account-specific** — `us-east-1a` in account A and account B may map to different physical AZs
- For cross-account alignment use **AZ IDs** (e.g., `use1-az1`)
- HA workloads span **≥ 2 AZs**; mission-critical span **≥ 3 AZs**

---

## Local Zones

- AWS-owned infrastructure in a **specific metro area**, extending a parent Region
- **Single-digit-millisecond latency** to end users in that metro
- Subset of services (EC2, EBS, RDS, ECS, EKS, ELB, etc.)
- See dedicated [AWS Local Zones](aws-local-zones.md) note

---

## Wavelength Zones

- AWS infrastructure embedded in **telco 5G networks**
- For ultra-low-latency mobile applications (AR/VR, live streaming, real-time gaming on phones)
- Partners: Verizon, KDDI, Vodafone, Bell Canada, SK Telecom, etc.
- **Carrier gateway** routes traffic to/from the 5G network

---

## AWS Outposts

- AWS hardware and services deployed **on customer premises** (your data center)
- For **data residency** or low-latency to on-prem systems
- Managed by AWS; connects back to the parent Region
- Variants: **Outposts rack** (42U cabinets), **Outposts servers** (1U / 2U)
- See dedicated [AWS Outposts](aws-outposts.md) note

---

## Edge Locations (Points of Presence / PoPs)

- **Hundreds** of AWS sites in major cities globally
- NOT for general compute / storage — used for **CloudFront**, **Lambda@Edge**, **CloudFront Functions**, **Route 53 DNS**, **AWS Shield / WAF edge protection**, **Global Accelerator**
- Reduce latency by serving cached content from the closest edge

---

## Regional Edge Caches

- Sit **between CloudFront Origin servers and Edge Locations**
- Larger cache than individual edge locations
- When data ages out at an edge, the next request can fetch from the Regional Edge Cache instead of the origin
- Lower latency, lower origin load
- Automatic — no configuration needed

---

## Lambda@Edge vs CloudFront Functions

| Feature               | **Lambda@Edge**                                                  | **CloudFront Functions**                       |
| --------------------- | ---------------------------------------------------------------- | ---------------------------------------------- |
| Runtime               | Node.js, Python                                                  | JavaScript only                                |
| Where it runs         | Regional Edge Caches                                             | Edge Locations (closer to user)                |
| Max execution time    | 5-30s by trigger                                                 | < 1 ms                                         |
| Max memory            | 128 MB - 10 GB                                                   | 2 MB                                           |
| Network / file system | Yes                                                              | No                                             |
| Cost                  | Higher                                                           | ~6× cheaper                                    |
| Triggers              | Viewer request, Origin request, Origin response, Viewer response | Viewer request, Viewer response only           |
| Use cases             | Heavy logic, external API calls, image manipulation              | URL rewrites, header manipulation, A/B routing |

- Both **require CloudFront**
- **Lightweight, ultra-fast** edge transforms → **CloudFront Functions**
- **Heavier logic** (auth tokens, API calls) → **Lambda@Edge**

---

## CloudFront

- AWS's global **CDN** — caches content at edge locations
- Origins: S3, custom HTTP servers, ALB, API Gateway, MediaPackage, Lambda function URLs, S3 static websites
- Integrates with **AWS WAF, Shield, ACM, Cognito**
- **Origin Shield** — additional caching layer between Regional Edge Caches and origins
- **Origin Access Control (OAC)** — secure S3 origin so only CloudFront can fetch (replaces older OAI)
- CloudFront distribution **tenants** for multi-tenant SaaS, enhanced security with managed rules, granular cache behaviors

---

## AWS Global Accelerator

- Static **anycast IPv4 + IPv6** addresses that route to the nearest AWS edge
- For **TCP and UDP** (CloudFront is HTTP-only)
- Backed by ALB, NLB, EC2 instances, or Elastic IPs
- **Health checks** at edge → automatic failover to healthy endpoints
- Lower latency, fewer hops than internet routing
- Use cases: gaming, IoT, voice/video, financial trading, multi-Region failover

---

## Picking the Right Edge Footprint

| Use case                                       | Service                             |
| ---------------------------------------------- | ----------------------------------- |
| Cache static / dynamic HTTP content            | **CloudFront**                      |
| Edge HTTP transforms (lightweight)             | **CloudFront Functions**            |
| Edge HTTP logic (heavier)                      | **Lambda@Edge**                     |
| TCP / UDP traffic acceleration with static IPs | **Global Accelerator**              |
| Latency-sensitive compute in a specific city   | **Local Zones**                     |
| Latency-sensitive compute on 5G                | **Wavelength Zones**                |
| Workloads on customer premises                 | **Outposts**                        |
| Global anycast DNS resolver                    | **Route 53 Global Resolver** (2026) |

---

## Exam Tips

- **HA design**: across **≥ 2 AZs** for standard HA, **≥ 3 AZs** for critical
- **DR design**: across **≥ 2 Regions** — backup & restore, pilot light, warm standby, multi-site active/active
- **AZ IDs vs AZ names** — for cross-account AZ alignment, use **AZ IDs** (e.g., `use1-az1`)
- **Edge locations ≠ AZs** — edge runs CloudFront / Lambda@Edge / Route 53 / Shield; AZs are full data centres
- **Regional Edge Cache** sits between origin and edge — reduces origin load
- **CloudFront Functions vs Lambda@Edge**: lightweight JS → CloudFront Functions; heavier Node/Python → Lambda@Edge
- **Global Accelerator vs CloudFront**: GA = TCP/UDP + static anycast IPs; CloudFront = HTTP caching
- **GovCloud and China are separate partitions** — separate IAM, separate accounts, separate ARNs
- **Service availability varies by Region** — verify before designing multi-Region

---

## Exam Traps

- **Edge locations are not AZs** — they cannot run general workloads, only CloudFront / Lambda@Edge / Route 53 / Shield
- **AZ names are account-specific** — `us-east-1a` in two accounts may map to different physical AZs; use **AZ IDs** for cross-account alignment
- **GovCloud and China accounts are completely separate** — IAM users do not work cross-partition
- **Newer services may not be in your chosen Region** — check Region availability before promising features
- **Lambda@Edge requires CloudFront** — cannot run without it
- **CloudFront Functions only runs JavaScript** at Viewer Request / Response — for Node.js / Python or origin events use Lambda@Edge
- **Global Accelerator is NOT a CDN** — does not cache content; it accelerates network routing
- **Cross-Region replication is NOT automatic** — design Region-to-Region replication (S3 CRR, DynamoDB Global Tables, Aurora Global Database, etc.)
- **Outposts are NOT a Region** — they extend a parent Region, sharing its API endpoints
