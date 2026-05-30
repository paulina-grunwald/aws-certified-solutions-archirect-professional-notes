# AWS Proton

> **Managed delivery service for container + serverless platform engineering. Platform team defines "environment templates" (shared infra: VPC, cluster, monitoring) and "service templates" (app-specific: pipeline, ECS service, Lambda). Developers self-deploy services into approved environments without touching infra code. Templates are versioned; Proton can roll forward all running services when a new template version ships. Compare to Service Catalog (one-shot self-service) vs Proton (continuous platform-as-a-product). Lower SAP-C02 frequency but tested as a distractor.**

Maps to: **Domain 4.3 — Modernization**, **Domain 3.4 — Improve deployments**, **Domain 1.4 — Multi-account governance**

---

## Overview

- **Platform-as-a-product** delivery service
- Platform engineers define **templates**; app developers consume them
- Two template types: **Environment** (shared infra) + **Service** (app workload)
- Supports **CloudFormation** + **Terraform** as IaC engines
- **Versioned templates** with controlled rollout to running services
- Integrates with **CodePipeline / CodeBuild** for app CI/CD

## Two Template Types

### Environment Template
- Defines **shared infrastructure** for a class of services
- Examples: VPC + EKS cluster + observability stack, or shared API Gateway + Cognito
- Provisioned once per environment instance

### Service Template
- Defines **service-specific resources**: ECS task / Lambda function + IAM role + CI/CD pipeline
- Deployed into a compatible environment
- Each developer team uses the same service template

A service template is **compatible** with one or more environment templates (declared in the template spec).

## Roles & Workflow

### Platform Engineer
- Authors environment + service templates (CloudFormation / Terraform + schema)
- Publishes new versions
- Manages template-to-environment compatibility
- Approves environment provisioning requests

### Developer
- Browses available service templates in Proton console / CLI
- Provisions a service into an existing environment
- Updates service code → Proton pipeline deploys
- No direct IaC access — works through Proton

## Versioning + Rollout

- Major + minor template versions
- Platform team publishes new version
- Service instances can be **upgraded individually or in bulk**
- **Cancel / rollback** if rollout fails
- Enables "platform-as-a-product" lifecycle: keep ALL running services current with the latest hardened template

## Proton vs Service Catalog

| | Proton | Service Catalog |
|---|---|---|
| Audience | Platform + dev teams | Any (including non-engineers) |
| Mental model | Continuous platform-as-a-product | Curated self-service catalog |
| Service lifecycle | Roll forward (templates evolve) | Provision-and-forget |
| Two template tiers | **Env + Service** (linked) | Single product |
| CI/CD pipeline | **Built-in service deployment pipeline** | Add separately |
| Best for | Standardize how teams ship microservices | Standardize one-shot product provisioning |
| Complexity | Higher | Lower |

## Proton vs CDK / Terraform

Proton **uses** CloudFormation or Terraform under the hood. The added value:
- **Self-service UI / API for developers**
- **Template versioning + rollout management**
- **Built-in CI/CD pipeline per service**
- **Environment + service separation enforced**

If you're a small team — direct CDK / Terraform is simpler. Proton shines at **10+ teams** all shipping services to shared infra.

## Integrations

| Integration | Use |
|---|---|
| **CodePipeline / CodeBuild** | Service deployment pipelines |
| **CloudFormation** | Template engine |
| **Terraform** | Template engine alternative |
| **GitHub / GitLab / Bitbucket** | Source repos for service code |
| **IAM Identity Center** | Federated access |
| **EventBridge** | Provisioning + deployment events |
| **CloudWatch** | Service + environment metrics |

## Common Patterns

### Microservices platform on EKS
- Environment template: shared EKS cluster, ArgoCD, observability
- Service template: K8s deployment + ALB rule + CodePipeline
- Dev team provisions new service via Proton → CI/CD ready in 10 min

### Serverless API platform
- Environment template: API Gateway + Cognito + Lambda layers
- Service template: Lambda function + DynamoDB table + pipeline
- Common patterns enforced; teams focus on business logic

### Multi-account organization
- Environments per stage (dev / staging / prod) in separate accounts
- Cross-account roles for Proton to provision
- Centralized template management

### Continuous platform update
- Platform team patches a CVE in base AMI → publishes env template v2
- Roll forward all 50 service instances → automated
- Compliance achieved org-wide

## Pricing

- **Free** — Proton itself
- Pay for underlying resources (EC2, Lambda, EKS, CodePipeline, etc.)

## Exam Tips

- "Self-service platform for developers, with versioned templates and rollout management" → **AWS Proton**
- Two template tiers: **environment + service**
- Supports **CloudFormation + Terraform**
- Built-in **service deployment pipelines**
- Free service
- Common SAP-C02 distractor against **Service Catalog**

## Exam Traps

- **Proton ≠ Service Catalog** — Proton is platform engineering with two-tier templates + rollout; Catalog is one-shot curated products
- **Proton ≠ Elastic Beanstalk** — Beanstalk is opinionated PaaS for a single app; Proton is meta-tooling for platform teams
- **Service templates need a compatible environment** — can't deploy without matching env
- **Template upgrades aren't always backward compatible** — test before mass rollout
- **Free service** — but the resources Proton provisions are not
- **Lower exam frequency than Service Catalog** — but still appears as a distractor on platform engineering questions
- **Proton supports Terraform Open Source** — not just CloudFormation
