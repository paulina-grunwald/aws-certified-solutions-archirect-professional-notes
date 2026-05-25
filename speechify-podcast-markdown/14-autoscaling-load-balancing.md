# Podcast 14 — Auto Scaling & Load Balancing

**Length target**: 15 min
**Repo references**: `domain-2-new-solutions/33-ec2-auto-scaling.md`, `29-elb.md`, `domain-3-continuous-improvement/08-application-auto-scaling.md`

## Topic & Scope

EC2 Auto Scaling Groups + Application Auto Scaling + Elastic Load Balancing. SAP-C02 tests which scaling service applies to which resource, and the four ELB types.

## Service Coverage Depth

**Deep**:
- EC2 Auto Scaling — launch templates, scaling policies, lifecycle hooks, instance refresh
- Application Auto Scaling — DynamoDB, Aurora replicas, ECS service, Lambda PC, etc.
- ELB types: ALB, NLB, GWLB, Classic
- Target groups, health checks, deregistration delay

**Brief**:
- AWS Auto Scaling (unified scaling plans console)
- Predictive scaling

## Structured Outline

1. **Open (30s)** — "Three different 'Auto Scaling' services sound the same. Plus four load balancer types. Untangle them now."
2. **EC2 Auto Scaling (4 min)** — ASG, launch templates, three scaling types (target tracking, step, simple), lifecycle hooks, instance refresh, mixed instance policies, warm pools
3. **Application Auto Scaling (3 min)** — for DynamoDB / Aurora replicas / ECS / Lambda PC / SageMaker / Spot Fleet / AppStream / Comprehend / Neptune / ElastiCache / Keyspaces
4. **ELB Types (4 min)** — ALB (HTTP/HTTPS, content routing, WAF-compatible), NLB (TCP/UDP, static IP, ultra-low-latency), GWLB (security appliance insertion), Classic (legacy)
5. **Target Groups + Health Checks (1.5 min)** — multiple TGs per ALB, health check types, deregistration delay (connection draining), cross-zone load balancing
6. **AWS Auto Scaling + Predictive (1 min)** — unified console, ML-based forecast
7. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- EC2 Auto Scaling: ASG with launch template + min/max/desired
- Scaling policies: target tracking (simplest), step scaling, simple scaling, scheduled scaling
- Lifecycle hooks: pause launch/terminate to run setup/cleanup
- Instance refresh: rolling replacement after launch template change
- Mixed instances policy: combine On-Demand + Spot in one ASG
- Warm pools: pre-initialized instances to reduce scale-out latency
- Application Auto Scaling supports: DynamoDB, Aurora Replicas, ECS, Lambda Provisioned Concurrency, SageMaker, Spot Fleet, AppStream, Comprehend, Neptune, ElastiCache, Keyspaces, EMR, Custom resources
- ALB: HTTP/HTTPS, path/host/header routing, WebSockets, can route to EC2/IP/Lambda/containers
- NLB: TCP/UDP/TLS, static IPs per AZ, ultra-low-latency, preserves source IP
- GWLB: insert third-party network appliances (firewalls, IDS), uses GENEVE protocol
- Classic Load Balancer: legacy, avoid for new
- Target groups: separate per protocol/port, supports lambda targets (ALB only)
- Health checks: TCP / HTTP / HTTPS with path; configurable thresholds
- Deregistration delay: connection draining window (default 300s)
- Cross-zone load balancing: ALB always on (free); NLB off by default (paid when on)
- Predictive scaling: ML-based forecast for cyclical workloads, available for EC2 ASG and AWS Auto Scaling plans

## Must-Mention Exam Traps

- EXAM TRAP: EC2 Auto Scaling ≠ Application Auto Scaling — first is EC2 ASGs; second is for non-EC2 resources
- EXAM TRAP: AWS Auto Scaling is the unified CONSOLE — most use per-service Application Auto Scaling directly
- EXAM TRAP: DynamoDB on-demand mode auto-scales internally — Application Auto Scaling only configures provisioned mode
- EXAM TRAP: ALB does NOT preserve source IP by default — uses X-Forwarded-For header; NLB preserves natively
- EXAM TRAP: ALB cannot do static IP — use Global Accelerator + ALB, or use NLB
- EXAM TRAP: NLB cross-zone is OFF by default — enabling costs extra
- EXAM TRAP: WAF attaches to ALB but NOT NLB
- EXAM TRAP: GWLB inserts third-party appliances — not by itself a firewall
- EXAM TRAP: Aurora Auto Scaling scales REPLICAS, not the writer — for writer scaling, use Aurora Serverless v2
- EXAM TRAP: ASG health checks include EC2 status + ELB health — must enable ELB health checks explicitly for ELB-based replacement
- EXAM TRAP: Spot in ASG via mixed instances policy — pure Spot ASG is risky for prod
- EXAM TRAP: Compute Optimizer recommends ASG sizing — don't confuse with Auto Scaling itself

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| Scale EC2 instances by CPU / requests | EC2 Auto Scaling target tracking |
| Scale DynamoDB capacity | Application Auto Scaling |
| Scale Aurora read replicas | Application Auto Scaling |
| Scale Lambda Provisioned Concurrency | Application Auto Scaling |
| HTTP load balancing with path-based routing | Application Load Balancer |
| TCP/UDP ultra-low-latency LB | Network Load Balancer |
| Static IPs for whitelisting | NLB (or Global Accelerator + ALB) |
| Insert firewall appliance inline | Gateway Load Balancer |
| Lambda as target | ALB |
| Preserve source IP natively | NLB |

## Tone & Style

- Three "Auto Scaling" framing — emphasize they are different
- Repeat ALB = HTTP, NLB = TCP/UDP throughout
- Use concrete examples: "Lambda PC scaling? Application Auto Scaling"

## Rapid-Fire Closer

"Scale DynamoDB?" — "Application Auto Scaling." "Static IP LB?" — "NLB." "Path-based routing?" — "ALB." "Insert third-party firewall?" — "GWLB." "Aurora writer scale?" — "Serverless v2, NOT replicas."
