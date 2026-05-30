# Elastic Load Balancing (ELB)

> **Four LB types: ALB (L7 HTTP/S/gRPC), NLB (L4 TCP/UDP/TLS with static IPs), Gateway Load Balancer (L3 traffic insertion for security appliances), Classic ELB (legacy). ALB: content-based routing, OIDC/Cognito auth, mTLS, Lambda targets, HTTP/3. NLB: ultra-low latency, static IPs, source IP preservation, security groups. GWLB: insert third-party firewalls/IDS into VPC traffic.**

Maps to: **Domain 2.5 — High-performance**, **Domain 1.1 — Networking**, **Domain 1.3 — Reliability**

---

## ELB Types

| LB | Layer | Protocols | Best for |
|---|---|---|---|
| **ALB** | L7 | HTTP, HTTPS, gRPC, WebSocket | Web apps, microservices, content routing |
| **NLB** | L4 | TCP, UDP, TLS | Ultra-low latency, static IPs, extreme scale |
| **GWLB** | L3 (IP) | All IP | Insert third-party security appliances |
| **CLB** | L4 + L7 | HTTP, HTTPS, TCP, SSL | **Legacy** |

> Default: **ALB for HTTP/S, NLB for TCP/UDP**. CLB is EOL-track.

---

## Core Components

- **Load Balancer** — internet-facing or internal
- **Listener** — port + protocol
- **Rules** — conditions + actions per listener (ALB)
- **Target Group** — targets (EC2 / IP / Lambda / container) + health checks
- **Targets** — instances / containers / IPs / functions

---

## ALB

### Routing Conditions

- **Host-based**, **Path-based**, **HTTP header**, **HTTP method**, **Query string**, **Source IP**
- **AND logic** to combine

### Target Types

- **Instance** (EC2)
- **IP** (VPC; reach on-prem via VPN / DX)
- **Lambda** (single function per target group)
- **ECS / Fargate**

### Authentication

- **OIDC** with any IdP
- **Cognito** user pools
- **mTLS** — client certificate authentication
- **JWT validation** offloaded from app to ALB

### Advanced

- **HTTP/2** + **HTTP/3**
- **gRPC** (server-side, HTTP/2)
- **WebSocket**
- **Sticky sessions** — duration cookie or app-controlled
- **Custom response** + **Redirect** actions
- **Weighted target groups** — blue/green, canary
- **Cross-zone LB ON by default** (free)
- **Slow start** for new targets

### Logging

- **Access logs** to S3 (5-min interval)
- **Connection logs** — TLS handshake detail
- **CloudWatch metrics** — request count, latency, target health

---

## NLB

### Highlights

- **Ultra-low latency** (~100 ms vs ALB ~400 ms)
- **Static IP per AZ** (one per subnet); **EIP** support
- **Millions of req/s scale**
- **Source IP preservation** natively
- **Cross-zone LB OFF by default** — opt-in per-GB charges

### Target Types

- **Instance** (EC2)
- **IP** (VPC, peered, on-prem via VPN/DX)
- **ALB** (NLB → ALB chaining for hybrid L4 + L7)

### Protocols

- **TCP, UDP, TCP_UDP** (same listener), **TLS** (termination at NLB)

### Security Groups

- **NLB supports SGs** on the LB itself
- **Preserve client IP** option — if disabled, SG rules see NLB ENI source

---

## GWLB

- **L3 / packet-level** LB
- **Insert third-party virtual appliances** (firewall, IDS/IPS, DPI)
- **GENEVE protocol** (port 6081)
- **Centralized inspection** — single point for many VPCs
- **Sticky flow** to one appliance per 5-tuple
- Pattern: **AWS Network Firewall** is AWS-managed; **GWLB** is the path for Palo Alto / Fortinet / Cisco / Check Point

---

## CLB (Legacy)

- Supports basic HTTP/S + TCP
- No content-based routing
- Never for new designs

---

## Health Checks

- HTTP/HTTPS / TCP / SSL/TLS / gRPC
- Configurable: protocol, port, path, interval (10–300s), timeout, healthy + unhealthy thresholds
- **Connection draining (CLB) / Deregistration delay (ALB, NLB)** — 0–3,600s (default 300s)

---

## Cross-Zone Load Balancing

| LB | Default | Pricing |
|---|---|---|
| **ALB** | ON | Free |
| **NLB** | OFF | Per-GB inter-AZ |
| **CLB** | OFF | Free when ON |

> **NLB cross-zone often the right answer** for even distribution; watch the cost.

---

## Use Cases

| Need | LB |
|---|---|
| HTTP/HTTPS web app | **ALB** |
| Microservices with path/host routing | **ALB** |
| Lambda HTTP backend (single function) | **ALB** with Lambda target |
| gRPC backend | **ALB** |
| WebSocket | **ALB** |
| TCP / UDP / TLS at extreme scale | **NLB** |
| Static IPs / EIPs | **NLB** |
| Source IP preservation natively | **NLB** |
| Insert third-party firewall / IDS | **GWLB** |
| AWS-managed L3/L4 firewall | **AWS Network Firewall** (not GWLB) |

---

## Exam Tips

- **ALB = L7**, **NLB = L4**, **GWLB = L3**, **CLB = legacy**
- **ALB content routing**: host / path / header / query / source IP / method
- **ALB target types**: instance, IP, Lambda, ECS / Fargate
- **ALB auth**: OIDC, Cognito, **mTLS**
- **ALB HTTP/3**, **HTTP/2 gRPC**
- **NLB static IPs per AZ** — IP allowlist + DNS A record stability
- **NLB Security Groups** — supported
- **NLB cross-zone LB OFF by default** — opt-in charges
- **GWLB** — insert third-party firewalls / IDS via GENEVE port 6081
- **Sticky sessions** — duration or app-controlled cookie
- **NLB → ALB chaining** for hybrid L4 + L7
- **Connection logs** on ALB

---

## Exam Traps

- **CLB doesn't support content-based routing** — use ALB
- **WAF does NOT integrate with NLB or CLB** — only ALB / CloudFront / API Gateway / AppSync
- **NLB SG support is newer** — old material may still say SGs are at targets only
- **NLB cross-zone LB OFF by default** — uneven distribution + charges if ON
- **GWLB ≠ AWS Network Firewall** — GWLB inserts third-party appliances; ANF is AWS-managed
- **ALB Lambda target invokes ONE function per target group**
- **NLB does NOT terminate HTTP/S** — use ALB or terminate at NLB-TLS then forward TCP
- **Static IPs are NLB-only** — ALB DNS resolves to changing IPs (use Global Accelerator for static)
- **Cross-Region failover** is Route 53 / Global Accelerator — NOT ELB
