# Podcast 13 — Compute & Purchasing

**Length target**: 15 min
**Repo references**: `domain-2-new-solutions/01-ec2.md`, `02-ec2-image-builder.md`, `03-compute-optimizer.md`, `domain-1-organizational/45-savings-plans.md`

## Topic & Scope

EC2 instance families + purchasing options. Picking the right mix of On-Demand, Spot, Savings Plans, RIs is heavily tested. Also: Image Builder + Compute Optimizer.

## Service Coverage Depth

**Deep**:
- EC2 instance families (general / compute / memory / storage / accelerated)
- Purchasing options: On-Demand, Spot, Savings Plans (Compute, EC2 Instance, SageMaker), RIs (Standard, Convertible), Dedicated Hosts vs Dedicated Instances
- Image Builder (golden AMI pipeline)
- Compute Optimizer (right-sizing recommendations)

**Brief**:
- Capacity Reservations (mention as distinct from RI/SP)
- Graviton (mention as cost/perf option)

## Structured Outline

1. **Open (30s)** — "Compute pricing is the most common cost-optimization question. Get the mental model first."
2. **EC2 Families (3 min)** — general purpose (M, T), compute (C), memory (R, X, U), storage (I, D), accelerated (P, G, F, Trn, Inf), Graviton (A1, M6g, etc.)
3. **On-Demand vs Spot (2.5 min)** — On-Demand (full price, no commitment), Spot (up to 90% off, 2-min interruption notice), Spot Fleet, Capacity-Optimized allocation strategy
4. **Savings Plans (3 min)** — Compute SP (most flexible: EC2 + Fargate + Lambda, up to 66%), EC2 Instance SP (family + region locked, up to 72%), SageMaker SP, 1y/3y, All/Partial/No Upfront
5. **Reserved Instances (2 min)** — Standard (lockable, sellable on Marketplace), Convertible (swappable), still needed for RDS / ElastiCache / Redshift / OpenSearch / DynamoDB Reserved Capacity
6. **Dedicated Hosts vs Dedicated Instances (1 min)** — DH for BYOL Windows/Oracle visibility
7. **Image Builder + Compute Optimizer (2 min)** — Image Builder for golden AMIs; Compute Optimizer for right-sizing recommendations
8. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- EC2 families: M = general, C = compute, R = memory, I = storage IOPS, P/G = GPU, Inf/Trn = AI accelerator, A/Mg/Cg = Graviton ARM
- Graviton instances: ~20% cheaper, ~40% better price-perf — recommended where ARM compatible
- Spot: up to 90% off, can be interrupted with 2-min notice, ideal for stateless / fault-tolerant / batch
- Spot allocation strategy: Capacity-Optimized (fewer interruptions, recommended) vs Lowest-Price (cheapest but more churn)
- Compute Savings Plan covers EC2 + Fargate + Lambda — most flexible
- EC2 Instance SP covers one family + one Region — highest discount
- Standard RI vs Convertible RI: Standard sellable on Marketplace + higher discount; Convertible exchangeable
- DynamoDB Reserved Capacity is separate from SP
- RDS / ElastiCache / Redshift / OpenSearch need RIs — no SP option
- Capacity Reservations (ODCR) reserve capacity, no discount; combine with SP/RI for both
- Dedicated Host: socket/core visibility, required for most BYOL Windows/Oracle, you pay per-host
- Dedicated Instance: dedicated hardware, no socket visibility, less BYOL-friendly
- Image Builder: pipeline (source AMI + components + tests + distribution + schedule)
- Compute Optimizer: ML-based right-sizing for EC2, ASG, EBS, Lambda, ECS on Fargate

## Must-Mention Exam Traps

- EXAM TRAP: Savings Plans do NOT cover RDS / ElastiCache / Redshift / OpenSearch — those need RIs
- EXAM TRAP: Savings Plans do NOT reserve capacity — combine with Capacity Reservation if capacity is needed
- EXAM TRAP: Spot is INTERRUPTIBLE — never put stateful or time-critical workloads on pure Spot
- EXAM TRAP: Spot Block (1–6 hour guaranteed) was deprecated — don't pick it
- EXAM TRAP: Standard RI cannot change instance family/OS/tenancy — Convertible can
- EXAM TRAP: Convertible RI cannot be sold on Marketplace — only Standard
- EXAM TRAP: Dedicated Instance ≠ Dedicated Host — different products
- EXAM TRAP: Compute Optimizer needs 14 days of CloudWatch data to make recommendations
- EXAM TRAP: SP/RI discount sharing across Org is ON by default — disable per linked account if needed
- EXAM TRAP: Graviton requires ARM-compatible code — check before assuming portability
- EXAM TRAP: EC2 hibernation requires encrypted EBS root + specific instance types
- EXAM TRAP: Burstable T-series instances use CPU credits — sustained high CPU stops them or charges Unlimited mode

## Key Decision Matrix

| Workload | Pick |
|---|---|
| Stateful prod, steady load, known family | EC2 Instance SP or Standard RI |
| Modern mixed prod (EC2 + Fargate + Lambda) | Compute Savings Plan |
| Stateless batch, ML training, CI runners | Spot |
| Unpredictable bursty traffic | On-Demand |
| Lowest cost stateful baseline + bursty top | SP/RI + On-Demand layered |
| BYOL Windows Server | Dedicated Host + License Manager |
| RDS / Aurora discount | Reserved Instance |
| Right-size existing EC2 fleet | Compute Optimizer |
| Pipeline for golden AMIs | EC2 Image Builder |
| Need guaranteed AZ capacity | On-Demand Capacity Reservation |

## Tone & Style

- Layer cake: "SP/RI for baseline, On-Demand for burst, Spot for batch"
- Repeat "Compute SP covers EC2 + Fargate + Lambda"
- Emphasize that RDS / ElastiCache / Redshift need RIs not SPs

## Rapid-Fire Closer

"Compute SP covers Lambda?" — "YES." "SP covers RDS?" — "NO." "Spot interruption notice?" — "2 minutes." "BYOL Oracle?" — "Dedicated Host." "Right-size EC2?" — "Compute Optimizer."
