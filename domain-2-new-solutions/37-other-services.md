# Other SAP-C02 Services (Consolidated Reference)

> **One-page reference for SAP-C02 services that appear as occasional distractors or "right tool for the job" answers but don't warrant a full standalone note: SES, SAM, App Runner, AppSync, Transfer Family, Signer, Elastic Beanstalk, EMR, Amazon Connect, Q Business. Memorize each one's "when to pick" line.**

Maps to: **Domain 2 — multiple subdomains**, **Domain 4.3 — Modernization**

---

## Amazon SES (Simple Email Service)

- **Managed email send / receive** at scale
- **Inbound + outbound** SMTP / API
- Built-in DKIM, SPF, DMARC signing
- **Dedicated IP pools** for reputation isolation
- **Configuration sets** for per-message tracking + event publishing
- **Mailbox simulator** for testing without hitting real inboxes
- Use cases: transactional email (signup, password reset, receipts), bulk marketing, app notifications, app-to-app email
- Pricing: $0.10 per 1,000 emails (from EC2: $0); inbound $0.10 per 1,000

**Pick when**: "send transactional email at scale" or "process inbound email programmatically" → **SES**.
**Don't pick**: as a notification fan-out service → use **SNS**.

---

## AWS SAM (Serverless Application Model)

- **Open-source IaC framework** for serverless apps
- **Transform of CloudFormation** — SAM templates compile to CloudFormation
- Native primitives: `AWS::Serverless::Function`, `Api`, `HttpApi`, `StateMachine`, `Layer`
- **SAM CLI** for local dev + test: `sam local invoke`, `sam local start-api`, `sam deploy --guided`
- Built-in patterns for Lambda + API Gateway + DynamoDB + EventBridge + Step Functions
- **SAM Pipelines** scaffolds CodePipeline / GitHub Actions / GitLab / Jenkins

**Pick when**: serverless-first IaC with shorter syntax than raw CloudFormation; local Lambda testing.
**Don't pick**: when AWS CDK gives more language flexibility (TypeScript, Python, Java, Go, .NET).

---

## AWS App Runner

- **Fully managed container-based PaaS**
- Deploy from source (GitHub repo) or container image (ECR)
- Auto-build, auto-deploy, auto-scale, HTTPS endpoint provisioned
- VPC connector for private resource access (RDS, ElastiCache)
- Pay per concurrent request + compute time
- No K8s knowledge needed

**Pick when**: small team wants to ship a containerized web service without learning ECS/EKS; "container deployment with minimum ops".
**Don't pick**: complex workloads needing K8s features → EKS; long-running workers → ECS Fargate.

---

## AWS AppSync (GraphQL APIs)

- **Managed GraphQL service**
- Real-time subscriptions, offline sync, conflict resolution
- Data sources: DynamoDB, Aurora Serverless, Lambda, OpenSearch, HTTP, EventBridge
- Auth via Cognito User Pools, API Key, IAM, OIDC, Lambda authorizer
- **AppSync Events** — WebSocket pub/sub channels for real-time apps
- **AppSync Merged APIs** — combine multiple source APIs into one client endpoint

**Pick when**: GraphQL frontend with mobile/web; real-time subscriptions for chat / collaboration / live dashboards.
**Don't pick**: simple REST CRUD → API Gateway + Lambda.

---

## AWS Transfer Family

- **Fully managed SFTP / FTPS / FTP / AS2** to S3 or EFS
- Replaces self-managed SFTP servers
- Identity sources: Service Managed (Cognito-like), AWS Directory Service, custom Lambda authorizer
- **AS2 protocol** for EDI / B2B partner integration
- Per-protocol-server hourly + per-GB transferred pricing

**Pick when**: legacy partners still use SFTP / AS2; B2B file exchange; replacing on-prem MFT.
**Don't pick**: HTTP file uploads → S3 multipart + presigned URLs; large bulk transfers → DataSync / Snow Family.

---

## AWS Signer

- **Code signing service** — sign Lambda packages, IoT firmware, container images
- Integrates with **CodeDeploy** (signed Lambda packages) and **Lambda** (only verified code runs)
- **Signing profiles** define key material + signature validity
- Combine with **AWS Notation** for OCI container signing (cryptographic supply-chain)

**Pick when**: regulated environment requires cryptographically signed deployable artifacts; supply-chain integrity.
**Don't pick**: just need IAM permissions to deploy → use IAM, not Signer.

---

## AWS Elastic Beanstalk

- **PaaS** for web apps + workers: Java, .NET, PHP, Node.js, Python, Ruby, Go, Docker
- Manages EC2 + ELB + Auto Scaling + RDS automatically
- **Deployment policies**: All-at-once, Rolling, Rolling-with-additional-batch, Immutable, Blue/Green
- `.ebextensions/*.config` for custom config
- **Saved configurations** + environment cloning for repeatable deploys
- **Worker tier** environments use SQS for async job processing

**Pick when**: app teams want quick deploy of web stack; minimal infra knowledge needed.
**Don't pick**: container-first → ECS / EKS / App Runner; serverless-first → Lambda + API GW; full control → raw EC2 + CFN.

---

## Amazon EMR (Elastic MapReduce)

- **Managed big-data platform**: Spark, Hive, Presto/Trino, HBase, Flink, Hudi, Iceberg
- Cluster types: **EMR on EC2**, **EMR on EKS**, **EMR Serverless**, **EMR on Outposts**
- Spot Instances + Capacity Reservations for cost savings
- **EMR Studio** notebooks (managed Jupyter)
- Integrates with Lake Formation, Glue Data Catalog, S3 as primary storage
- **EMR Serverless** auto-scales compute per job

**Pick when**: large-scale Spark / Hive / Presto workloads; existing Hadoop ecosystem; ML training at scale.
**Don't pick**: ad-hoc SQL on S3 → Athena; visual ETL → Glue Studio; managed PostgreSQL warehouse → Redshift.

---

## Amazon Connect

- **Cloud contact center** (call center as a service)
- Inbound + outbound voice + chat + tasks
- **Contact Flows** = visual drag-drop IVR designer
- Integrates with Lex (NLP), Lambda (custom logic), Polly (TTS), Transcribe (call recording), Comprehend (sentiment), Kinesis (stream call data)
- **Customer Profiles** unifies customer data
- **Wisdom / Agent Assist** uses GenAI to suggest responses

**Pick when**: replace on-prem call center (Avaya, Cisco) with cloud; AI-powered IVR + agent assist.
**Don't pick**: SMS / push notifications → Pinpoint / SNS.

---

## Amazon Q (Business + Developer + Connect)

- **Generative AI assistant** family
- **Q Business** — connects to corporate data sources (S3, Confluence, SharePoint, Slack, Salesforce, ServiceNow, M365), answers employee questions with RAG
- **Q Developer** — code completion + chat in IDE (replaces CodeWhisperer); transforms / refactors code, generates tests, agent for AWS operations
- **Q in Connect** — agent-assist in contact center
- **Q in QuickSight** — natural language BI

**Pick when**: end-user GenAI app on top of AWS without DIY Bedrock plumbing; "AI assistant for our employees / developers".
**Don't pick**: building a custom GenAI feature in your own app → use **Bedrock** directly.

---

## Decision Matrix — "Right Tool for the Job"

| Scenario | Pick |
|---|---|
| Send transactional / marketing email | **SES** |
| Receive inbound email and trigger Lambda | **SES** |
| GraphQL API with subscriptions | **AppSync** |
| SFTP server for legacy partners | **Transfer Family** |
| B2B EDI / AS2 | **Transfer Family** |
| Containerized web app, low-ops | **App Runner** |
| PaaS for web stack (Java, .NET, Python) | **Elastic Beanstalk** |
| Serverless IaC framework | **SAM** (or CDK) |
| Sign Lambda packages / firmware | **Signer** |
| Big data Spark / Hive / Presto | **EMR** (or **EMR Serverless** for no-ops) |
| Ad-hoc SQL on S3 | **Athena** (not EMR) |
| Cloud contact center | **Amazon Connect** |
| Employee AI assistant on corp data | **Amazon Q Business** |
| Developer AI coding assistant | **Amazon Q Developer** |
| Custom GenAI in your own app | **Bedrock** (not Q) |

---

## Exam Tips

- **SES** is the email service; **SNS** is the pub/sub / fan-out service — they're different
- **SAM** = CloudFormation transform for serverless; **CDK** is multi-language IaC
- **App Runner** is the easy-mode container PaaS; ECS / EKS for full control
- **AppSync Events** added WebSocket pub/sub channels — distinct from GraphQL subscriptions
- **Transfer Family** is the only managed AS2 / SFTP / FTPS service on AWS
- **Signer** is the answer for "cryptographically sign Lambda packages / containers"
- **Elastic Beanstalk Worker tier** uses SQS — useful for async background jobs
- **EMR Serverless** auto-scales — cheaper than always-on cluster for bursty jobs
- **Connect** is the contact-center-as-a-service; pairs with Lex / Bedrock / Lambda
- **Q Business** consumes corporate data with built-in connectors and IAM Identity Center auth
- **Q Developer** replaced CodeWhisperer
- **Bedrock vs Q**: Bedrock is the building block (FMs as API); Q is the end-user product

## Exam Traps

- **SES NOT for instant messaging / push** — that's SNS / Pinpoint / mobile-push
- **SES sandbox** is opt-out (production access requires AWS request) — first 24 hours after account creation
- **SAM templates ARE CloudFormation** — same resource limits + drift behavior
- **App Runner doesn't replace EKS** for K8s-native features (operators, CRDs, custom scheduling)
- **AppSync subscriptions = WebSocket** under the hood — IoT / mobile clients need to handle reconnect logic
- **Transfer Family pricing is per-protocol-server-hour** + per-GB transferred — can be expensive for low-volume use cases
- **Elastic Beanstalk environments live in your account VPC** — but the EB management plane is AWS-managed
- **EMR is NOT Athena** — EMR is cluster-based (even Serverless EMR); Athena is ad-hoc SQL
- **EMR on EKS uses your existing EKS cluster** — share cluster with other workloads
- **Connect is not Connect Customer Profiles** — Profiles is a separate sub-service for unified CX data
- **Amazon Q ≠ Bedrock** — Q is built on Bedrock under the hood; you can't customize Q deeply, only Q Business connectors
- **Q Developer is not just code completion** — it has agents for transforms, refactor, doc gen, AWS resource queries
- **Q Business pricing is per user/month** — significant ongoing cost vs Bedrock per-token
- **CodeWhisperer is renamed to Q Developer** — older study materials use the old name
