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

## Notes Index

### Domain 1 — Design Solutions for Organizational Complexity

**Foundations**
1. [AWS Global Infrastructure](domain-1-organizational/01-aws-global-infrastructure.md)

**Identity & Access**
2. [IAM](domain-1-organizational/02-iam.md)
3. [IAM Identity Center (AWS SSO)](domain-1-organizational/03-iam-identity-center.md)
4. [AWS Directory Service](domain-1-organizational/04-aws-directory-service.md)

**Multi-Account Management**
5. [AWS Organizations](domain-1-organizational/05-aws-organizations.md)
6. [AWS Control Tower](domain-1-organizational/06-aws-control-tower.md)
7. [AWS RAM (Resource Access Manager)](domain-1-organizational/07-aws-ram.md)

**Networking**
8. [VPC](domain-1-organizational/08-vpc.md)
9. [VPC Flow Logs](domain-1-organizational/09-vpc-flow-logs.md)
10. [AWS PrivateLink](domain-1-organizational/10-aws-privatelink.md)
11. [Route 53](domain-1-organizational/11-route-53.md)
12. [AWS Direct Connect (and Hybrid Networking)](domain-1-organizational/12-direct-connect.md)
13. [AWS Local Zones](domain-1-organizational/13-aws-local-zones.md)
14. [AWS Outposts](domain-1-organizational/14-aws-outposts.md)

**Network Security**
15. [AWS Network Firewall](domain-1-organizational/15-aws-network-firewall.md)
16. [AWS Firewall Manager](domain-1-organizational/16-aws-firewall-manager.md)

**Encryption & Key Management**
17. [KMS](domain-1-organizational/17-kms.md)
18. [CloudHSM](domain-1-organizational/18-cloudhsm.md)

**Security, Compliance & Audit**
19. [Security Findings (GuardDuty, Inspector, Detective, Security Hub)](domain-1-organizational/19-security-findings.md)
20. [AWS Config](domain-1-organizational/20-aws-config.md)
21. [CloudTrail](domain-1-organizational/21-cloudtrail.md)
22. [AWS Audit Manager](domain-1-organizational/22-aws-audit-manager.md)
23. [AWS Artifact](domain-1-organizational/23-aws-artifact.md)

**Operations & Monitoring**
24. [CloudWatch](domain-1-organizational/24-cloudwatch.md)
25. [AWS Systems Manager](domain-1-organizational/25-aws-systems-manager.md)
26. [Parameter Store](domain-1-organizational/26-parameter-store.md)

**Certificates**
27. [ACM (Amazon Certificate Manager)](domain-1-organizational/27-acm.md)

**Networking (additions)**
28. [CloudFront](domain-1-organizational/28-cloudfront.md)
29. [Global Accelerator](domain-1-organizational/29-global-accelerator.md)
30. [Site-to-Site VPN](domain-1-organizational/30-site-to-site-vpn.md)
31. [Client VPN](domain-1-organizational/31-client-vpn.md)
32. [VPC Peering (+ Transit Gateway recap)](domain-1-organizational/32-vpc-peering.md)

**Security (additions)**
33. [AWS WAF](domain-1-organizational/33-waf.md)
34. [AWS Shield](domain-1-organizational/34-shield.md)
35. [AWS Secrets Manager](domain-1-organizational/35-secrets-manager.md)
36. [Amazon Macie](domain-1-organizational/36-macie.md)

**Cost & Operations**
37. [AWS Cost Explorer](domain-1-organizational/37-cost-explorer.md)
38. [AWS Cost Anomaly Detection](domain-1-organizational/38-cost-anomaly-detection.md)
39. [AWS Trusted Advisor](domain-1-organizational/39-trusted-advisor.md)

**Identity (additions) & Zero-Trust**
40. [Amazon Cognito](domain-1-organizational/40-cognito.md)
41. [AWS Verified Access](domain-1-organizational/41-verified-access.md)

**End-User Computing & Security Ops**
42. [Amazon WorkSpaces (+ WorkSpaces Web + AppStream)](domain-1-organizational/42-workspaces.md)
43. [Penetration Testing in AWS](domain-1-organizational/43-pentest.md)

**Cost Management (additions)**
44. [AWS Budgets](domain-1-organizational/44-budgets.md)
45. [AWS Savings Plans (+ RI comparison)](domain-1-organizational/45-savings-plans.md)
46. [AWS Cost and Usage Report (CUR)](domain-1-organizational/46-cost-and-usage-report.md)

**Multi-Account Governance (additions)**
47. [AWS License Manager](domain-1-organizational/47-license-manager.md)
48. [AWS Service Catalog](domain-1-organizational/48-service-catalog.md)
49. [AWS Service Quotas](domain-1-organizational/49-service-quotas.md)

### Domain 2 — Design for New Solutions

**Compute**
1. [EC2 (Elastic Compute Cloud)](domain-2-new-solutions/01-ec2.md)
2. [EC2 Image Builder](domain-2-new-solutions/02-ec2-image-builder.md)
3. [AWS Compute Optimizer](domain-2-new-solutions/03-compute-optimizer.md)
4. [Lambda](domain-2-new-solutions/04-lambda.md)
5. [Containers (ECS, Fargate, ECR, EKS)](domain-2-new-solutions/05-containers.md)
6. [AWS Batch](domain-2-new-solutions/06-aws-batch.md)

**Storage**
7. [S3](domain-2-new-solutions/07-s3.md)
8. [EBS (Elastic Block Store)](domain-2-new-solutions/08-ebs.md)

**Databases**
9. [RDS (Relational Database Service)](domain-2-new-solutions/09-rds.md)
10. [Aurora](domain-2-new-solutions/10-aurora.md)
11. [DynamoDB](domain-2-new-solutions/11-dynamodb.md)
12. [ElastiCache (& MemoryDB)](domain-2-new-solutions/12-elasticache.md)
13. [Timestream](domain-2-new-solutions/13-timestream.md)
14. [Redshift](domain-2-new-solutions/14-redshift.md)

**Analytics**
15. [Athena](domain-2-new-solutions/15-athena.md)
16. [OpenSearch Service](domain-2-new-solutions/16-opensearch.md)
17. [Lake Formation](domain-2-new-solutions/17-lake-formation.md)
18. [AWS Glue](domain-2-new-solutions/18-glue.md)
19. [Amazon QuickSight](domain-2-new-solutions/19-quicksight.md)

**Orchestration & IaC**
20. [Step Functions](domain-2-new-solutions/20-step-functions.md)
21. [CloudFormation](domain-2-new-solutions/21-cloudformation.md)

**Data Protection & IoT**
22. [AWS Backup](domain-2-new-solutions/22-aws-backup.md)
23. [AWS IoT](domain-2-new-solutions/23-aws-iot.md)

**Messaging & Streaming**
24. [SQS](domain-2-new-solutions/24-sqs.md)
25. [SNS](domain-2-new-solutions/25-sns.md)
26. [EventBridge](domain-2-new-solutions/26-eventbridge.md)
27. [Amazon MQ](domain-2-new-solutions/27-amazon-mq.md)
28. [Kinesis (+ MSK)](domain-2-new-solutions/28-kinesis.md)

**Networking & APIs**
29. [Elastic Load Balancing (ELB)](domain-2-new-solutions/29-elb.md)
30. [API Gateway](domain-2-new-solutions/30-api-gateway.md)

**File Storage**
31. [EFS (Elastic File System)](domain-2-new-solutions/31-efs.md)
32. [FSx (Windows / Lustre / NetApp ONTAP / OpenZFS)](domain-2-new-solutions/32-fsx.md)

**Compute & Scaling**
33. [EC2 Auto Scaling](domain-2-new-solutions/33-ec2-auto-scaling.md)

**Niche Databases**
34. [Neptune + Keyspaces](domain-2-new-solutions/34-neptune-keyspaces.md)

**Generative AI & ML**
35. [Amazon Bedrock](domain-2-new-solutions/35-bedrock.md)
36. [Amazon SageMaker (AI + Unified Studio)](domain-2-new-solutions/36-sagemaker.md)

**Other Services (consolidated reference)**
37. [SES, SAM, App Runner, AppSync, Transfer Family, Signer, Elastic Beanstalk, EMR, Amazon Connect, Amazon Q](domain-2-new-solutions/37-other-services.md)

**Databases (additions)**
38. [Amazon DocumentDB (MongoDB-compatible)](domain-2-new-solutions/38-documentdb.md)

**Modernization & Platform**
39. [AWS Proton](domain-2-new-solutions/39-proton.md)
40. [AWS Amplify](domain-2-new-solutions/40-amplify.md)
41. [Amazon AppFlow](domain-2-new-solutions/41-appflow.md)

**Compute (additions)**
42. [Amazon Lightsail](domain-2-new-solutions/42-lightsail.md)
43. [AWS Wavelength](domain-2-new-solutions/43-wavelength.md)

**Developer Tools & Engagement**
44. [Amazon CodeGuru (Reviewer + Profiler + Security)](domain-2-new-solutions/44-codeguru.md)
45. [AWS Data Exchange](domain-2-new-solutions/45-data-exchange.md)
46. [Amazon Pinpoint](domain-2-new-solutions/46-pinpoint.md)
47. [AWS Device Farm](domain-2-new-solutions/47-device-farm.md)

**AI/ML Services (consolidated)**
48. [Comprehend, Polly, Transcribe, Rekognition, Textract, Translate, Lex, Kendra, Personalize, Fraud Detector](domain-2-new-solutions/48-ml-services.md)

**Niche Services (consolidated)**
49. [Managed Blockchain, Elastic Transcoder, IoT Things Graph](domain-2-new-solutions/49-niche-services.md)

### Domain 3 — Continuous Improvement for Existing Solutions

**Disaster Recovery & Observability**
1. [Disaster Recovery](domain-3-continuous-improvement/01-disaster-recovery.md)
2. [AWS Elastic Disaster Recovery (DRS)](domain-3-continuous-improvement/02-aws-drs.md)
3. [AWS X-Ray (+ OpenTelemetry)](domain-3-continuous-improvement/03-x-ray.md)

**Resilience & Chaos**
4. [AWS Resilience Hub](domain-3-continuous-improvement/04-resilience-hub.md)
5. [AWS Fault Injection Service (FIS)](domain-3-continuous-improvement/05-fault-injection-service.md)

**Governance & Health**
6. [AWS Well-Architected Tool](domain-3-continuous-improvement/06-well-architected-tool.md)
7. [AWS Health Dashboard](domain-3-continuous-improvement/07-aws-health-dashboard.md)

**Performance & Observability (additions)**
8. [Application Auto Scaling (+ AWS Auto Scaling)](domain-3-continuous-improvement/08-application-auto-scaling.md)
9. [CloudWatch Synthetics + RUM](domain-3-continuous-improvement/09-cloudwatch-synthetics-rum.md)
10. [Managed Grafana + Managed Prometheus (AMP)](domain-3-continuous-improvement/10-managed-grafana-prometheus.md)

### Domain 4 — Accelerate Workload Migration and Modernization

**Strategy**
1. [Cloud Migrations (7 R's)](domain-4-migration/01-cloud-migrations.md)

**Database & Server**
2. [Database Migration Service (DMS)](domain-4-migration/02-dms.md)
3. [Application Migration Service (MGN, ex-SMS)](domain-4-migration/03-mgn.md)

**Data Transfer**
4. [DataSync](domain-4-migration/04-datasync.md)
5. [AWS Snow Family](domain-4-migration/05-snow-family.md)
6. [Storage Gateway](domain-4-migration/06-storage-gateway.md)

**Modernization & CI/CD**
7. [CI/CD: CodePipeline + CodeBuild + CodeDeploy + CodeArtifact](domain-4-migration/07-cicd.md)
