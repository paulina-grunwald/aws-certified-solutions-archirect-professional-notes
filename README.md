# AWS Certified Solutions Architect – Professional (SAP-C02)

Study notes and reference material for the AWS Certified Solutions Architect – Professional exam.

## Exam Overview

| Item | Detail |
|---|---|
| **Exam Code** | SAP-C02 |
| **Level** | Professional |
| **Duration** | 180 minutes |
| **Number of Questions** | 75 |
| **Question Format** | Multiple choice, multiple response |
| **Passing Score** | 750 / 1000 (scaled) |
| **Cost** | $300 USD |
| **Delivery** | Pearson VUE test center or online proctored |
| **Languages** | English, Japanese, Korean, Simplified Chinese |
| **Validity** | 3 years |
| **Prerequisites** | None (AWS recommends 2+ years of AWS experience) |

## Scoring

- Scaled score from **100 to 1000**, passing at **750**.
- Compensatory scoring — you do **not** need to pass each domain individually, only the overall exam.
- Some questions are **unscored** (used for statistical evaluation) but are not identified.
- Results are pass/fail with a section-level performance breakdown.

## Exam Domains

| # | Domain | Weight |
|---|---|---|
| 1 | Design Solutions for Organizational Complexity | **26%** |
| 2 | Design for New Solutions | **29%** |
| 3 | Continuous Improvement for Existing Solutions | **25%** |
| 4 | Accelerate Workload Migration and Modernization | **20%** |

---

### Domain 1 — Design Solutions for Organizational Complexity (26%)

**1.1 Architect network connectivity strategies**
- Hybrid connectivity (Direct Connect, Site-to-Site VPN, Transit Gateway)
- Inter-VPC and inter-Region connectivity
- DNS strategies (Route 53, hybrid DNS, Resolver endpoints)
- Network segmentation and traffic inspection

**1.2 Prescribe security controls**
- Multi-account identity (IAM Identity Center, federation, SAML, OIDC)
- Cross-account access patterns and permission boundaries
- Centralized logging and audit (CloudTrail, Config, Security Hub, GuardDuty)
- Data protection (KMS, CloudHSM, encryption in transit/at rest)

**1.3 Design reliable and resilient architectures**
- Multi-account, multi-Region disaster recovery
- RTO/RPO trade-offs (backup & restore, pilot light, warm standby, multi-site active/active)
- Failover patterns and Route 53 routing policies

**1.4 Design a multi-account AWS environment**
- AWS Organizations, SCPs, OUs
- Control Tower, Landing Zone, Account Factory
- Centralized billing, consolidated cost management

**1.5 Determine cost optimization and visibility strategies**
- Cost Explorer, Budgets, Cost and Usage Report, Cost Categories
- Savings Plans vs. Reserved Instances vs. Spot

---

### Domain 2 — Design for New Solutions (29%)

**2.1 Design deployment strategies**
- IaC (CloudFormation, CDK, StackSets)
- CI/CD (CodePipeline, CodeBuild, CodeDeploy, blue/green, canary)

**2.2 Design solutions for business continuity**
- Backup & restore strategies (AWS Backup, cross-Region/cross-account)
- DR patterns and Recovery Point/Time Objectives

**2.3 Determine security controls for new workloads**
- Encryption, KMS key policies and grants
- Secrets Manager, Parameter Store
- WAF, Shield, network ACLs, security groups

**2.4 Design reliable and resilient solutions**
- Decoupling (SQS, SNS, EventBridge, Step Functions)
- Auto Scaling, health checks, circuit breakers, retries with backoff

**2.5 Design high-performance architectures**
- Compute selection (EC2 families, Graviton, Lambda, containers)
- Storage selection (S3 classes, EBS types, EFS, FSx)
- Database selection (RDS, Aurora, DynamoDB, Redshift, ElastiCache)
- Edge and caching (CloudFront, Global Accelerator)

**2.6 Determine cost-optimized solutions**
- Right-sizing, Auto Scaling, lifecycle policies
- Serverless vs. provisioned trade-offs

---

### Domain 3 — Continuous Improvement for Existing Solutions (25%)

**3.1 Improve reliability**
- Failure mode analysis, self-healing systems
- Multi-AZ / multi-Region patterns

**3.2 Improve performance**
- Bottleneck identification (CloudWatch, X-Ray, Compute Optimizer)
- Caching strategies, read replicas, sharding

**3.3 Improve security**
- Security Hub, Inspector, Macie, Detective
- Patch management (Systems Manager Patch Manager)

**3.4 Improve deployment processes**
- Pipeline observability and automated rollback
- Feature flags, deployment strategies

**3.5 Improve cost**
- Workload right-sizing, Compute Optimizer
- Storage tiering (S3 Intelligent-Tiering, lifecycle rules)

**3.6 Improve operational excellence**
- CloudWatch metrics, alarms, dashboards
- Systems Manager (OpsCenter, Run Command, Automation)

---

### Domain 4 — Accelerate Workload Migration and Modernization (20%)

**4.1 Select existing workloads for migration**
- Application Discovery Service, Migration Hub
- 7 Rs (Retire, Retain, Rehost, Relocate, Repurchase, Replatform, Refactor)

**4.2 Determine optimal migration approach**
- Database Migration Service (DMS), Schema Conversion Tool (SCT)
- Server Migration (MGN, Application Migration Service)
- Storage migration (DataSync, Snow family, Transfer Family, Storage Gateway)

**4.3 Determine new architecture and process for existing workloads**
- Container modernization (ECS, EKS, Fargate, App Runner)
- Serverless modernization (Lambda, Step Functions, EventBridge)
- Microservices decomposition

**4.4 Determine opportunities for modernization and enhancement**
- Managed services adoption
- Event-driven and decoupled patterns

---

## Reference Material

- [Official Exam Guide (SAP-C02)](https://d1.awsstatic.com/training-and-certification/docs-sa-pro/AWS-Certified-Solutions-Architect-Professional_Exam-Guide.pdf)
- [Sample Questions](https://d1.awsstatic.com/training-and-certification/docs-sa-pro/AWS-Certified-Solutions-Architect-Professional_Sample-Questions.pdf)
- [AWS Skill Builder — Exam Prep](https://explore.skillbuilder.aws/learn)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

## Repository Layout

Notes are organized per topic — folders and `.md` files will be added as topics are studied.

```
.
├── README.md                   
├── domain-1-organizational/
├── domain-2-new-solutions/
├── domain-3-continuous-improvement/
└── domain-4-migration/
```
