# VPC Peering (+ Transit Gateway recap)

> **VPC Peering — point-to-point network connection between two VPCs using private IPs over AWS backbone. Same or different accounts; same or cross-Region. NOT transitive (A↔B, B↔C does NOT mean A↔C). No bandwidth bottleneck. CIDRs MUST NOT overlap. For more than a handful of VPCs use Transit Gateway; for service-level exposure use PrivateLink.**

Maps to: **Domain 1.1 — Inter-VPC connectivity**, **Domain 1.3 — Resilience patterns**

---

## Overview

- **Point-to-point** private connection between **2 VPCs**
- Traffic over **AWS backbone** — no internet, no VPN, no bottleneck
- **Same or different accounts**; **same or cross-Region**
- Private IPs only
- **VPC CIDRs must NOT overlap**
- Free for cross-AZ in same Region; cross-Region peering incurs inter-Region transfer

---

## Architecture

```mermaid
flowchart LR
    VPCA[VPC A<br/>10.0.0.0/16] -- peering --- VPCB[VPC B<br/>192.168.0.0/16]
    VPCA -. peering .- VPCC[VPC C<br/>172.16.0.0/16]
    VPCB -. NO transitive route .x VPCC
```

A ↔ B and A ↔ C exist; B ↔ C still requires its own peering.

---

## Setup

1. **Requester** sends peering request from VPC A
2. **Accepter** (other account / Region) accepts
3. Update **route tables** in BOTH VPCs for the peer's CIDR
4. Adjust **security groups** to allow traffic (peer SG by ID within Region)
5. Optional: **DNS resolution** across peering (opt-in)

---

## Key Rules

- **CIDRs MUST NOT overlap** — even partially
- **NOT transitive** — peering A↔B↔C does NOT make A↔C
- **Cannot route 0.0.0.0/0 over peering** for internet egress
- **NAT / IGW / VGW are NOT shared** across peering
- **Edge-to-edge limitations** — peer A's VPN cannot egress B (and vice versa)
- **SG rules by SG ID** work only within the same Region; cross-Region uses CIDRs

---

## Cross-Region Peering

- Cross-Region peering supported
- Traffic encrypted in transit on AWS backbone
- IPv6 supported
- Per-GB inter-Region transfer charges

---

## Inter-VPC Comparison

| Need | Service |
|---|---|
| 2 VPCs, full IP routing | **VPC Peering** |
| Many VPCs + on-prem hub-and-spoke | **Transit Gateway** |
| Expose a single service | **PrivateLink + VPC Endpoint Service** |
| Cross-Region private at scale | **TGW peering** OR **Cloud WAN** |

---

## Transit Gateway (TGW) — recap

> Detailed in [12-direct-connect.md](12-direct-connect.md).

- **Hub-and-spoke** — many VPCs + VPN + DX through one TGW
- **Transitive routing** (subject to route tables)
- **TGW peering** for inter-Region
- **Multi-account** via AWS RAM sharing
- Up to **5,000 attachments**; 50 Gbps per attachment (ECMP for VPN)
- Use when: > ~5 VPCs, multi-Region, hybrid + multi-VPC together, multi-account hub
- Trade-off: TGW hourly fee + data processing (peering cheaper for 2 VPCs)

### TGW Network Manager + Cloud WAN

- **Network Manager** — central observability for TGW + DX + VPN
- **AWS Cloud WAN** — managed global WAN; simpler than DIY TGW peering mesh

---

## Decision Matrix

| Scenario | Best fit |
|---|---|
| 2 VPCs, full IP routing | **VPC Peering** |
| 5+ VPCs in a Region | **TGW** |
| Multi-Region hub-and-spoke | **TGW peering** OR **Cloud WAN** |
| SaaS service exposure | **PrivateLink** |
| Hybrid + many VPCs | **DXGW + TGW + Transit VIF** |
| Overlapping CIDRs | **PrivateLink** (peering / TGW won't work) |

---

## Cost

- **VPC peering**: data transfer per GB (cross-AZ + cross-Region); no per-hour fee
- **TGW**: per-attachment hourly + per-GB data processing
- For 2–4 VPCs in one Region, peering cheaper
- For 5+ VPCs or multi-Region, TGW is the right answer (operationally + financially)

---

## Exam Tips

- **VPC Peering is NOT transitive** — common exam trap
- **CIDRs must NOT overlap** — design upfront for future peering
- **Update route tables in BOTH VPCs**
- **Cross-account peering**: requester → accepter
- **Cross-Region peering** supported
- **SG ID references work cross-peering within the same Region**; cross-Region needs CIDR
- **For > ~5 VPCs use TGW** — peering becomes O(n²)
- **VPC peering connects only 2 VPCs**
- **PrivateLink for SaaS exposure** — doesn't require non-overlapping CIDRs
- **TGW peering** for multi-Region; **Cloud WAN** for global managed WAN

---

## Exam Traps

- **"A↔B↔C means A↔C"** = WRONG. **Peering is NOT transitive**
- **Overlapping CIDRs**: peering / TGW won't work; use **PrivateLink**
- **Cannot route 0.0.0.0/0 over peering** for internet egress
- **NAT / IGW / VGW NOT shared** — peer can't use your NAT for internet
- **Edge-to-edge**: peer A's VPN cannot carry peer B's traffic — use TGW
- **DNS resolution across peering is opt-in**, both sides must enable
- **TGW does NOT solve overlapping CIDRs** — use PrivateLink (or re-IP one side)
- **SG references by SG ID** work cross-peering only within the same Region
