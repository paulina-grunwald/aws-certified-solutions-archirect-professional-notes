# Amazon Route 53

> **Highly available, scalable, fully managed authoritative DNS. Three main functions: domain registration, DNS routing, and health checking. The answer for global traffic steering, hybrid DNS, and deterministic multi-Region failover.**

Maps to: **Domain 1.1 — Architect network connectivity strategies** (DNS / global routing)

---

## Overview

- **Global service** (not region-scoped)
- Supports **public** and **private hosted zones**
- **100% availability SLA**
- Supports **DNSSEC** for signing and validation
- **Hosted zone** = DNS container for a domain and its subdomains, holding the records that tell the internet (or VPC) where to send traffic

---

## DNS Record Types

| Record Type | Description |
|---|---|
| **A** | Maps hostname to IPv4 |
| **AAAA** | Maps hostname to IPv6 |
| **CNAME** | Maps hostname to another hostname. **Cannot be used for zone apex** (e.g., `example.com`) |
| **Alias** | Route 53-specific. Maps hostname to AWS resource. **Works at zone apex**. Free for alias queries to AWS resources. Native health-check evaluation |
| **NS** | Name servers for the hosted zone |
| **MX** | Mail exchange |
| **TXT** | Text records (SPF, DKIM, domain verification) |
| **SRV** | Service locator |
| **CAA** | Certificate Authority Authorization |
| **DS** | Delegation Signer (DNSSEC) |
| **NAPTR** | Name Authority Pointer |

### Alias vs CNAME

| Feature | Alias | CNAME |
|---|---|---|
| Zone apex (naked domain) | Yes | **No** |
| Charge for queries | Free (to AWS resources) | Standard DNS charges |
| Health check integration | Native (Evaluate Target Health) | Not native |
| Target | AWS resources only | Any hostname |
| Record type | A or AAAA | CNAME |
| TTL | Set by Route 53 automatically | Configurable |

**Alias targets supported**: ELB, CloudFront, API Gateway, Elastic Beanstalk, S3 website endpoints, VPC Interface Endpoints, Global Accelerator, another Route 53 record in the same zone, **VPC Lattice service custom domain endpoints**, **CloudFront distribution tenants**.
**Cannot alias to**: EC2 DNS name directly.

---

## Hosted Zones

### Public Hosted Zones
- Records for internet-facing traffic
- Auto-created when registering a domain via Route 53
- Can be created for domains registered with other registrars

### Private Hosted Zones
- Records for traffic **within one or more VPCs**
- Must be associated with at least one VPC
- VPC must have `enableDnsHostnames` AND `enableDnsSupport` = true
- **Cross-account**: share via **Route 53 Profiles** or VPC associations via CLI/SDK (not Console)
- Enables **split-view DNS** — same domain in public and private with different records

### Cross-Account DNS Delegation
- Use NS records in parent hosted zone to delegate subdomains to other AWS accounts
- Common pattern in AWS Organizations multi-account setups

---

## Routing Policies

### Simple
- Maps a domain to one or more values; if multiple, client picks at random
- **No health checks** when multiple values are configured
- Use for a single resource

### Weighted
- Send a percentage of traffic to each resource (weights are **relative**, not required to sum to 100)
- Weight 0 = stop sending; all weights 0 = equal distribution
- Supports health checks
- Use cases: blue/green, A/B, gradual migration

### Latency-Based
- Routes to the AWS Region with **lowest latency** for the user
- Latency is measured between user and **AWS Regions**, not specific endpoints
- Can be combined with health checks for failover

### Failover
- Active-passive: **Primary** + **Secondary** records
- Primary must have a health check
- If primary fails, Route 53 auto-routes to secondary
- DR pattern (DNS-driven, TTL-bound — see ARC for deterministic alternative)

### Geolocation
- Routes by **user's geographic location** (continent, country, US state)
- Must create a **default** record for unmatched IPs
- Specificity wins: US state > country > continent > default
- Use cases: content localization, regional restrictions

### Geoproximity (Traffic Flow only)
- Routes by geographic location of users AND resources
- **Bias** values (-99 to +99) shift traffic between resources
- Positive bias = expand region from which traffic flows to a resource; negative = shrink
- Resources can be AWS (specify Region) or non-AWS (specify lat/long)
- Requires **Route 53 Traffic Flow**

### Multivalue Answer
- Returns up to **8 healthy records** at random per query
- Each record can have its own health check
- **Not a substitute for ELB** — this is client-side load balancing

### IP-Based
- Routes by **client IP address (CIDR collections)**
- Map CIDRs to specific records
- Use cases: route specific ISPs/enterprises to specific endpoints, optimize network paths

---

## Health Checks

### Types
1. **Endpoint health checks** — monitor an IP or domain
2. **Calculated** — parent monitors child checks via AND / OR / threshold (e.g., 2 of 3 healthy)
3. **CloudWatch alarm** — health check tied to an alarm state

### Endpoint Health Check Details
- ~15 global health checkers probe the endpoint
- Healthy/unhealthy threshold: default 3 (configurable 1-10)
- Interval: 30s default (10s for higher cost)
- Protocols: HTTP, HTTPS, TCP
- HTTP/HTTPS: pass if 2xx or 3xx within 4s
- String match against response body (first **5120 bytes** only)
- Allow Route 53 health checker IPs in firewalls / SGs

### Private Resources
- Route 53 health checkers are **outside the VPC** — cannot reach private endpoints directly
- **Solution**: CloudWatch metric → CloudWatch alarm → CloudWatch-alarm-based health check

---

## Route 53 Resolver (Hybrid DNS)

DNS resolution between on-prem and AWS VPCs.

### Inbound Endpoints
- On-prem DNS resolvers forward queries **into** Route 53 Resolver via ENIs in your VPC
- Use case: on-prem needs to resolve AWS **private hosted zone** records

### Outbound Endpoints
- Route 53 Resolver forwards queries **out** to on-prem DNS resolvers
- Uses **Resolver rules** (conditional forwarding for specific domains)
- Use case: AWS resources need to resolve on-prem domains

### Resolver Rules
- **Forwarding rules**: for specific domains, forward to target IPs (on-prem DNS)
- **System rules**: exceptions to forwarding (e.g., for AWS private hosted zones)
- **Shareable across accounts via AWS RAM** — key for multi-account hybrid DNS

### Route 53 Profiles
- Shareable configuration bundle for Route 53 across multiple VPCs / accounts
- Can include: private hosted zone associations, Resolver rules, DNS Firewall rule groups, DNSSEC validation settings, **Resolver query logging configs**
- Can associate VPC interface endpoints
- Available in an expanding set of Regions

---

## Route 53 Global Resolver

Internet-reachable **anycast DNS resolution** globally — an enterprise-managed alternative to public resolvers like 1.1.1.1 or 8.8.8.8.

- Single set of **IPv4 and IPv6 anycast IPs** — routes queries to the nearest AWS Region
- Supports **DNS-over-HTTPS (DoH)** and **DNS-over-TLS (DoT)** in addition to standard UDP (Do53)
- Resolves both **public internet domains** AND **Route 53 private hosted zones** — eliminates split-DNS forwarding
- Integrated **DNS Firewall** with AWS Managed Domain Lists (malware, phishing, content filtering) AND **Dictionary-based DGA protection** at GA
- **Centralized query logging** built-in
- Deploy across **2+ AWS Regions** for HA with automatic failover
- Available in **30 Regions** at GA
- Use case: secure DNS for distributed workforce, branch offices, IoT, mobile fleets

---

## DNSSEC

### Signing (Authoritative)
- Supported for **public hosted zones**
- Route 53 manages the **Zone Signing Key (ZSK)**
- You provide the **Key Signing Key (KSK)** via **AWS KMS in us-east-1**, asymmetric, `ECC_NIST_P256`
- Establish **chain of trust** by adding the **DS record** to the parent zone

### Validation (Resolver)
- Route 53 Resolver can validate DNSSEC signatures for VPC queries
- Enable per-VPC or via Route 53 Profiles
- Failed validation → SERVFAIL

---

## DNS Firewall

Filter outbound DNS traffic from your VPCs.

- **Rule groups** with ordered rules → domain lists + actions (ALLOW / BLOCK / ALERT)
- Block responses: NODATA, NXDOMAIN, or custom OVERRIDE
- **AWS Managed Domain Lists**: malware, botnet, phishing, content filtering
- **DNS Firewall Advanced** — real-time anomaly detection:
  - **DNS Tunneling** — data exfiltration via DNS queries
  - **Domain Generation Algorithm (DGA)** — malware-generated random domains
  - **Dictionary-based DGA** — DGA variant using dictionary word concatenations, mimicking legit domains; GA in all Regions including GovCloud
  - Configurable confidence: High / Medium / Low
  - Detects on query length, entropy, frequency — catches **previously unknown** domains, not just known-bad lists
- Rule groups shareable via **AWS RAM**, includable in **Route 53 Profiles**
- Failure mode: Open (allow) or Closed (block)

---

## Query Logging

- **Public hosted zones**: CloudWatch Logs in **us-east-1**
- **Resolver query logs**: CloudWatch Logs, S3, or Kinesis Data Firehose
- Resolver query log configs **shareable via AWS RAM**, **includable in Route 53 Profiles**
- Use case: investigation, compliance, malware / exfiltration detection (pair with DNS Firewall Advanced)

---

## Route 53 Application Recovery Controller (ARC)

Deterministic, low-latency, operator-controlled failover for multi-Region apps — more reliable than DNS health-check-driven failover because routing is explicit.

- **Readiness checks** — continuously verify standby Region matches active (capacity, config parity, quotas)
- **Routing controls** — simple ON/OFF switches per Region, evaluated by Route 53 health checks; flipping a control updates DNS responses instantly
- **Safety rules** — enforce invariants ("at least one Region ON", "only one ON at a time") — prevent total outage or split-brain
- **Cluster** = 5 Regional endpoints (Region-redundant control plane); use any endpoint during outages
- **Zonal Shift / Zonal Autoshift** — evacuate a single AZ from ALB / NLB / ECS / EKS without flipping the whole Region
- Use case: active-active multi-Region with one-click controlled failover; active-passive DR; AZ evacuation under impairment

---

## Route 53 Traffic Flow

- Visual editor for complex routing configurations
- **Traffic policies** (versioned) and **policy records**
- Supports all routing policies and can combine them (latency + weighted + failover)
- Reusable, versioned policies; policy records apply policies to hosted zones
- Cost: per policy record per month
- **New Traffic Flow console experience**

---

## Domain Registration

- Register and manage domains
- Transfer in/out, auto-renewal
- **Registrar lock** (transfer lock) to prevent unauthorized transfers
- Support for `.ai` and other newer TLDs

---

## Key Integrations

| Target | Record Type |
|---|---|
| S3 static website (bucket name must match domain) | Alias to S3 website endpoint |
| CloudFront | Alias to distribution domain |
| ELB (ALB / NLB / CLB) | Alias to load balancer DNS |
| API Gateway | Alias to regional or edge endpoint |
| Elastic Beanstalk | Alias to environment URL |
| Global Accelerator | Alias to accelerator DNS |
| VPC Interface Endpoints | Alias |
| VPC Lattice | Alias to service custom domain endpoints |

---

## Exam Tips

- **Zone apex / naked / root domain** = **Alias**, never CNAME
- **Failover routing requires a health check** on the primary — without it Route 53 cannot detect failure
- **Geolocation vs Geoproximity**: Geolocation = by user location (country/continent); Geoproximity = by proximity with adjustable **bias** for gradual shifts
- **Private hosted zone + on-prem**: use Resolver **inbound endpoints** so on-prem can resolve AWS private zones; **outbound endpoints** so VPC resources can resolve on-prem
- **Health checks for private resources**: use **CloudWatch-alarm-based health checks** (R53 checkers can't reach private endpoints)
- **Multivalue answer ≠ ELB** — returns up to 8 healthy IPs; client picks; no real load balancing
- **Latency-based** measures latency to **AWS Regions**, not endpoints
- **IP-based routing** — deterministic routing by client CIDR collections
- **Weighted with weight=0** stops traffic; useful during maintenance or migrations
- **DNSSEC KSK** must use **KMS in us-east-1**, asymmetric **ECC_NIST_P256**
- **Resolver rules shared via AWS RAM** — key for multi-account hybrid DNS
- **DNS Firewall Advanced** detects DNS tunneling, DGA, **Dictionary DGA** — answer for DNS exfiltration / unknown malicious domains
- **Multi-Region failover that must be deterministic** → **Route 53 ARC routing controls + safety rules**, not pure failover routing
- **Zonal Shift / Zonal Autoshift** = AZ-level evacuation for ALB / NLB / ECS / EKS — first-line response for single-AZ impairment
- **Global Resolver** = anycast public + private resolver, replaces 1.1.1.1 / 8.8.8.8 for fleet; **Resolver endpoints** = hybrid DNS forwarding to/from on-prem

---

## Exam Traps

- **CNAME at zone apex** is invalid — any answer using CNAME for root domain is wrong
- **Geolocation without a default record** — unmatched users get no response
- **Geolocation ≠ Latency** — Geolocation strictly uses location; an Australian user goes to AU even if another Region has lower latency
- **Alias evaluates target health automatically** (Evaluate Target Health flag); different from an explicit attached health check
- **Private hosted zone requires** `enableDnsHostnames` AND `enableDnsSupport` = true on the VPC
- **Resolver endpoints ≠ health check endpoints** — Resolver = hybrid DNS forwarding, not resource monitoring
- **Public hosted zone query logs only go to CloudWatch Logs in us-east-1** — don't pick "any Region"
- **Route 53 ARC vs failover routing**: failover routing = automatic, DNS-health-check driven, TTL-bound (minutes); ARC routing controls = operator-controlled, deterministic, instant — pick ARC for "controlled / orchestrated / strict invariants"
- **Zonal Shift evacuates a single AZ**, not a whole Region; works for ALB / NLB / ECS / EKS, not all services
- **Global Resolver ≠ Resolver Inbound/Outbound** — Global Resolver is internet-facing anycast (replaces 8.8.8.8); Resolver endpoints are hybrid DNS for VPC ↔ on-prem
- **Calculated health checks** use threshold logic (e.g., 2 of 3 healthy) — exam may present AND/OR scenarios
- **Traffic Flow costs money** — policy records billed monthly; simple routing does NOT require Traffic Flow
- **Route 53 Profiles vs direct VPC association** — Profiles scale better for multi-account / multi-VPC; the exam may test which is more operationally efficient
- **Alias TTL is set automatically** by Route 53 — not manually configurable like CNAME TTL
