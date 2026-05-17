# VPC

> **Logically isolated virtual network in AWS, scoped to a single Region.**

Maps to: **Domain 1.1 — Architect network connectivity strategies**

---

## VPC Fundamentals

- A VPC is a logically isolated virtual network in AWS, scoped to a single Region
- Default VPC created per Region with /16 CIDR, one /20 public subnet per AZ, internet gateway, default route table, default NACL, and default security group
- VPC CIDR: /16 (largest) to /28 (smallest); can add up to 5 secondary CIDRs
- Supports IPv4 and IPv6 (dual-stack); IPv6 CIDRs are /56 from Amazon pool or BYOIP
- Tenancy: default (shared hardware) or dedicated (single-tenant); set at VPC creation and **cannot be changed back** from dedicated
- DNS: `enableDnsSupport` (Route 53 Resolver at VPC+2 address) and `enableDnsHostnames` (public DNS for instances) — both needed for VPC endpoints to work with private DNS

---

## Subnets

- Subnet is tied to a single AZ; **cannot span AZs**
- AWS reserves 5 IPs per subnet: network address, VPC+1 (DNS), VPC+2 (future), VPC+3 (future), broadcast
- **Public subnet**: route table has `0.0.0.0/0` → Internet Gateway; instances need public/Elastic IP
- **Private subnet**: no direct internet route; uses NAT Gateway/Instance for outbound
- Each subnet must be associated with exactly one route table and one NACL

---

## Internet Gateway (IGW)

- Horizontally scaled, redundant, HA — no bandwidth constraints
- One IGW per VPC
- Performs NAT for instances with public IPv4 addresses
- Supports IPv4 and IPv6; for IPv6-only outbound, use **Egress-Only Internet Gateway**

---

## NAT Gateway

- Managed, **highly available within a single AZ** (deploy one per AZ for HA)
- Supports 5 Gbps, auto-scales to 100 Gbps
- Uses Elastic IP; charged per hour + per GB processed
- Cannot be used by instances in the same subnet (must be in a different subnet)
- NAT Gateway does **NOT** support security groups; use NACLs on the subnet
- **NAT Gateway vs NAT Instance**: NAT Gateway is managed, higher bandwidth, HA within AZ, no bastion capability; NAT Instance is self-managed EC2, can be used as bastion, supports port forwarding, cheaper for low traffic
- **Private NAT Gateway**: for VPC-to-VPC traffic via Transit Gateway (no EIP, no IGW needed), useful for **overlapping CIDR** scenarios

---

## Elastic IP Addresses

- Static public IPv4 address; limited to 5 per Region by default
- Can be moved between instances/ENIs for failover
- Charged when NOT associated with a running instance
- As of February 2024, AWS charges for **all public IPv4 addresses** ($0.005/hr) including those on running instances

---

## Route Tables

- Main route table auto-assigned to subnets not explicitly associated
- Custom route tables for specific routing needs
- **Most specific route wins** (longest prefix match)
- Local route (VPC CIDR) cannot be removed
- Can propagate routes from VPN/Direct Connect via Virtual Private Gateway
- **Edge association**: route table associated with IGW or VGW for middlebox routing

---

## Security Groups vs NACLs

### Security Groups
- **Stateful** — return traffic automatically allowed
- Applied at ENI level (instance level)
- **Allow rules only**; no deny rules
- All rules evaluated before deciding
- Can reference other security groups (including cross-account with VPC peering)
- Default: all outbound allowed, all inbound denied

### Network ACLs (NACLs)
- **Stateless** — must explicitly allow both inbound and outbound (including ephemeral ports 1024–65535)
- Applied at subnet level
- Support **allow AND deny** rules
- Rules evaluated in order (lowest number first); first match wins
- Default NACL allows all; custom NACL denies all by default
- One NACL per subnet; one subnet can only have one NACL
- Use NACLs to **block specific IPs** (cannot do this with security groups alone)

---

## VPC Peering

- Connect two VPCs privately using AWS network (same or different accounts/Regions)
- **NOT transitive**: A↔B and B↔C does NOT mean A↔C
- **No overlapping CIDRs** allowed
- Must update route tables on both sides
- Security group references work cross-account within same Region
- Cross-Region peering: supported but security group cross-referencing not supported
- DNS resolution: can enable to resolve private DNS hostnames of peered VPC
- Data transfer charges apply for cross-Region peering
- **Limitation**: does not support edge-to-edge routing (cannot route through a peered VPC's IGW, NAT, VPN, or Direct Connect)

---

## Transit Gateway (TGW)

- Regional **hub-and-spoke** model: connect thousands of VPCs, VPNs, Direct Connect, and peered TGWs
- Supports **transitive routing** (unlike VPC peering)
- **Inter-Region TGW peering**: connect TGWs across Regions (static routes only, no route propagation)
- Works with **RAM** (Resource Access Manager) for cross-account sharing
- **Route tables**: supports multiple route tables for network segmentation (e.g., shared services vs isolated VPCs)
- **Appliance mode**: ensures symmetric routing when using network appliances (stateful firewalls)
- **Multicast support**: only service on AWS that supports multicast
- **ECMP**: Equal-Cost Multi-Path routing for aggregating VPN bandwidth (multiple VPN tunnels)
- **TGW Connect**: GRE tunnel support for SD-WAN integration; higher bandwidth than IPsec (up to 5 Gbps per Connect peer, max 4 peers = 20 Gbps)
- Charges: per attachment per hour + per GB data processed
- **When to use**: many VPCs needing transitive connectivity, centralized egress/ingress, complex routing, VPN aggregation

---

## VPC Endpoints

### Gateway Endpoints
- **S3 and DynamoDB only**
- **Free**; no data processing charges
- Added as a route table entry (prefix list)
- **Cannot be extended outside VPC** (no VPN, peering, TGW access)
- One per VPC per service; use endpoint policies for access control

### Interface Endpoints (AWS PrivateLink)
- ENI with private IP in your subnet
- Supports most AWS services and custom services
- Per-hour + per-GB charges
- Can be accessed from on-premises via VPN/Direct Connect or via VPC peering/TGW
- Private DNS enabled by default (resolves public service endpoint to private IP)
- Supports security groups for access control
- Can use endpoint policies

### Gateway Load Balancer Endpoint (GWLBE)
- For traffic inspection with third-party appliances (firewalls, IDS/IPS)
- Works with Gateway Load Balancer in a service provider VPC
- Uses **GENEVE encapsulation** (port 6081)

---

## AWS PrivateLink

- Expose a service from your VPC to other VPCs securely (one-way access)
- Service provider creates NLB (or GWLB); consumer creates Interface Endpoint
- No VPC peering, TGW, or public internet needed
- Works with **overlapping CIDRs**
- Scales to thousands of consumer VPCs
- **When to use**: SaaS connectivity, exposing services to partners/customers, keeping traffic private, avoiding transitive routing complexity

---

## VPC Peering vs Transit Gateway vs PrivateLink — Decision Matrix

| Scenario | Best Option |
|---|---|
| Two VPCs, direct low-latency connection | VPC Peering |
| Many VPCs needing any-to-any connectivity | Transit Gateway |
| Centralized egress/ingress through firewall | Transit Gateway |
| Expose a specific service to consumers | PrivateLink |
| On-premises to multiple VPCs | Transit Gateway + VPN/DX |
| Overlapping CIDRs between VPCs | PrivateLink (or Private NAT GW + TGW) |
| Cross-Region VPC connectivity | Inter-Region TGW Peering or Cross-Region VPC Peering |
| Third-party SaaS integration privately | PrivateLink (via AWS Marketplace) |
| Need transitive routing | Transit Gateway (peering is NOT transitive) |

---

## VPC Flow Logs

Capture IP traffic **metadata** at VPC, subnet, or ENI level. Three destinations (CloudWatch Logs, S3, Amazon Data Firehose), 1-min or 10-min aggregation, **metadata only** — no payload.

→ Full coverage: [vpc-flow-logs.md](vpc-flow-logs.md)

---

## AWS Network Firewall

- Managed **stateful firewall** service deployed in its own subnet within VPC
- Supports **Suricata**-compatible IPS/IDS rules
- Operates at layers 3–7: IP, port, protocol, domain filtering, TLS SNI inspection
- Centralized deployment via Transit Gateway (**inspection VPC** pattern)
- Integrates with Firewall Manager for multi-account governance
- Logging: alert and flow logs to CloudWatch, S3, or Kinesis; CloudWatch dashboards for visibility
- **Flow capture and flow flush** (2025): capture active flow metadata for monitoring; selectively terminate flows during security incidents
- Supports both stateful and stateless rule groups
- **When to use**: domain filtering, IPS/IDS, centralized inspection, Suricata rules, compliance requirements

---

## VPC Lattice (2023–2025)

- **Application-layer service mesh** for service-to-service communication
- Simplifies connectivity across VPCs and accounts without TGW, PrivateLink, or NAT
- Supports HTTP/HTTPS, gRPC, and TCP protocols
- Key concepts:
  - **Service Network** — logical grouping
  - **Service** — a single application
  - **Target Group** — instances, IPs, Lambda, ALB
- **Resource Gateway** (2024–2025): enables access to TCP resources (databases, DNS names, IPs) across VPCs/accounts
  - Configurable IP addresses per ENI for resource gateways (Oct 2025)
- **Service Network VPC Endpoint**: powered by PrivateLink, allows connectivity from on-premises via DX/VPN
- Auth: IAM policies + security groups; supports SigV4 signing
- Automatically handles service discovery, load balancing, and connectivity
- Integrates with Network Firewall for combined service-level + network-level security
- **When to use**: microservices architecture, cross-account service connectivity, replacing complex TGW+PrivateLink setups for app-layer routing

---

## VPC Encryption Controls (Nov 2025)

- New capability to audit and enforce **encryption in transit** within and across VPCs in a Region
- Modes:
  - **Monitor** — audit existing traffic encryption
  - **Enforce** — block unencrypted traffic
- Applied at VPC level; works for traffic between VPCs and within a VPC
- Uses **hardware-level encryption on Nitro-based instances**
- Useful for compliance requirements (HIPAA, PCI-DSS, financial regulations)
- Charged per hour for VPCs with encryption controls enabled (free introductory period ended March 2026)

---

## Direct Connect with VPC

- **Private VIF**: access VPC resources via Virtual Private Gateway (single VPC) or Direct Connect Gateway (multiple VPCs, multiple Regions)
- **Transit VIF**: access VPCs via Transit Gateway through Direct Connect Gateway
- **DX Gateway**: global resource that allows a single DX connection to reach VPCs in multiple Regions
- **DX + VPN**: IPsec encryption over DX for added security (lower throughput due to encryption overhead)

---

## VPN Connectivity

- **Site-to-Site VPN**: IPsec tunnel over internet; VGW or TGW on AWS side, Customer Gateway on-premises
- **Accelerated VPN**: uses AWS Global Accelerator for improved performance; only with TGW (not VGW)
- **Client VPN**: managed OpenVPN for remote user access to AWS and on-premises networks
- VPN over TGW supports **ECMP** for bandwidth aggregation (up to 50 Gbps with multiple tunnels)

---

## Cross-Region and Cross-Account Patterns

- **Cross-account VPC sharing** (RAM): share subnets from an Organizations-managed VPC; participants launch resources in shared subnets
- **Cross-Region**: TGW peering (static routes), VPC peering, PrivateLink (via inter-Region peering), DX Gateway
- **Centralized VPN**: TGW with VPN attachments shared across accounts via RAM
- **Centralized internet egress**: NAT Gateway in shared VPC, route through TGW
- **Centralized inspection**: Network Firewall in a dedicated inspection VPC with TGW routing

---

## Exam Tips

- If the question mentions **"transitive routing"** or connecting many VPCs → **Transit Gateway**, not VPC peering
- **"Expose a service to thousands of consumers"** or **"vendor provides service"** → **PrivateLink** (NLB + Interface Endpoint)
- **S3/DynamoDB private access with no extra cost** → **Gateway Endpoint** (free, route-table based)
- **"Access AWS services from on-premises"** → **Interface Endpoint** (accessible via DX/VPN); Gateway Endpoints are NOT accessible from outside VPC
- **"Overlapping CIDRs"** between two VPCs needing connectivity → **PrivateLink** or **Private NAT Gateway + TGW**
- **"Block a specific IP address"** → **NACL** (security groups have allow-only rules)
- **"Inspect all traffic between VPCs"** → **Network Firewall** in centralized inspection VPC with TGW
- **"Multicast"** → **Transit Gateway** (only AWS service supporting multicast)
- **"Increase VPN bandwidth"** → **ECMP over TGW** (multiple VPN tunnels; NOT supported with VGW)
- **VPC Lattice questions** → application-layer service mesh replacing complex TGW+PrivateLink for microservices
- **"Encryption in transit within VPC"** + compliance → **VPC Encryption Controls** (2025 feature)
- When two connectivity options seem valid, compare cost: VPC peering is cheapest for 2-VPC scenarios; TGW charges per attachment + data

---

## Exam Traps

- **VPC peering is NOT transitive** — if the answer chains peering connections for A→B→C routing, it is wrong
- **Gateway Endpoints cannot be accessed from outside the VPC** (VPN, DX, peering, TGW) — only Interface Endpoints can
- **NAT Gateway is AZ-scoped** — deploying one NAT GW is NOT highly available; need one per AZ
- **Security groups cannot deny traffic** — if the question requires blocking specific IPs, you need NACLs
- **NACLs are stateless** — forgetting to allow ephemeral ports (1024–65535) on outbound/inbound will break traffic
- **Interface Endpoint ≠ free** — they have hourly + data charges; Gateway Endpoints (S3, DynamoDB) are free
- **PrivateLink requires NLB** (or GWLB), NOT ALB — if an answer says "ALB + PrivateLink," it is wrong
- **TGW inter-Region peering uses static routes only** — no dynamic route propagation across Regions
- **VPC Encryption Controls require Nitro instances** — older instance types do not support hardware encryption
- **VPC Lattice is NOT a network-layer solution** — it operates at application layer (L7); for L3/L4 inspection use Network Firewall
