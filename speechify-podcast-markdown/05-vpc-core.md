# Podcast 05 — VPC Core

**Length target**: 15 min (~2,200 spoken words)
**Repo references**: `domain-1-organizational/08-vpc.md`, `09-vpc-flow-logs.md`

## Topic & Scope

The VPC building blocks: subnets, gateways, route tables, security groups, NACLs. The fundamentals every networking question assumes you know cold.

## Service Coverage Depth

**Deep**:
- VPC CIDR + subnet design
- Internet Gateway, NAT Gateway, NAT Instance
- Route tables — main vs custom
- Security Groups (stateful) vs NACLs (stateless)
- VPC Flow Logs — capture levels, destinations

**Brief**:
- Elastic IPs, ENIs (mentioned)
- VPC Lattice (named, deferred to deeper coverage if exam-relevant)

## Structured Outline

1. **Open (30s)** — "VPC is the backbone of every AWS workload. Master the building blocks first; the connectivity patterns are next podcast."
2. **VPC + CIDR + Subnets (3 min)** — RFC1918 ranges, /16 typical, public vs private subnets defined by route, AZ-pinned
3. **Gateways (3 min)** — IGW (bidirectional, public), NAT Gateway (egress only, managed, AZ-resident), NAT Instance (legacy DIY), egress-only IGW for IPv6
4. **Route Tables (2 min)** — main vs custom, longest prefix match, default route 0.0.0.0/0
5. **Security Groups vs NACLs (3 min)** — SG stateful at ENI level, NACL stateless at subnet level, deny-only in NACL, allow-only in SG
6. **VPC Flow Logs (2 min)** — VPC/subnet/ENI level, ACCEPT/REJECT/ALL, destinations: CloudWatch Logs / S3 / Firehose
7. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- VPC CIDR: between /16 (65k IPs) and /28 (16 IPs), RFC1918 private ranges
- A subnet is "public" only because its route table has a route to IGW
- NAT Gateway is managed, scales to 100 Gbps, AZ-resident (deploy per-AZ for HA)
- NAT Instance is legacy — managed by you, requires source/dest check disabled
- IGW is regional, attached to one VPC at a time
- Route table: longest prefix match; default route 0.0.0.0/0 is the catch-all
- Security Group: allow rules only, stateful, attached to ENIs
- NACL: allow + deny rules, stateless, attached to subnets, rule order matters (lowest number wins)
- Default NACL allows all in + out; custom NACL denies all by default
- VPC Flow Logs capture metadata (NOT packet contents) — every accept/reject/all
- Flow Logs destinations: CloudWatch Logs, S3, Kinesis Data Firehose
- AWS reserves 5 IPs per subnet (network, VPC router, DNS, future use, broadcast)

## Must-Mention Exam Traps

- EXAM TRAP: NAT Gateway is per-AZ — single NAT GW is a SPoF for cross-AZ workloads
- EXAM TRAP: Security Group is stateful — return traffic is allowed automatically
- EXAM TRAP: NACL is stateless — must allow ephemeral return ports explicitly
- EXAM TRAP: NACL rules are evaluated in number order — lowest wins
- EXAM TRAP: An SG cannot DENY — only allow
- EXAM TRAP: A subnet is private/public based on routing, not naming
- EXAM TRAP: VPC peering does not provide transitive connectivity — A→B and B→C doesn't give A→C
- EXAM TRAP: Flow Logs do NOT capture packet contents — for that, use traffic mirroring
- EXAM TRAP: Flow Logs do NOT capture some traffic (DHCP, license activation, metadata, DNS queries to Amazon)
- EXAM TRAP: IGW is required for bidirectional internet — egress-only IGW is IPv6-only
- EXAM TRAP: NAT Gateway costs include hourly + per-GB processing — large data egress through NAT GW is expensive
- EXAM TRAP: VPC endpoints (Gateway for S3/DynamoDB; Interface for everything else) bypass NAT GW for AWS service traffic — major cost saver

## Key Decision Matrix

| Need | Pick |
|---|---|
| Stateful firewall at instance | Security Group |
| Stateless deny at subnet | NACL |
| Outbound-only internet for private subnet | NAT Gateway |
| Inbound + outbound internet | Internet Gateway |
| Log every packet's metadata | VPC Flow Logs |
| Log packet CONTENT | Traffic Mirroring |
| Private S3 / DynamoDB access | Gateway VPC Endpoint |
| Private access to any AWS API | Interface VPC Endpoint (PrivateLink) |

## Tone & Style

- Repeat "SG = stateful, NACL = stateless" four times
- Use a concrete 10.0.0.0/16 VPC with two AZs as the running example
- Explicitly contrast NAT Gateway vs NAT Instance

## Rapid-Fire Closer

"SG stateful?" — "YES." "NACL stateful?" — "NO." "SG can deny?" — "NO." "NAT GW per-AZ?" — "YES." "Flow Logs capture content?" — "NO."
