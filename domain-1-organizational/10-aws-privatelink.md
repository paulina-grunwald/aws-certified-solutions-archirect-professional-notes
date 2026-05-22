# AWS PrivateLink

> **Privately connect your VPC to AWS services, third-party SaaS services, and your own services across accounts — without VPC peering, NAT, IGW, or internet exposure. Traffic stays on the AWS backbone.**

Maps to: **Domain 1.1 — Architect network connectivity strategies** (private service connectivity)

---

## Overview

- Highly available, scalable technology to **privately connect your VPC** to:
  - AWS services (S3, DynamoDB, KMS, SQS, SNS, etc.)
  - Services in **other AWS accounts** (VPC endpoint services)
  - **AWS Marketplace** partner / SaaS services
- **No VPC peering, NAT, IGW, or public IPs required**
- Provider side: **Network Load Balancer (NLB)** — ALB is **not** supported
- Consumer side: **Elastic Network Interface (ENI)** with private IP in the consumer VPC
- Multi-AZ NLB + multi-AZ ENI = fault-tolerant deployment

> If a question mentions exposing a service to **tens, hundreds, or thousands of customer VPCs** — the answer is **AWS PrivateLink**, not VPC peering.

---

## Why Not VPC Peering?

- VPC peering opens the **entire VPC network** — fine for two-party use, but not for SaaS-style fan-out
- Peering at scale means many bilateral relationships with overlap risk
- PrivateLink exposes **only the specific service** (not the whole VPC) and scales to thousands of consumers

---

## VPC Endpoint Types

| Type | Services | Cost | On-Prem Access | Mechanism |
|---|---|---|---|---|
| **Gateway Endpoint** | S3, DynamoDB **only** | Free | No | Route table entry |
| **Interface Endpoint** | Most AWS services + custom + Marketplace | Paid (hourly + GB) | Yes (via Private VIF / VPN) | ENI with private IP |
| **Gateway Load Balancer Endpoint (GWLBE)** | Third-party virtual appliances | Paid | Yes | Inline traffic inspection (IDS/IPS/firewalls) |

---

## DNS and Private DNS

- Interface Endpoints get a **private DNS hostname** that resolves to the ENI's private IP
- Enable **Private DNS** on the endpoint so existing SDK/CLI calls (default service URL like `s3.us-east-1.amazonaws.com`) automatically route through the endpoint — **no code changes**
- Requires **DNS resolution** AND **DNS hostnames** enabled on the VPC
- Across **VPC peering**, the **Private DNS name does NOT propagate** — peers must use the endpoint-specific DNS name

---

## Security Groups and Endpoint Policies

- **Interface Endpoints** support **security groups** — control which resources can reach the endpoint
- **Gateway Endpoints** do NOT support security groups — only endpoint policies
- **Endpoint policies** are resource-based — restrict which principals and actions are permitted through the endpoint
- Endpoint policies are **additive with IAM** — both must allow the request
- Use bucket policy with `aws:sourceVpce` condition to **restrict S3 access to traffic from your VPC endpoint only**

---

## On-Premises Access via Direct Connect / VPN

- **Gateway Endpoints are NOT reachable from on-prem** — heavy exam distractor
- For on-prem → AWS service privately:
  1. Provision **Interface Endpoint** in the VPC
  2. Establish **Direct Connect with Private VIF** (or VPN) to that VPC
  3. On-prem traffic routes via Direct Connect → VPC → Interface Endpoint → AWS service
- This keeps the **entire path private** — no internet at any hop

---

## Gateway Load Balancer Endpoint (GWLBE)

- Pairs with **Gateway Load Balancer** to insert **third-party virtual appliances** (firewalls, IDS/IPS) inline
- Traffic transparently forwarded to the appliance fleet, then returned to its original destination
- Common pattern: **centralized inspection VPC** with GWLBE fanning traffic to appliance fleet
- Used for compliance scenarios requiring **inline packet inspection**

---

## Cross-Region and Cross-VPC

- Interface Endpoint **must be in the same Region** as the service it exposes
- For multi-Region access: provision endpoints in each Region
- Cross-VPC consumption via **VPC peering** works for the endpoint **as long as you use the endpoint-specific DNS name** (Private DNS doesn't traverse peering)

---

## Decision Matrix

| Scenario | Use |
|---|---|
| Expose a SaaS service to many customer VPCs | **PrivateLink (Interface Endpoint Service)** |
| EC2 → S3, in-VPC only, low cost | **Gateway Endpoint** (free) |
| On-prem → S3 via Direct Connect | **Interface Endpoint + Private VIF** |
| Centralized inline traffic inspection | **GWLBE + Gateway Load Balancer** |
| Full network connectivity between two VPCs | **VPC Peering** or **Transit Gateway** |

---

## Exam Tips

- **"Hundreds/thousands of VPCs"** + service exposure = **PrivateLink**, not VPC peering
- **Gateway Endpoints are free** — prefer them for S3 / DynamoDB when on-prem access is not needed
- **Direct Connect / VPN to AWS services** always needs an **Interface Endpoint** — Gateway Endpoints don't work from on-prem
- **Private VIF** is required on the Direct Connect side to reach Interface Endpoints
- **NLB on provider + ENI on consumer** — NLB only, ALB not supported
- Multi-AZ NLB + multi-AZ ENI = fault-tolerant PrivateLink
- Enable **Private DNS** so SDKs / CLIs need no code changes
- Endpoint policies + `aws:sourceVpce` in bucket policy = restrict S3 access to VPC-only
- **GWLBE** is the answer for inline inspection / IDS / IPS / 3rd-party firewall appliances

---

## Exam Traps

- **Gateway Endpoints are NOT reachable from on-prem** — heavy distractor. For Direct Connect / VPN to S3 use **Interface Endpoint**
- **PrivateLink ≠ VPC Peering** — PrivateLink exposes one service; peering opens the entire VPC network
- **ALB cannot back a PrivateLink endpoint service** — only NLB is supported
- **Interface Endpoint must be in the same Region** as the service — no cross-Region endpoints
- **DNS hostname AND DNS resolution must be enabled** on the VPC, or Private DNS won't work
- **Endpoint policies don't override IAM** — both must allow the action
- **GWLBE ≠ Interface Endpoint** — GWLBE is for traffic inspection, not service access
- **Private DNS name does NOT propagate across VPC peering** — peers must use the endpoint-specific DNS name
