# AWS Site-to-Site VPN

> **IPsec tunnels between AWS VPC and on-premises (or another VPC / cloud). Two tunnels per VPN connection for HA. ~1.25 Gbps per tunnel (~2.5 Gbps aggregate). Static or BGP dynamic routing. Accelerated VPN routes through Global Accelerator. Attaches to VGW (single VPC) or Transit Gateway (multi-VPC, ECMP up to 50 Gbps). Fast setup; cheaper than Direct Connect; encrypted by design.**

Maps to: **Domain 1.1 — Hybrid connectivity**, **Domain 1.3 — Reliable / resilient**

---

## Overview

- **IPsec tunnel** over public internet between AWS and remote network
- Endpoints: **AWS VPC ↔ on-premises**, **VPC ↔ VPC**, **AWS ↔ other cloud**
- **2 tunnels per VPN connection** by default (HA, different AZs)
- **~1.25 Gbps per tunnel** (~2.5 Gbps aggregate)
- **Encrypted by design** — IPsec with AES-256, SHA-2
- **Fast setup** (minutes); cheaper than DX; ideal for migration / failover / non-critical
- For higher bandwidth + lower jitter → **Direct Connect**; for hybrid + encryption → **VPN over DX**

---

## Architecture

```mermaid
flowchart LR
    onprem[On-prem router<br/>Customer Gateway] -- IPsec tunnel 1 --> vgw[VGW or TGW]
    onprem -- IPsec tunnel 2 --> vgw
    vgw --> vpc[VPC]
```

---

## Components

| Component | Description |
|---|---|
| **Customer Gateway (CGW)** | AWS-side representation of on-prem device; public IP, ASN, optional cert |
| **Customer Gateway Device** | Physical / virtual router (Cisco, Juniper, pfSense, OpenSwan) |
| **Virtual Private Gateway (VGW)** | VPN endpoint on a single VPC; configurable BGP ASN |
| **Transit Gateway (TGW)** | VPN endpoint for multi-VPC + ECMP |
| **VPN Connection** | Pair of IPsec tunnels between CGW and VGW / TGW |

---

## Routing Options

| Routing | Behavior |
|---|---|
| **Static** | Manually configure VPC route table to VGW for on-prem CIDR; no BGP |
| **Dynamic (BGP)** | CGW and AWS exchange routes via BGP; automatic failover on tunnel failure |

> **Use BGP for production** — automatic failover + route propagation.

---

## High Availability Patterns

### 2-tunnel HA (default)

- AWS provides **2 tunnel endpoints** in different AZs
- Configure both on CGW for **Active/Active** (BGP routes both, ~2× throughput) or **Active/Passive** (BGP `local-preference`)

### Multiple CGWs

- Deploy **2 CGW devices** in the data center, each with its own VPN connection
- **4 tunnels total**; BGP for routing

### TGW VPN with ECMP

- Transit Gateway VPN attachments support **ECMP (Equal-Cost Multi-Path)**
- Spread traffic across parallel tunnels → up to **50 Gbps aggregate** per attachment
- **BGP required**

---

## Accelerated Site-to-Site VPN

- Traffic enters at **AWS edge** via Global Accelerator anycast
- Routed over AWS backbone to TGW endpoint
- **Lower jitter + higher consistency**
- Charged additionally
- **TGW-attached VPN only**

---

## IKE / IPsec Parameters

- **IKE versions**: IKEv1 + IKEv2 (IKEv2 preferred)
- **AES**: AES128-GCM-16, AES256-GCM-16 (AEAD), AES128-CBC, AES256-CBC
- **Integrity**: SHA1, SHA-256, SHA-384, SHA-512
- **DH groups**: 2, 5, 14–18, 19–21, 22–24 (22–24 = ECDSA)
- **Pre-Shared Key (PSK)** default; **Certificate-based auth** via ACM Private CA (since 2021)
- **Dead Peer Detection (DPD)** — auto-reset tunnel on failure
- **Rekey margins** + **lifetime** configurable

---

## Throughput

- **~1.25 Gbps per tunnel** (~2.5 Gbps aggregate per VPN connection)
- **TGW VPN + ECMP** — up to **50 Gbps aggregate** across multiple connections
- For higher consistent throughput → **Direct Connect** (1 / 10 / 100 Gbps dedicated)

---

## CloudHub Pattern

- Multiple S2S VPNs to a single VGW
- VGW acts as a hub — branches communicate via AWS
- Hub-and-spoke for remote sites
- BGP routing recommended

---

## Pricing

- **Per VPN connection hour** (~$0.05/hr)
- Data transfer out to internet (over the encrypted tunnel)
- **Accelerated VPN**: extra $0.018/GB
- Much cheaper than DX for low-bandwidth / temporary

---

## S2S VPN vs Direct Connect

| Feature | **S2S VPN** | **DX** |
|---|---|---|
| Transport | Public internet (IPsec) | Private fiber via AWS partner |
| Throughput | ~2.5 Gbps/VPN; 50 Gbps via TGW ECMP | 1 / 10 / 100 Gbps dedicated |
| Latency | Variable (internet) | Consistent low |
| Setup | Minutes | Weeks–months |
| Cost | Low | High |
| Encryption | Yes (IPsec) | No by default — combine with VPN over DX |
| Failover | Built-in 2 tunnels | Use 2nd DX or VPN backup |

> Common pattern: **DX steady-state + S2S VPN backup** (or VPN over DX for encrypted private).

---

## Exam Tips

- "Encrypted hybrid connectivity, fast to set up" → **Site-to-Site VPN**
- "High-bandwidth, dedicated, predictable" → **Direct Connect**
- "Encryption over Direct Connect" → **VPN over DX** (private VIF + IPsec)
- "VPN backup for DX" → standard pattern; route via BGP preference
- "Higher throughput VPN to multiple VPCs" → **TGW + ECMP** (up to 50 Gbps)
- "Lower latency / less jitter for VPN" → **Accelerated Site-to-Site VPN**
- "BGP dynamic routing" — preferred for production
- **2 tunnels per VPN connection** by default — configure both for HA
- **CGW = on-prem device representation in AWS** (public IP + ASN)
- **VGW for single VPC; TGW for multi-VPC + ECMP**
- **PSK default; certificate-based auth** via ACM Private CA (more secure)
- **CloudHub** = hub-and-spoke S2S VPN to a single VGW

---

## Exam Traps

- **S2S VPN throughput is ~1.25 Gbps per tunnel** — not the answer for 10+ Gbps; use DX
- **DX is NOT encrypted by default** — combine with VPN over DX
- **Single tunnel = no HA** — always configure both
- **Static routing has NO automatic failover** — use BGP for production
- **VGW does NOT connect multiple VPCs** — use TGW
- **TGW VPN ECMP requires BGP** — won't work with static
- **Accelerated VPN only works with TGW**, not VGW
- **VPN data transfer is NOT free** — outbound charges apply
- **CGW public IP must be static, routable** — NAT'd devices need a dedicated public IP
- **DH group 1 + SHA1 are deprecated** — use AES256-GCM, SHA-256+, DH 14+
