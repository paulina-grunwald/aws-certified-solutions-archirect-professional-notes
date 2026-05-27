# AWS Fault Injection Service (FIS)

> **Managed chaos engineering service. Run controlled failure experiments against AWS workloads to validate resilience. Define experiment templates (which targets, which fault actions, in what order, with what stop conditions). Supports EC2, ECS, EKS, RDS, networking, CloudWatch alarms as stop guards. Pre-built scenarios for AZ failure, Region disruption, and common patterns. Integrates with Resilience Hub (which generates FIS templates). Previously branded "Fault Injection Simulator".**

Maps to: **Domain 3.1 — Improve reliability**, **Domain 2.4 — Reliable / resilient solutions**

---

## Overview

- **Chaos engineering as a managed service** — inject real faults, observe behavior
- **Experiments are safe by design** — stop conditions roll back automatically
- Test the recovery path **before a real outage exposes the gap**
- Works with **EC2, ECS, EKS, RDS, Aurora, networking (VPC), Lambda, SSM-managed instances**
- Used by **AWS Resilience Hub** to validate resilience improvements

## Core Concepts

| Concept | Meaning |
|---|---|
| **Experiment template** | Reusable definition of an experiment |
| **Action** | A specific fault (e.g., `aws:ec2:terminate-instances`) |
| **Target** | Which resources the action runs against (by tag, ARN, filter) |
| **Stop condition** | CloudWatch alarm that aborts the experiment |
| **Experiment** | A run of a template |

## Fault Actions (Examples)

### EC2
- `aws:ec2:stop-instances`
- `aws:ec2:terminate-instances`
- `aws:ec2:reboot-instances`
- `aws:ec2:send-spot-instance-interruptions`

### ECS / EKS
- `aws:ecs:stop-task`
- `aws:ecs:drain-container-instances`
- `aws:eks:terminate-nodegroup-instances`
- `aws:eks:pod-cpu-stress` (via Kubernetes API)
- `aws:eks:pod-network-latency`

### RDS / Aurora
- `aws:rds:reboot-db-instances`
- `aws:rds:failover-db-cluster` (Aurora)

### Networking
- `aws:network:disrupt-connectivity` — block subnets/AZs by route table manipulation
- `aws:network:route-table-disrupt-cross-region-connectivity`

### Systems Manager / SSM
- `aws:ssm:send-command` — run a script that injects CPU stress, memory pressure, disk fill, kernel panic
- Lets you simulate any application-layer fault

### Lambda
- `aws:lambda:invocation-add-delay`
- `aws:lambda:invocation-error`
- `aws:lambda:invocation-http-integration-response`

## Stop Conditions (Critical Safety Net)

- A CloudWatch alarm tied to the experiment
- If alarm fires (e.g., error rate > 5%, latency > 2s), FIS **automatically halts** the experiment and rolls back where possible
- Multiple stop conditions per experiment

## Pre-Built Scenarios

AWS provides scenario templates:

- **AZ Availability: Power Interruption** — disrupts a full AZ's compute + networking
- **Cross-Region: Connectivity** — simulates Region partition
- **Many more** for ECS, EKS, RDS, Outposts

These cover common SAP-C02 patterns without writing the YAML.

## Multi-Account / Multi-Region

- Use **IAM cross-account role** for the FIS experiment role
- Run experiments across multiple Regions simultaneously
- Combine with Resilience Hub for org-wide chaos testing program

## Integration with Resilience Hub

- Resilience Hub recommendations include **pre-generated FIS templates**
- Run them → verify the recommendation works
- Re-assess → confirm score improves

## Logging

- All experiment events to **CloudWatch Logs** + **S3**
- Useful for post-experiment retro + audit

## IAM

- FIS uses a **service role** to perform actions on your behalf
- Scope the role tightly — give it permission only for the experiment's targets

## Pricing

- **$0.10 per action-minute** (per fault action, per minute of duration)
- Example: terminate instances for 30 min = $3
- Downstream AWS costs (replacement instances, CloudWatch alarms) separate

## Common Patterns

### Test Auto Scaling response

- Action: terminate 50% of ASG instances by tag
- Stop condition: ALB 5xx > 1%
- Expected: ASG launches replacements; ELB drains failing targets

### Validate Multi-AZ RDS failover

- Action: `aws:rds:failover-db-cluster`
- Observe: app reconnects, time to recovery, query latency spike
- Confirm RTO target met

### Simulate AZ outage

- Use pre-built **AZ Availability: Power Interruption** scenario
- Validates ASG spreads, RDS Multi-AZ, ELB cross-AZ behavior

### GameDay exercises

- Run experiments during a scheduled window
- Engineers practice incident response
- Document gaps → fix them

## GameDay & Best Practices

- Start in **non-prod** — build confidence
- Establish **steady-state hypothesis** before running ("error rate stays below 1%")
- **Run during business hours** with the team watching (not at 3am)
- **Document blast radius** — which resources, what could go wrong
- Always set **CloudWatch alarm stop conditions**
- **Use tags** to limit targets — never run against the whole account

## Exam Tips

- "Inject failures to test resilience" → **AWS FIS**
- "Simulate AZ outage / Region partition" → **FIS pre-built scenarios**
- "Validate Multi-AZ failover automatically" → **FIS with RDS failover action**
- Stop conditions are **CloudWatch alarms**
- Used by **Resilience Hub** to validate recommendations
- Supports Lambda, EC2, ECS, EKS, RDS, networking, SSM-driven faults
- Pricing is **per action-minute**

## Exam Traps

- **FIS doesn't auto-rollback every action** — termination is irreversible; rollback is best-effort. Plan recovery into the architecture
- **Stop conditions are mandatory in practice** — without one, a runaway experiment can take down prod
- **Not a load testing tool** — that's CloudWatch Synthetics + custom load
- **Not a security testing tool** — that's Inspector, GuardDuty, pentest authorization
- **Resilience Hub doesn't run experiments** — it generates FIS templates; FIS runs them
- **Don't run in prod first** — start in staging or a copy; build confidence
- **"Fault Injection Simulator" old name** — same service, AWS renamed to "Fault Injection Service"
