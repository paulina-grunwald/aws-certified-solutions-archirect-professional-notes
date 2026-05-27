# AWS Application Auto Scaling (and AWS Auto Scaling)

> **Generic scaling service for AWS resources that are NOT EC2 instances. Scales DynamoDB, Aurora replicas, ECS service tasks, Lambda provisioned concurrency, EMR clusters, AppStream fleets, SageMaker endpoints, Spot Fleet, Neptune, Comprehend, ElastiCache replicas, Keyspaces tables, and more. Target tracking, step, and scheduled scaling — same three policy types as EC2 Auto Scaling. "AWS Auto Scaling" is a higher-level console that orchestrates Application Auto Scaling + EC2 Auto Scaling together for a Resource Group.**

Maps to: **Domain 3.2 — Improve performance**, **Domain 2.4 — Reliable / resilient solutions**, **Domain 3.5 — Cost optimization**

---

## Three Things That Sound the Same — Don't Confuse

| Service | Scales |
|---|---|
| **EC2 Auto Scaling** | EC2 Auto Scaling Groups (instances behind ALB/NLB) |
| **Application Auto Scaling** | Non-EC2 scalable resources (DynamoDB, ECS, Aurora replicas, Lambda, etc.) |
| **AWS Auto Scaling** | Unified console / "scaling plan" that orchestrates both for a Resource Group |

For SAP-C02: when a question mentions scaling **DynamoDB / Aurora replicas / ECS tasks / Lambda concurrency**, the answer is **Application Auto Scaling**.

## Supported Scalable Resources (memorize)

- **DynamoDB** — table + GSI read/write capacity (provisioned mode only)
- **Aurora** — Aurora Replicas count
- **ECS service** — desired task count
- **Lambda** — provisioned concurrency
- **EMR** — task instance group count
- **AppStream 2.0** — fleet capacity
- **EC2 Spot Fleet** — target capacity
- **SageMaker endpoints** — variant instance count
- **Comprehend** — endpoint inference units
- **ElastiCache replication group** — replica count + node group count (Redis)
- **Keyspaces (Cassandra)** — table read/write throughput
- **Neptune** — read replica count
- **Custom resources** via CloudWatch custom metric

## Three Scaling Policy Types

### Target Tracking
- Define a target metric value (e.g., DynamoDB consumed capacity at 70%)
- Application Auto Scaling adds/removes capacity to keep metric at target
- **Most common + simplest** — recommended first choice
- Uses CloudWatch alarms under the hood

### Step Scaling
- Define thresholds + step adjustments
- E.g., "if CPU > 70%, add 2 tasks; if > 85%, add 5 tasks"
- More granular control for non-linear workloads

### Scheduled Scaling
- Scale at a specific time / cron expression
- Use case: pre-warm capacity before known traffic spike (sales event, batch job)
- Combine with target tracking — scheduled bumps the min, tracking handles within range

## Configuration Pattern

```
register-scalable-target
  ServiceNamespace: dynamodb
  ResourceId: table/MyTable
  ScalableDimension: dynamodb:table:ReadCapacityUnits
  MinCapacity: 5
  MaxCapacity: 100

put-scaling-policy
  PolicyType: TargetTrackingScaling
  TargetValue: 70.0
  PredefinedMetricSpecification: DynamoDBReadCapacityUtilization
```

## DynamoDB Auto Scaling (most-tested pattern)

- Provisioned mode only (on-demand mode auto-scales differently)
- Set min + max RCU/WCU + target utilization (default 70%)
- Application Auto Scaling adjusts within bounds
- **Scaling latency** — can take minutes; **don't rely on it for sudden spikes** → use on-demand or reserve burst with provisioned + high min

## Aurora Auto Scaling

- Scales **Aurora Replicas** count (1–15)
- Trigger: average CPU or average connections across existing replicas
- Replicas added based on policy; cluster endpoint routes reads
- **Does not scale writer** — that's vertical scaling or Aurora Serverless

## ECS Service Auto Scaling

- Target tracking on ALB request count, ECS CPU/memory utilization
- Combined with **Capacity Provider Reservation** for ASG-backed clusters
- For Fargate: no capacity-provider concern, just task count scales

## Lambda Provisioned Concurrency Auto Scaling

- Scales the number of provisioned (pre-warmed) execution environments
- Use case: latency-sensitive APIs where cold starts are unacceptable
- Combine with Application Auto Scaling target tracking on utilization

## AWS Auto Scaling (Unified Console)

- A scaling "plan" applied across mixed resources in a Resource Group
- Pulls in EC2 ASG + DynamoDB + Aurora + ECS in one view
- Pre-configured strategies: optimize for availability / cost / balance
- Less granular than configuring each service's Application Auto Scaling individually — most teams skip this and configure per-service

## Predictive Scaling

- Available for **EC2 Auto Scaling** + supported in **AWS Auto Scaling**
- Uses ML to forecast load and pre-scale 48h ahead
- Combine with dynamic scaling for unexpected spikes
- Not available on every Application Auto Scaling target

## Pricing

- **Application Auto Scaling itself is free**
- You pay for CloudWatch alarms used under the hood (~$0.10 per alarm-month) + the underlying resource (DynamoDB capacity, EC2 instances, etc.)

## Common Patterns

### DynamoDB hot-table burst
- Target tracking at 70% utilization with min 100 / max 10000 RCU
- Combine with **DynamoDB on-demand** if traffic is truly unpredictable
- Cache reads in **DAX** for very hot items

### ECS scale-to-zero non-prod
- Scheduled scaling: scale to 0 at 7 PM, scale to N at 7 AM
- Cuts dev/test Fargate cost ~70%

### Lambda PC during business hours
- Scheduled: PC = 50 weekdays 9–5
- Off-hours: PC = 5
- Caps cold-start latency while paying for warm capacity only when needed

### Aurora replica autoscaling for read bursts
- Target tracking on `CPUUtilization` at 60%
- Min 2 / max 10 replicas
- Reader endpoint routes new connections evenly

## Exam Tips

- "Scale DynamoDB capacity dynamically" → **Application Auto Scaling** (target tracking on utilization)
- "Scale Aurora Replicas / ECS service tasks / Lambda PC / SageMaker endpoints" → **Application Auto Scaling**
- "Scale EC2 instances" → **EC2 Auto Scaling** (different service)
- Three policy types: **target tracking, step, scheduled**
- Target tracking is the default first choice
- Scheduled scaling for known traffic patterns (sales events, batch windows)
- Predictive scaling is EC2-focused but available in AWS Auto Scaling plans
- AWS Auto Scaling = unified scaling plan across services for a Resource Group

## Exam Traps

- **DynamoDB Auto Scaling has scale-up latency** — doesn't react to sub-minute spikes well; use on-demand or pre-warm
- **Aurora Auto Scaling scales replicas only** — writer is vertical or Serverless
- **ECS service Auto Scaling ≠ ECS cluster Auto Scaling** — service scales tasks; cluster (EC2-backed) scales instances via ASG capacity provider
- **Application Auto Scaling is free; underlying resources are not** — DynamoDB scaling can blow your bill if max is too high
- **On-demand DynamoDB doesn't use Application Auto Scaling** — it auto-scales internally
- **Predictive scaling needs 24h of metric history** — not for brand-new workloads
- **AWS Auto Scaling (the unified console) confuses people** — most teams configure per-service Application Auto Scaling directly
- **Lambda concurrency limit ≠ Lambda PC scaling** — concurrency limit is account-wide; PC is pre-warmed environments per function
