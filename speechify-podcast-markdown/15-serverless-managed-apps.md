# Podcast 15 — Serverless & Managed Apps

**Length target**: 15 min
**Repo references**: `domain-2-new-solutions/04-lambda.md`, `06-aws-batch.md`, `37-other-services.md` (App Runner, SAM, Beanstalk), `42-lightsail.md`, `39-proton.md`

## Topic & Scope

Lambda is heavily tested; the surrounding "managed app" tier (App Runner, Beanstalk, Lightsail, Proton, Batch) creates distractors. Sort them.

## Service Coverage Depth

**Deep**:
- Lambda — runtime, layers, concurrency, provisioned concurrency, VPC config, destinations, event sources
- App Runner vs Elastic Beanstalk vs Lightsail decision

**Brief**:
- AWS Batch (managed batch with job queues)
- AWS SAM (Lambda IaC)
- AWS Proton (platform engineering)

## Structured Outline

1. **Open (30s)** — "Lambda gets the most love. But App Runner, Beanstalk, Lightsail, Batch all appear as distractors. Let's draw the lines."
2. **Lambda Deep (5 min)** — execution model, runtimes, layers, environment variables, VPC config (ENI), reserved + provisioned concurrency, destinations, event source mappings (SQS, Kinesis, DynamoDB Streams), error handling
3. **Event Sources & Patterns (2 min)** — API Gateway, EventBridge, SQS, S3, DynamoDB Streams, Kinesis, ALB
4. **App Runner (2 min)** — managed containerized HTTP service, auto-deploys from GitHub/ECR, scales to zero
5. **Elastic Beanstalk (1.5 min)** — opinionated PaaS for EC2-based web apps, multiple deployment modes
6. **Lightsail + Batch + Proton (2 min)** — Lightsail = simple VPS distractor; Batch = job queues; Proton = platform engineering templates
7. **Decision Matrix + Rapid-Fire (2 min)**

## Must-Mention Exam Tips

- Lambda max: 15-min timeout, 10 GB memory, 10 GB ephemeral /tmp, 250 MB unzipped package (or 10 GB container image)
- Lambda concurrency: account-level (1,000 default), reserved (cap per function), provisioned (pre-warmed)
- Lambda cold start mitigation: provisioned concurrency, smaller package, SnapStart for Java
- Lambda in VPC: ENI created in subnets, can talk to RDS / ElastiCache in VPC; no internet unless NAT GW
- Event source mappings (poll-based): SQS, Kinesis, DynamoDB Streams, MSK, MQ, Self-managed Kafka
- Async invocation: destinations for success/failure (SNS, SQS, Lambda, EventBridge); DLQ for fallback
- Lambda Layers: shared dependencies up to 5 layers per function
- App Runner: managed container web service, automatic scaling + HTTPS, scales to zero
- Elastic Beanstalk: managed EC2-based PaaS, supports Docker / Java / .NET / Node / Python / Ruby
- AWS Batch: managed job orchestration on EC2 / Fargate / Spot with job queues + compute environments
- AWS SAM: Lambda-focused IaC (extends CloudFormation)
- Proton: platform engineering tool — environment + service templates for dev teams

## Must-Mention Exam Traps

- EXAM TRAP: Lambda max execution is 15 minutes — for longer, use Step Functions / ECS / EC2 / Batch
- EXAM TRAP: Lambda in VPC adds cold-start latency (ENI provisioning) — improved with Hyperplane ENI
- EXAM TRAP: Provisioned concurrency costs money even when not used — only for latency-sensitive paths
- EXAM TRAP: Lambda reserved concurrency CAPS — limits how much that function can scale
- EXAM TRAP: SQS DLQ on Lambda is for failures the function couldn't process — not for batch DLQ
- EXAM TRAP: App Runner ≠ Elastic Beanstalk — App Runner is container; Beanstalk is EC2-based
- EXAM TRAP: Lightsail is rarely the right answer in SAP-C02 — usually distractor for "predictable cheap"
- EXAM TRAP: Batch ≠ Step Functions — Batch is job queues at scale; Step Functions is workflow orchestration
- EXAM TRAP: Beanstalk creates resources you might not see — uses CloudFormation under the hood
- EXAM TRAP: Proton ≠ Service Catalog — Proton is continuous platform tooling; Catalog is one-shot products
- EXAM TRAP: Lambda function URLs are a feature for HTTPS endpoint — not always best vs API Gateway (no auth/throttling/caching features)
- EXAM TRAP: SAM ≠ CDK — both are IaC; SAM is Lambda-focused YAML, CDK is general-purpose code
- EXAM TRAP: Lambda response size max 6 MB sync, 256 KB async invocation payload

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| Event-driven short function | Lambda |
| Long-running job > 15 min | ECS / Fargate / Batch / EC2 |
| Containerized web service, auto-scale to zero | App Runner |
| EC2-based managed PaaS, multiple languages | Elastic Beanstalk |
| Massive batch jobs with queue/priority | AWS Batch |
| Personal blog / WordPress on AWS | Lightsail |
| Platform engineering templates for many teams | Proton |
| Reduce Lambda cold starts on critical path | Provisioned Concurrency / SnapStart |
| Lambda access to RDS in private subnet | Lambda in VPC + ENI |

## Tone & Style

- Lead with Lambda — most exam questions live there
- Frame App Runner as "Lambda for containers (HTTP only)"
- Lightsail mentioned as the trap distractor

## Rapid-Fire Closer

"Lambda max runtime?" — "15 min." "App Runner workload?" — "Container HTTP." "Beanstalk underlying?" — "EC2 PaaS." "Lightsail right answer?" — "RARELY." "Lambda in VPC needs internet?" — "NAT Gateway."
