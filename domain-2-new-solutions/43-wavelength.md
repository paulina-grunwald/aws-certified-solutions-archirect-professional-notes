# AWS Wavelength

> **Compute + storage at the edge of telecom 5G networks. Wavelength Zones are AWS infrastructure deployed inside Verizon / KDDI / Vodafone / etc. 5G provider data centers, providing single-digit-millisecond latency to mobile devices on the carrier's network. Use for ultra-low-latency mobile workloads: AR/VR, real-time vehicle telemetry, multiplayer mobile games, ML at the edge. Lower SAP-C02 frequency but tested as the "ultra-low latency to 5G devices" answer.**

Maps to: **Domain 1.1 — Network connectivity**, **Domain 2.5 — High performance**, **Domain 4.3 — Modernization**

---

## Overview

- **AWS compute + storage** physically deployed in **5G carrier networks**
- **Wavelength Zone** = embedded AWS infrastructure inside a telco's metro
- Traffic from 5G devices reaches the Zone **without traversing the public internet**
- Result: **single-digit millisecond latency** to mobile users on that carrier
- Designed for **mobile-first ultra-low-latency** workloads

## Architecture

- **Wavelength Zone** = subset of EC2 / EBS / VPC services
- Each Zone is an **extension of a parent Region**
- Traffic from carrier-network devices → Wavelength → optionally to parent Region
- VPC spans parent Region + Wavelength Zone
- Subnet in the Zone is a "Wavelength subnet" with carrier IP range

## Services Available in Wavelength Zones

- EC2 (limited families: t3, r5, g4, m5)
- EBS (gp2)
- VPC + Wavelength subnet + carrier gateway
- ECS / EKS
- Some Lambda support (selected Regions)
- Limited compared to full Region — most other services live in parent Region

## Carrier Gateway

- Network gateway providing **internet egress** + **carrier-network ingress**
- Replaces internet gateway concept
- Routes traffic to/from mobile devices on the carrier's 5G network
- No public internet hop

## Use Cases

- **AR / VR** mobile apps requiring sub-20ms latency
- **Connected vehicle** telemetry + V2X (vehicle-to-everything)
- **Live broadcast** with on-site real-time processing
- **Smart manufacturing** with 5G-connected industrial IoT
- **Multiplayer mobile gaming** with low-latency player state sync
- **AI inference at the edge** (model serving close to user)

## Wavelength vs Local Zones vs Outposts vs Regions

| | Wavelength | Local Zone | Outposts | Region |
|---|---|---|---|---|
| Location | Inside 5G carrier metro | Edge city near user | On-prem (your DC) | AWS DC |
| Latency target | Sub-10ms to 5G device | Sub-10ms to city | Sub-ms on-prem | Region-typical |
| Audience | 5G mobile devices | Web users in city | On-prem workloads | All |
| Carrier traffic | Carrier-network only | Internet | Internet / on-prem LAN | Internet |
| AWS services available | Subset | Subset | Subset | All |

**Rule of thumb**:
- 5G mobile latency → **Wavelength**
- City-local web latency → **Local Zone**
- On-prem co-location → **Outposts**
- Cloud-native default → **Region**

## Carriers

- **US**: Verizon (Boston, NYC, Atlanta, Miami, DC, Charlotte, Dallas, Las Vegas, Houston, Phoenix, San Francisco, LA, Denver, Detroit, Chicago, Nashville, Minneapolis, Seattle)
- **Japan**: KDDI (Tokyo, Osaka)
- **South Korea**: SK Telecom (Seoul, Daejeon)
- **Europe**: Vodafone (London, Manchester, Birmingham, Düsseldorf), Bell Canada (Toronto, Montréal, Vancouver)
- Coverage expanding

## Networking & Connectivity

- **Carrier IPs** assigned to instances — reachable from carrier devices via carrier gateway
- **Parent Region VPC** — extend via subnet in Wavelength Zone
- **Direct path** to parent Region (AWS backbone)
- **Optional internet egress** through carrier gateway
- Cross-Zone traffic to other Wavelength Zones = via parent Region

## Pricing

- **Slight premium** over normal EC2 pricing in the parent Region
- Data transfer between Wavelength and parent Region: standard intra-Region
- Carrier gateway: usage-based
- EBS: standard rates

## Common Patterns

### AR mobile shopping
- App on 5G phone uses ML model to recognize products
- Model inference on Wavelength Zone EC2 g4 (GPU)
- 5ms round-trip vs 80ms+ from Region
- User sees instant AR overlay

### Connected vehicle telemetry
- Vehicle sends sensor data over 5G
- Wavelength Zone processes anomaly detection
- Critical events → vehicle in <10ms
- Aggregated data → parent Region for fleet analytics

### Live event streaming
- 5G cameras at venue
- Wavelength processes feeds with low latency
- Distribute to attendees on-site over 5G
- Cloud upload aggregated to Region

## Exam Tips

- "Ultra-low latency to 5G mobile devices" → **AWS Wavelength**
- "AR/VR on 5G phones" → **Wavelength**
- "Vehicle-to-cloud sub-10ms" → **Wavelength**
- Embedded in **telco carrier networks** (Verizon, KDDI, Vodafone, etc.)
- **Carrier gateway** replaces IGW for Wavelength subnets
- Limited service set vs full Region
- Wavelength ≠ Local Zone ≠ Outposts — different use cases

## Exam Traps

- **Wavelength serves 5G devices on the carrier** — not general internet users
- **Devices on Wi-Fi don't benefit** — must be on the carrier's 5G network
- **Limited service set** in Wavelength Zone — most services stay in parent Region
- **Not for on-prem latency** — that's Outposts
- **Not for city-wide web latency** — that's Local Zones
- **Pricing premium** over Region pricing
- **Cross-Wavelength communication goes via parent Region** — adds latency
- **Each carrier covers their own footprint** — multi-carrier requires multiple Wavelength Zones
- **Wavelength is Region-tied** — Boston Wavelength is tied to us-east-1
