# Podcast 06 — VPC Connectivity Patterns

**Length target**: 15 min (~2,200 spoken words)
**Repo references**: `domain-1-organizational/08-vpc.md` (TGW + PrivateLink + Endpoints), `10-aws-privatelink.md`, `32-vpc-peering.md`

## Topic & Scope

Connecting VPCs together and reaching AWS services privately. The decision matrix between VPC Peering, Transit Gateway, PrivateLink, and VPC Endpoints is the highest-yield SAP-C02 networking knowledge.

## Service Coverage Depth

**Deep**:
- VPC Peering — 1:1, non-transitive, simple
- Transit Gateway — hub-and-spoke, route tables, attachments
- PrivateLink (VPC Interface Endpoints) — one-way service exposure
- VPC Endpoints — Gateway (S3, DynamoDB) vs Interface (everything else)

**Brief**:
- VPC Lattice (mention as the "service-to-service mesh across VPCs" newer option)

## Structured Outline

1. **Open (30s)** — "Four ways to connect VPCs. Picking the wrong one = wasted money or broken architecture."
2. **VPC Peering (3 min)** — 1:1 connection, manual routing, no transitive, same/cross-Region/cross-account
3. **Transit Gateway (4 min)** — hub-and-spoke, scales to thousands of VPCs, TGW route tables, attachments (VPC, VPN, DX, peering), inter-region peering
4. **PrivateLink (3 min)** — expose service in your VPC via NLB to other VPCs via Interface Endpoint, one-way, no peering / no overlapping CIDR issues
5. **VPC Endpoints (3 min)** — Gateway (S3, DynamoDB — free, route table entry) vs Interface (any AWS service — ENI + DNS, hourly cost), endpoint policies
6. **Decision Matrix (1.5 min)**
7. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- VPC Peering: 1:1, non-transitive, same or cross-Region or cross-account
- Peering requires non-overlapping CIDRs and explicit route table entries
- Transit Gateway: hub-and-spoke for many VPCs, transitive routing through TGW route tables
- TGW supports VPC, VPN, Direct Connect Gateway, and TGW peering attachments
- TGW inter-region peering: encrypted over AWS backbone
- PrivateLink exposes a service from one VPC to consumer VPCs via NLB + Interface Endpoint
- PrivateLink does NOT require non-overlapping CIDRs (uses DNS name + private IPs in consumer VPC)
- Gateway VPC Endpoint: S3 + DynamoDB only, FREE, adds prefix list to route table
- Interface VPC Endpoint: ENI in your subnet, supports most AWS services + Marketplace, hourly + data processing charges
- Endpoint policies further restrict what the endpoint allows
- Resource shares of TGW + Resolver rules via RAM
- VPC Lattice: application-layer service-to-service mesh — newer option for HTTP/gRPC across VPCs/accounts

## Must-Mention Exam Traps

- EXAM TRAP: VPC Peering is NOT transitive — A↔B + B↔C does NOT give A↔C
- EXAM TRAP: VPC Peering with overlapping CIDRs is impossible
- EXAM TRAP: Transit Gateway route tables are DIFFERENT from VPC route tables — both must be configured
- EXAM TRAP: TGW attachment costs are hourly + data — at scale TGW gets expensive vs PrivateLink for one-service exposure
- EXAM TRAP: PrivateLink is ONE-WAY — provider exposes, consumer connects; reverse needs another endpoint
- EXAM TRAP: Gateway Endpoint is FREE; Interface Endpoint costs hourly per AZ + per-GB
- EXAM TRAP: Gateway Endpoint only works for S3 and DynamoDB — everything else is Interface Endpoint
- EXAM TRAP: VPC Endpoint policy + IAM identity policy + bucket policy ALL apply — most restrictive wins
- EXAM TRAP: Direct Connect Gateway allows DX to reach multiple VPCs across Regions — but TGW is the modern path
- EXAM TRAP: PrivateLink doesn't traverse the public internet — but you still pay for endpoint hours + data
- EXAM TRAP: For S3 from many VPCs across Regions, use S3 Multi-Region Access Points + Gateway Endpoints — not PrivateLink

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| Connect 2 VPCs, same workload, simple | VPC Peering |
| Connect 50+ VPCs in mesh | Transit Gateway |
| Expose one service to 100 consumer VPCs | PrivateLink (Interface Endpoint) |
| Access S3 / DynamoDB from private subnet without NAT | Gateway VPC Endpoint |
| Access any AWS API privately from VPC | Interface VPC Endpoint |
| Overlapping CIDRs need to talk | PrivateLink (peering won't work) |
| Hybrid + multi-VPC + multi-account routing | Transit Gateway |

## Tone & Style

- Frame as four-way decision tree
- Use the "non-transitive" property of peering as the recurring trap
- For Gateway vs Interface Endpoint: "S3 + DynamoDB are special — they get free Gateway Endpoints. Everything else is Interface."

## Rapid-Fire Closer

"Peering transitive?" — "NO." "Free endpoint for S3?" — "Gateway Endpoint." "Expose one service to many VPCs?" — "PrivateLink." "50 VPCs to connect?" — "Transit Gateway." "Overlapping CIDRs?" — "PrivateLink, not peering."
