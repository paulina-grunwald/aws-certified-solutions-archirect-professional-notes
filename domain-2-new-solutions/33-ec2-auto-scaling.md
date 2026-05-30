# EC2 Auto Scaling

> **Automatically maintain desired EC2 capacity across AZs. Launch Templates (preferred) + Mixed Instances Policy for On-Demand + Spot mix. Scaling policies: Target Tracking (preferred), Step Scaling, Scheduled, Predictive. Lifecycle Hooks for graceful pre-launch / pre-terminate. Warm Pools for fast scale-out. Instance Refresh for rolling AMI/template updates. Capacity Rebalancing for Spot interruption protection. Cross-AZ self-healing via health checks (EC2, ELB, custom).**

Maps to: **Domain 2.5 — High-performance**, **Domain 1.3 — Reliability**, **Domain 2.6 / 3.5 — Cost**

---

## Overview

- **Maintain desired EC2 capacity** across AZs
- **Scale out / in** based on metrics or schedule
- **Replace unhealthy instances** automatically (self-healing)
- Works with **EC2 + ALB / NLB / Application Auto Scaling**
- **No additional charge** — only pay for the EC2 / EBS / data transfer

---

## Components

- **Launch Template** (preferred over deprecated Launch Configuration) — versioned, supports all features (Spot, mixed instance, T-unlimited)
- **Auto Scaling Group (ASG)** — min / desired / max + AZs + subnets
- **Scaling Policies** — when to add / remove capacity
- **Lifecycle Hooks** — pause `Pending:Wait` or `Terminating:Wait` for custom logic
- **Health Checks** — EC2 (default), ELB, custom (via API)

---

## Launch Template vs Launch Configuration

- **Launch Configuration** is **legacy / deprecated**
- **Use Launch Templates** for all new ASGs — versioned, supports Mixed Instances Policy, advanced features

---

## Mixed Instances Policy

- Combine **On-Demand + Spot** in one ASG
- Choose multiple instance types (e.g., m5.large, m5a.large, c5.large)
- **Allocation strategies**:
  - **`price-capacity-optimized`** (recommended, current default for new ASGs) — balances cost + interruption risk
  - **`capacity-optimized`** — minimize Spot interruption (good for training jobs)
  - **`lowest-price`** — legacy; high interruption risk
- **On-Demand base + Spot %** for predictable + bursty mix

---

## Scaling Policies

| Policy | When |
|---|---|
| **Target Tracking** (preferred default) | "Keep average CPU at 50%" — ASG calculates how many instances |
| **Step Scaling** | Escalating responses based on alarm magnitude (e.g., +1 if 60–70%, +3 if > 70%) |
| **Simple Scaling** (legacy) | Single +/- N on alarm; no cooldown management |
| **Scheduled Scaling** | Known peaks (e.g., Black Friday, business hours) |
| **Predictive Scaling** | ML forecasts load and scales preemptively (24-hour ahead) |

> **Target Tracking is the default best practice** for unknown / variable load.

---

## Lifecycle Hooks

- Pause instance in `Pending:Wait` or `Terminating:Wait` state
- Time budget: configurable timeout
- Use cases:
  - **Pre-launch** bootstrap (install agents, register, warm cache)
  - **Pre-terminate** drain (graceful shutdown, deregister from custom LB, flush logs)
- Hooks publish to SNS / SQS for handler to receive notifications

---

## Warm Pools

- Pool of **pre-initialized stopped instances**
- Fast scale-out for **slow-booting AMIs** (Windows, custom enterprise images)
- Pay for EBS storage of warm instances only (no compute when stopped)
- Configurable pool size + reuse policy

---

## Instance Refresh

- **Rolling replacement** of instances when AMI / Launch Template updates
- Configurable batch size + warm-up time
- **Checkpoints** to monitor progress
- **Skip Matching** — avoid replacing instances already on the new version
- Use case: rolling AMI upgrades, security patches without downtime

---

## Capacity Rebalancing

- Proactively replace Spot instances flagged for interruption
- Triggered by **EC2 Rebalance Recommendation** signal (before 2-min notice)
- Reduces downtime + work loss from Spot reclaims

---

## Health Checks

| Type | Description |
|---|---|
| **EC2 health check** (default) | System / instance status from EC2 |
| **ELB health check** | Use ALB / NLB / target group health |
| **Custom health check** | Set via `SetInstanceHealth` API from your app |

> Combine EC2 + ELB for thorough health detection.

---

## Termination Policies

When scaling in, ASG picks which instances to terminate:

- **default** — even AZ distribution → oldest Launch Configuration / Launch Template version → closest to next billing hour
- **OldestInstance** — replace older instances first
- **NewestInstance** — replace newest first
- **OldestLaunchTemplate** / **OldestLaunchConfiguration** — replace those on old versions
- **AllocationStrategy** — keep mixed-instance allocation aligned

---

## Cooldown vs Warm-up

- **Cooldown** — wait after scaling activity before next (legacy Simple Scaling)
- **Warm-up time** — instances marked InService but excluded from metrics until warmed up (Target / Step Scaling)
- **Default 300s** for both

---

## Standby State

- Take an instance **out of rotation** without termination
- Use for troubleshooting, manual updates
- Does not count toward desired capacity

---

## Lifecycle Diagram

```
Launch Hook → Pending:Wait → bootstrap → Pending → InService
                                                      ↓
                                  Standby ↔ InService → Terminating:Wait → Terminate Hook → Terminated
```

---

## Common Patterns

### Auto Scaling + ALB

- ALB target group health → ASG health check
- ASG registers / deregisters instances with ALB on scale events
- Pattern: web tier behind ALB

### Self-Healing (Domain 1.3)

- ASG with min=1, max=1, desired=1
- Single critical instance auto-replaced on failure
- Cheaper alternative to EC2 Auto-Recovery for self-managed workloads

### Spot-Heavy Mixed Fleet

- Mixed Instances Policy
- On-Demand base = 1 (reliability floor)
- 90% Spot above base, `price-capacity-optimized` allocation
- Capacity Rebalancing ON
- Lifecycle hook to drain Spot before termination

---

## Exam Tips

- **Launch Template > Launch Configuration** (LC is deprecated)
- **Mixed Instances Policy** for On-Demand + Spot diversification
- **Target Tracking** is the default best practice scaling policy
- **Predictive Scaling** for known cyclical patterns
- **Warm Pools** for slow-booting AMIs (Windows, custom enterprise)
- **Instance Refresh + Skip Matching** for rolling updates
- **Capacity Rebalancing** for proactive Spot interruption protection
- **Lifecycle Hooks** for bootstrap + graceful drain
- **Custom health check** via `SetInstanceHealth` API
- **Termination protection (instance level) does NOT block ASG-initiated termination**
- **ASG can span AZs but NOT Regions** — for multi-Region use Route 53 / Global Accelerator
- **`price-capacity-optimized`** is the current default Spot allocation strategy

---

## Exam Traps

- **Launch Configuration is deprecated** — don't propose it for new designs
- **EC2 instance termination protection does NOT prevent ASG-initiated termination** — only direct API calls
- **`SetInstanceHealth` → Unhealthy** triggers ASG replacement
- **ASG does NOT span Regions** — single Region only
- **Spot Fleet is NOT the same as ASG** — Spot Fleet is legacy; Mixed Instances Policy in ASG is the modern path
- **Cooldown applies to Simple Scaling**; Target / Step use **warm-up time** instead
- **Warm Pool instances incur EBS charges** even when stopped
- **Lifecycle Hooks have a max timeout** — long bootstrap should be in pre-baked AMI
- **Predictive Scaling needs 24 hours of historical data** before recommendations
