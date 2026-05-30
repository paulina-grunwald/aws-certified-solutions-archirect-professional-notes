# AWS Savings Plans (+ Reserved Instances comparison)

> **Flexible commitment-based discount model. Commit to $/hour of compute for 1 or 3 years; get up to 72% off On-Demand. Three types: Compute SP (most flexible — any EC2 family/region/OS + Fargate + Lambda), EC2 Instance SP (one family in one region, up to 72% off), SageMaker SP. Newer + simpler than Reserved Instances for most cases. Cost Explorer has built-in SP/RI recommendations. Critical SAP-C02 distractor topic — know when to pick SP vs RI vs Spot vs On-Demand.**

Maps to: **Domain 1.5 — Cost optimization**, **Domain 3.5 — Improve cost**

---

## Three Types of Savings Plans

### Compute Savings Plans (most flexible)
- Apply to **EC2 (any family / size / region / OS / tenancy) + Fargate + Lambda**
- Up to **66% off** On-Demand
- Best for **variable / unpredictable workloads** — auto-applies to any compute usage
- Recommended **default choice** unless you're locked to one instance family

### EC2 Instance Savings Plans
- Lock to **one instance family in one region** (e.g., m5 in us-east-1)
- Up to **72% off** On-Demand (highest discount)
- Size flexibility within family (m5.large / m5.xlarge / m5.metal — all eligible)
- Best for **stable, predictable EC2 workloads** on known family

### SageMaker Savings Plans
- Apply to **SageMaker** instance usage
- Up to 64% off
- Compute SP does NOT cover SageMaker — separate plan

## Commitment Mechanics

- Commit to **$X/hour for 1 year or 3 years**
- Payment options: **All Upfront** (deepest discount), **Partial Upfront**, **No Upfront**
- Usage **up to $X/hour** discounted; usage **above** is On-Demand price
- Auto-applies across linked accounts in an Org (if SP sharing enabled)

## Reserved Instances (RI) — Legacy Comparison

RIs predate Savings Plans. Largely **superseded by SP for EC2** but still relevant for some services.

| Aspect | Savings Plans | Reserved Instances |
|---|---|---|
| Applies to | EC2 + Fargate + Lambda (Compute SP) | EC2 / RDS / ElastiCache / Redshift / OpenSearch / DynamoDB-DAX (each per-service) |
| Flexibility | Auto-applies across family/region (Compute SP) | Standard RI = locked; Convertible RI = exchange allowed |
| Sale on marketplace | No | Yes (Standard RI only) |
| Discount | Up to 72% | Up to 72% |
| Effort | Low — just commit $/hr | Higher — pick instance type, size, region, OS |
| Best for | Modern EC2 + Lambda + Fargate workloads | RDS / ElastiCache / Redshift / OpenSearch (no SP for these) |

**Rule of thumb:** For **EC2/Fargate/Lambda → Savings Plans**. For **RDS/ElastiCache/Redshift/OpenSearch → RIs** (no SP option).

## Standard RI vs Convertible RI

| | Standard RI | Convertible RI |
|---|---|---|
| Modify family/OS/tenancy | No | Yes |
| Discount | Higher (up to 72%) | Slightly lower (up to 66%) |
| Sell on Marketplace | Yes | No |
| Best for | Locked-in workload | Workload that might change instance type |

## RI/SP Sharing in Organizations

- **Discount sharing ON by default** — all linked accounts benefit from any RI/SP purchase
- Can disable per-account in management account
- Common pattern: dedicated "billing account" buys all SPs/RIs; rest of Org uses them
- **Consolidated billing** required (i.e., AWS Organizations)

## Capacity Reservations vs Savings Plans

Often confused:

| | Capacity Reservation | Savings Plan |
|---|---|---|
| Reserves capacity | **Yes** (guaranteed launch in AZ) | No |
| Provides discount | No (unless paired with RI) | **Yes** (up to 72%) |
| Billing | Charged whether used or not | Charged regardless of usage |

**On-Demand Capacity Reservation (ODCR)** = guaranteed capacity, On-Demand price. Combine with RI/SP for both.

## Spot vs On-Demand vs SP/RI

| | Discount | Interruption | Use |
|---|---|---|---|
| **On-Demand** | 0% | None | Spiky, unpredictable, dev/test |
| **Spot** | Up to 90% | Yes (2-min notice) | Stateless, fault-tolerant, batch |
| **SP / RI** | Up to 72% | None | Baseline steady-state |

**Layered strategy** (most cost-efficient):
- **Baseline**: SP or RI
- **Burst**: On-Demand
- **Batch / async**: Spot

## Cost Explorer Recommendations

- Cost Explorer analyzes your last 7 / 30 / 60 days of usage
- Recommends specific SP / RI purchases with expected savings
- Filter by 1y vs 3y, upfront option, account scope
- Use these recommendations as starting point — don't over-commit

## Reserved Instance Specific Mechanics

### Zonal vs Regional RIs (EC2)
- **Zonal RI** — reserves capacity in a specific AZ, AZ flexibility off
- **Regional RI** — no capacity reservation, but **AZ + size flexibility** within family

### Convertible RI exchange
- Trade convertible RIs for different instance type / OS / tenancy
- Must be of equal or greater value (more $)
- Time remaining preserved

### RI Marketplace
- Sell unused **Standard RIs**
- Buyer gets the remaining term at potentially discounted price
- Sellers in US bank account only

## DynamoDB Reserved Capacity

- DynamoDB has its own **Reserved Capacity** (not SP-eligible)
- 1y or 3y commitment for provisioned RCU/WCU
- ~50% discount

## Common Patterns

### Modernization workload (EC2 + Fargate + Lambda)
- **Compute Savings Plan** — covers all three
- Don't bother with EC2-Instance-SP unless you're locked to one family

### Steady EC2 workload, known family
- **EC2 Instance Savings Plan** — highest discount (72%)
- Lower flexibility but max savings

### Mixed RDS + EC2 production
- **Compute SP** for EC2
- **RDS Reserved Instances** for RDS (no SP option for RDS)
- Aurora I/O Optimized + RIs combined for I/O-heavy DBs

### Variable analytics workload
- Mix: **Compute SP** baseline + **Spot** for EMR / batch
- Burst with On-Demand if Spot capacity unavailable

## Pricing & Commitment Tips

- Start with **No Upfront 1y Compute SP** — least risk to over-commit
- Move to **3y All Upfront EC2-Instance-SP** once usage stable
- Use **Cost Explorer recommendations** with conservative settings (1y, 95% confidence)
- **Don't over-commit** — uncovered SP commitment is wasted spend
- Combine **Compute SP (broad) + EC2-Instance-SP (specific family)** for layered discount

## Exam Tips

- "Most flexible discount for EC2 + Fargate + Lambda" → **Compute Savings Plan**
- "Highest discount for known steady EC2 family" → **EC2 Instance Savings Plan** (up to 72%)
- "RDS / ElastiCache / Redshift discount" → **Reserved Instances** (no SP option)
- "DynamoDB discount" → **DynamoDB Reserved Capacity** (separate from SP)
- "SageMaker training cost discount" → **SageMaker Savings Plan**
- "Guaranteed EC2 capacity in an AZ regardless of price" → **On-Demand Capacity Reservation** (not RI/SP)
- **Discount sharing ON by default** across Org
- 1y / 3y terms; All / Partial / No Upfront
- Standard RI = sellable; Convertible RI = swappable

## Exam Traps

- **SP doesn't cover RDS / ElastiCache / Redshift / OpenSearch** — those need RIs
- **SP doesn't cover SageMaker** — separate SageMaker SP
- **SP doesn't reserve capacity** — combine with ODCR if capacity is required
- **Convertible RI can NOT be sold on Marketplace** — only Standard RIs
- **RI Marketplace is US-only sellers** (US bank account)
- **Compute SP covers Lambda + Fargate** — easy to forget; not just EC2
- **EC2 Instance SP is one family in one region** — losing instance flexibility for max discount
- **Spot is NOT a Savings Plan** — separate purchasing model
- **DynamoDB On-Demand has no Reserved Capacity** — only provisioned mode is reservable
- **Cost Explorer recommendations are based on history** — wrong if workload pattern is about to change
- **Capacity Reservation ≠ Reservation (RI)** — totally different concepts
