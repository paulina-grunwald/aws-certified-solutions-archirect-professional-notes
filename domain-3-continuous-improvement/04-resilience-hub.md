# AWS Resilience Hub

> **Central service for assessing, validating, and continuously improving application resilience against RTO / RPO targets. Define a resilience policy (disruption tolerance), Resilience Hub analyzes the app's infrastructure (CloudFormation, Terraform, EKS, AppRegistry, Resource Groups), simulates failure modes, scores recovery against your targets, and generates recommendations + SOPs + FIS experiment templates. Integrates with AWS FIS for actual chaos testing. Pairs naturally with AWS DRS, Backup, Route 53 ARC, and CloudWatch alarms.**

Maps to: **Domain 3.1 — Improve reliability**, **Domain 1.3 — Reliable / resilient architectures**, **Domain 2.2 — Business continuity**

---

## Overview

- **Managed resilience assessment + tracking** for production workloads
- Define **resilience policies** (RTO / RPO targets per disruption type)
- Resilience Hub **scores** your app against those targets
- Provides **recommendations** (architecture changes, config changes) to close gaps
- Generates **SOPs** (Systems Manager runbooks) for recovery
- Generates **FIS experiment templates** to test recoverability
- Integrates with **AWS FIS** to actually run the chaos experiments
- Tracks **resilience drift** over time (CI/CD-friendly via API)

## Resilience Policy

A policy defines **target RTO / RPO** for four disruption categories:

| Disruption | Example |
|---|---|
| **Software** | Bug, bad deploy, app crash |
| **Hardware** | Single instance / AZ failure |
| **AZ** | Full AZ outage |
| **Region** | Regional outage |

You set RTO + RPO targets per category. Hub then assesses whether the architecture **can meet** those targets.

## Discovering an Application

Resilience Hub discovers resources via:

- **CloudFormation stacks**
- **Terraform state files** (S3)
- **AWS Resource Groups**
- **AppRegistry**
- **EKS clusters** (Kubernetes resources)

Selecting the source defines the **app boundary**.

## Assessment Output

- **Resilience score** (0–100) — overall + per disruption category
- **Compliance status** — policy met / not met
- **Component-level breakdown** — which components fail the policy
- **Recommendations**:
  - **Optimize for cost** — cheapest path to policy compliance
  - **Optimize for AZ resilience**
  - **Optimize for Region resilience**
  - **Optimize for infrastructure** (architecture changes)
- **Estimated RTO / RPO** per component
- **SOPs** — Systems Manager runbooks for failover steps
- **Tests** — FIS experiment templates aligned to recommendations
- **Alarms** — CloudWatch alarms aligned to recommendations

## Recommended Improvements (Common)

- Add **Multi-AZ** to RDS / ElastiCache / EFS
- Add **cross-Region replication** (S3 CRR, Aurora Global, DynamoDB Global Tables)
- Add **Route 53 health checks + failover**
- Configure **AWS Backup** with cross-Region copy
- Add **Auto Scaling** + ELB health checks
- Replicate **EBS snapshots** cross-Region with AWS Backup or DLM
- Use **Route 53 Application Recovery Controller (ARC)** for active-passive failover

## Integration with AWS FIS

- Recommendations include **pre-built FIS experiment templates**
- Run experiments → verify the app actually recovers within policy
- Re-assess → confirm score improves

## Integration with Operational Tools

- **CloudWatch alarms** — observability for the resilience policy
- **Systems Manager** — SOPs as Automation runbooks
- **EventBridge** — assessment results published as events
- **AWS Config** — track resource compliance with resilience requirements

## Continuous Resilience (CI/CD)

- API-driven — call from a CodePipeline / GitHub Actions stage
- **Fail the pipeline** if resilience score drops below threshold
- Detect "resilience drift" when devs ship changes that weaken recovery

## Pricing

- **$15 per application per month** (per assessment) — flat
- FIS, CloudWatch, Systems Manager costs separate

## Common Patterns

### Validating DR strategy

- Define policy: RTO 1h, RPO 5min, Region disruption
- Assess → Hub flags that Aurora is single-Region
- Recommendation: enable Aurora Global Database
- Apply → re-assess → score now meets policy

### Continuous compliance in CI/CD

- Resilience Hub API runs after each deploy
- Score < 80 fails the deploy
- Forces dev teams to maintain resilience as they ship

### Pre-production chaos testing

- Hub generates FIS templates (terminate instance, simulate AZ failure)
- Run in staging → observe app behavior
- Tune the architecture before promoting

## Exam Tips

- "Continuously assess and track resilience against RTO/RPO targets" → **AWS Resilience Hub**
- "Generate FIS experiment templates from recommendations" → **Resilience Hub**
- "Score an app's compliance with a resilience policy" → **Resilience Hub**
- Sources: CloudFormation, Terraform, Resource Groups, AppRegistry, EKS
- Output includes SOPs (SSM), alarms, FIS experiments, recommendations
- Four disruption categories: **Software / Hardware / AZ / Region**
- $15 per app per month

## Exam Traps

- **Resilience Hub does not run failure tests itself** — it generates FIS templates; AWS FIS runs them
- **Not a backup service** — uses AWS Backup but doesn't replace it
- **Not real-time monitoring** — assessments are point-in-time; integrate with CI/CD for continuous
- **Doesn't auto-fix** — recommends; you apply
- **Trusted Advisor ≠ Resilience Hub** — Trusted Advisor is broad best-practice checks; Resilience Hub is resilience-specific with RTO/RPO policy
- **Well-Architected Tool ≠ Resilience Hub** — WA Tool covers all six pillars via questionnaire; Resilience Hub is resilience-only with automated assessment
