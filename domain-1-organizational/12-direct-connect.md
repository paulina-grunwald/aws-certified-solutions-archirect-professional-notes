# AWS Direct Connect (and Hybrid Networking)

> **Dedicated private fiber from your data center to AWS. Low latency, high bandwidth, deterministic performance, predictable costs. Bypasses the public internet. Pair with VPN as backup, Transit Gateway for multi-VPC, Direct Connect Gateway for multi-Region.**

Maps to: **Domain 1.1 — Architect network connectivity strategies** (hybrid connectivity)

---

## Overview

**AWS Direct Connect (DX)** is a dedicated, private network connection between on-prem (or colocation) and AWS, via physical fiber circuit.

This note covers the complete **hybrid networking** picture: Direct Connect, Site-to-Site VPN, Transit Gateway, and Direct Connect Gateway.

---

## Connection Types

### Dedicated Connection
- Physical Ethernet port dedicated to one customer
- Capacities: **1, 10, 100, 400 Gbps**
- Provisioned by AWS, completed by a DX Partner
- Required for **MACsec** encryption (10/100/400 Gbps only)
- Required for **LAG** (Link Aggregation Group)

### Hosted Connection
- Provided through a DX Partner
- Capacities: **50 Mbps to 25 Gbps** (partner-dependent)
- Faster provisioning, scalable on demand
- **Cannot** use MACsec or LAG

> Provisioning a new dedicated connection often takes **a month or more**. Use VPN as interim/backup while DX is being provisioned.

---

## Virtual Interfaces (VIFs)

| VIF Type | Connects To | Use Case |
|---|---|---|
| **Private VIF** | Single VPC via **Virtual Private Gateway (VGW)** or **Direct Connect Gateway** | Private connectivity to private IPs in VPCs |
| **Public VIF** | AWS public services (S3, DynamoDB, KMS public endpoints) over private fiber, using public IPs | Reach AWS public services without internet |
| **Transit VIF** | **Direct Connect Gateway + Transit Gateway** | Hub-and-spoke to many VPCs and Regions through one TGW |

- VIFs are logical Layer-2 connections over the physical DX link
- BGP sessions per VIF for dynamic route exchange
- **BGP MD5 authentication** secures BGP sessions

---

## Direct Connect Gateway (DXGW)

- **Globally available** — create in any Region, access from all Regions
- Connects DX to **VPCs in multiple Regions** and multiple accounts
- Pairs with **Private VIFs** (to reach VGWs in any Region) OR **Transit VIFs** (to reach TGWs)
- For scale, use Transit Gateway + Transit VIF
- **DXGW + TGW + Transit VIF** is the most common SAP-C02 hybrid pattern

### SiteLink
- Direct connectivity **between DX locations** worldwide via AWS backbone
- Bypasses AWS Regions — DX location → AWS backbone → DX location
- Use case: connect on-prem branch offices via AWS instead of MPLS
- Billed per port hours + per GB

---

## Encryption on Direct Connect

DX traffic is **private but NOT encrypted on the wire by default**.

Three encryption options:
1. **MACsec** (Layer 2) — hardware-accelerated, near line-rate, encrypts all traffic including BGP / ARP. Dedicated 10/100/400 Gbps only
2. **Site-to-Site VPN over DX** (Public VIF + IPsec) — VPN tunnel uses DX as transport; encrypted end-to-end
3. **Application-layer TLS** — per-application encryption

### MACsec Details
- IEEE 802.1AE standard, Layer-2 encryption
- Configuration: **CAK** (Connectivity Association Key, 256-bit pre-shared), **CKN** (key name), derived **MACsec Secret Key**
- Modes: `must_encrypt`, `should_encrypt`, `no_encrypt`
- **Extended Packet Numbering (XPN)** required for 100/400 Gbps
- Both customer device AND AWS DX endpoint must support MACsec

---

## Direct Connect Resiliency Models

| Model | Setup | SLA |
|---|---|---|
| **Development / Test** | 1 connection at 1 DX location | None |
| **High Resiliency** | 2 connections at 2 DX locations | **99.9%** |
| **Maximum Resiliency** | 2 connections at each of 2 DX locations (4 total) | **99.99%** |
| **DX + VPN backup** | 1 DX + Site-to-Site VPN over internet | Lower SLA, cheaper |

- Mission-critical hybrid: **Maximum Resiliency** OR **DX + VPN backup**
- VPN backup goes over public internet — expect higher latency / jitter during failover

---

## Link Aggregation Group (LAG)

- Logical interface using **LACP** to bundle multiple **dedicated** connections at a single DX endpoint
- Up to **4 connections**, all same speed
- Treated as single managed connection
- All connections must terminate at the **same AWS device**
- **Dynamic LACP** (not static)

---

## Site-to-Site VPN

- Encrypted IPsec tunnel between customer gateway (CGW) and AWS over the **public internet**
- AWS side: **Virtual Private Gateway (VGW)** OR **Transit Gateway (TGW)**
- VGW attaches to **one VPC only**; AWS provides two tunnel endpoints (auto-failover)
- TGW attaches to many VPCs across many accounts
- **Accelerated Site-to-Site VPN** uses Global Accelerator edge for lower latency

### Use Cases
- Quick, cheap hybrid (minutes to provision)
- Backup for Direct Connect (encrypted failover path)
- Initial path while DX is being provisioned

---

## Transit Gateway (TGW)

- **Hub-and-spoke** for connecting many VPCs and on-prem networks
- Replaces many VPC peerings (which don't scale or transit)
- Attachments: VPCs, VPN, **DXGW via Transit VIF**, peering with other TGWs (cross-Region)
- Route tables control which spokes reach which
- Inter-Region TGW peering for global hub-and-spoke
- VPC peering is **NOT transitive** — TGW is the answer for many-VPC scenarios

---

## DX + TGW + DXGW (SAP-C02 standard pattern)

```
on-prem router  ──(DX dedicated link)──>  DX location
                                              │
                                       Transit VIF
                                              │
                                   Direct Connect Gateway
                                              │
                                  Transit Gateway (Region A)
                                       ╱   │   ╲
                                    VPC1 VPC2 VPC3
                                              │
                                  Transit Gateway peering
                                              │
                                  Transit Gateway (Region B)
                                       ╱   │   ╲
                                    VPC4 VPC5 VPC6
```

- One DX link → DXGW → Transit VIF → TGW(s) → many VPCs across many Regions / accounts
- Routes propagated via BGP from on-prem to TGW

---

## High Availability Best Practices

- Use **two CGWs** on-prem, each pointing to both VGW tunnel endpoints (4 tunnels total)
- Use **two DX locations** for diverse fiber paths
- Combine **DX + VPN backup** for cost-effective resilience
- **BGP** dynamic routing — let routes fail over automatically, not static
- Monitor with **CloudWatch metrics** (`ConnectionState`, `ConnectionBpsEgress`, BGP session status)
- For Region-scale events use **Route 53 ARC** for deterministic shift (not DNS health checks)

---

## Exam Tips

- **Dedicated ≥ 10 Gbps + need encryption** = **MACsec** (no app changes, line-rate)
- **DX is not encrypted by default** — for IPsec on top of DX, build VPN over a Public VIF
- **Long lead time (~1 month+) for new DX** — use **Site-to-Site VPN as interim**, switch to DX later
- **Multi-VPC, multi-Region from one DX** = **Direct Connect Gateway + Transit VIF + Transit Gateway**
- **Multiple VPCs in the same Region with DX** = TGW + Transit VIF (don't create many Private VIFs)
- **DX + VPN backup**: VPN encrypts failover path; routes failover via BGP
- **Maximum Resiliency** = 2 connections at each of 2 locations (4 total) — 99.99% SLA
- **Public VIF** reaches AWS public services (S3, DynamoDB) over private fiber with public IPs
- **LAG** bundles up to 4 same-speed dedicated connections via LACP
- **SiteLink** connects DX locations to each other via AWS backbone — for on-prem WAN replacement
- **Accelerated Site-to-Site VPN** uses Global Accelerator edges for faster VPN
- **BGP MD5 authentication** required for DX BGP sessions

---

## Exam Traps

- **DX traffic is private but NOT encrypted on the wire** — common distractor. "Encrypted in transit" requires MACsec, VPN-over-DX, or app-layer TLS
- **VGW attaches to ONE VPC only** — for many VPCs use TGW
- **Private VIF goes to a VGW or DXGW**, not directly to a VPC — VGW is the anchor
- **Gateway Endpoints (S3 / DynamoDB) are NOT reachable from on-prem** — use **Interface Endpoint + Private VIF** instead
- **VPC peering is NOT transitive** — for many-VPC + on-prem use TGW
- **MACsec NOT available on hosted connections** — dedicated 10/100/400 Gbps only
- **Lead time for new DX is long** (weeks to months) — answer for "fast hybrid" is VPN
- **DXGW is global, but VIFs are regional** — Private VIF attaches to a VGW in one Region; Transit VIF to a TGW in one Region
- **DX does NOT terminate in a VPC subnet** — it terminates at the DX link / VIF; routing into VPC is via VGW or DXGW + TGW
- **Site-to-Site VPN over the public internet does NOT use DX** — only when VPN is built over a DX Public VIF does the IP path traverse DX

---

## Quick Reference

| Need | Service / Pattern |
|---|---|
| Cheap, fast hybrid | Site-to-Site VPN |
| Dedicated, low-latency, high-bandwidth | Direct Connect |
| Encrypted dedicated | MACsec on DX (≥10 Gbps) OR VPN over DX |
| Multi-VPC, multi-Region from one DX | DXGW + Transit VIF + TGW |
| DX failover | Two DX connections at two locations OR DX + VPN backup |
| On-prem WAN over AWS backbone | DX SiteLink |
| Lower-latency VPN | Accelerated Site-to-Site VPN |
