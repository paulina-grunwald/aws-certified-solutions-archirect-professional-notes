# Containers on AWS (ECS, Fargate, ECR, EKS)

> **AWS's container stack: ECR stores images, ECS orchestrates them (with EC2 or serverless Fargate), EKS provides managed Kubernetes for portable workloads. Picking the right combination is a recurring SAP-C02 pattern. Bonus knowledge: ECS Express Mode (App Runner successor), ECS Service Connect (App Mesh successor), Fargate Spot.**

Maps to: **Domain 2.5 — High-performance architectures**, **Domain 4.3 — Container modernization**, **Domain 1.2 — Image supply-chain security**

---

## Container Decision Matrix

| Need | Service |
|---|---|
| Simplest container deployment, no infra knowledge | **ECS Express Mode** (replaces App Runner) |
| Serverless containers with ECS orchestration | **ECS + Fargate** |
| Full control over instances, GPU, Windows | **ECS + EC2** |
| Kubernetes compatibility required | **EKS** (Fargate or EC2 or Auto Mode) |
| Hybrid / on-premises containers managed from AWS | **ECS Anywhere** or **EKS Anywhere** |
| Batch processing of containerized jobs | **AWS Batch** (uses ECS under the hood) |
| ML model serving | **SageMaker** (containers under the hood) |
| Edge container workloads | **AWS Outposts** with ECS or EKS |

---

## ECS — Elastic Container Service

### Core Concepts

- **Cluster** — logical grouping of tasks/services running on EC2, Fargate, or external (ECS Anywhere) instances; multi-AZ
- **Task Definition** — JSON describing one or more containers that run together: image, CPU/memory, networking mode, IAM roles, logging, volumes
- **Task** — instantiation of a task definition (standalone or part of a Service)
- **Service** — maintains a desired count of tasks; integrates with load balancers; supports rolling, blue/green, canary deployments
- **Capacity Provider** — strategy for scaling infrastructure to meet task demand

### Launch Types

| Launch Type | You manage | Best for |
|---|---|---|
| **Fargate** | Nothing — AWS manages compute | Variable workloads, microservices, batch jobs, minimal ops |
| **EC2** | EC2 instances (patching, scaling, security) | GPU, Windows, specific instance types, persistent host storage, Reserved Instances / Savings Plans cost optimization |
| **ECS Anywhere** | Your on-prem / VMs / bare metal | Hybrid, data sovereignty, on-prem workloads orchestrated from AWS |

- **Fargate Spot** — up to 70% discount for fault-tolerant tasks; 2-minute interruption warning
- **ECS Anywhere** does **NOT** support Fargate, ALB/NLB integration, or Service Connect on external instances

### Networking Modes

| Mode | Description | Required for / common with |
|---|---|---|
| **awsvpc** (recommended) | Each task gets its own ENI + private IP; task-level security groups | **Required for Fargate**; recommended for EC2 launch type with ALB/NLB |
| **bridge** | Default Docker bridge on EC2; dynamic port mapping with ALB | Legacy EC2 launch type |
| **host** | Uses the host's network stack directly; one task per port per host | High-throughput, specific port binding |
| **none** | No external networking | Rare |

### Load Balancer Integration

- **ALB** — most common; supports path/host-based routing, dynamic port mapping (bridge mode), gRPC, HTTP/2
- **NLB** — TCP/UDP, ultra-low latency, static IPs, PrivateLink
- **CLB** — legacy, do not select for new designs
- With **awsvpc** mode, ALB/NLB targets the **task ENI directly** (no dynamic port mapping)

### Task Placement Strategies (EC2 launch type only — NOT Fargate)

- **binpack** — fewest instances (cost optimization)
- **random** — random distribution
- **spread** — even distribution across an attribute (e.g., `attribute:ecs.availability-zone`, `instanceId`)
- **Strategies are composable**: spread by AZ, then binpack by memory

**Task Placement Constraints**:
- **distinctInstance** — one task per container instance
- **memberOf** — Cluster Query Language expression (e.g., specific instance type)

### Auto Scaling

**Service Auto Scaling (task-level)** — Application Auto Scaling on the desired count:
- **Target Tracking** — `ECSServiceAverageCPUUtilization`, `ECSServiceAverageMemoryUtilization`, `ALBRequestCountPerTarget`
- **Step Scaling** — CloudWatch alarm with step adjustments
- **Scheduled Scaling** — known peaks

**Cluster Auto Scaling (EC2 launch type only)**:
- **Capacity Providers** automatically scale the underlying ASG to match task demand
- Target capacity (e.g., 100% = exactly match) governs over-/under-provisioning
- Recommended over manual ASG scaling policies
- Not needed with Fargate (Fargate scales infrastructure automatically)

### Capacity Providers

- Define how tasks are distributed across infrastructure
- Built-in: **FARGATE**, **FARGATE_SPOT**
- Custom: link to EC2 Auto Scaling Groups
- **Capacity Provider Strategy** — combine providers with weight + base
  - Example: `base=1` on `FARGATE`, then weight `1:FARGATE / 3:FARGATE_SPOT` → 25% on-demand / 75% Spot

### ECS Service Connect

- Service mesh for ECS — **replaces App Mesh** (discontinued **September 30, 2026**)
- Fully managed **Envoy sidecar** — no manual proxy management
- Built-in health checks, outlier detection, retries, round-robin load balancing
- Works across services within and across clusters/VPCs in the **same Region**
- Traffic metrics auto-emitted to CloudWatch
- Service discovery without DNS-based Cloud Map (but can integrate)
- **Single-Region only** — for cross-Region service mesh, look at PrivateLink / Global Accelerator / external mesh

### ECS Express Mode

- **Replaces AWS App Runner** (App Runner stops accepting new customers **April 30, 2026**)
- Single API call: container image + two IAM roles
- AWS provisions a complete stack: ECS service on Fargate, ALB, auto scaling, networking
- Preserves App Runner simplicity while unlocking the full ECS feature set
- Best for fast container deployment without deep ECS knowledge

### IAM Roles

- **Task IAM role** — permissions for application code inside the container (DynamoDB, S3, etc.)
- **Task execution role** — permissions for the ECS agent to pull images from ECR, fetch secrets from Secrets Manager / Parameter Store, push logs to CloudWatch
- **Don't confuse the two** — common exam trap

---

## AWS Fargate

- **Serverless** container compute for ECS and EKS
- AWS manages the underlying infrastructure; you only define CPU/memory per task/pod
- Supports Linux (default) and Windows containers
- Pay per task vCPU + memory per second (1-second granularity, 1-minute minimum)
- **Fargate Spot** — up to 70% discount; 2-min warning on interruption
- **Best for**: variable workloads, microservices, batch jobs, anything where you want zero infra management
- **Compute Savings Plans** apply to Fargate (Compute SP, not EC2 Instance SP)

---

## Amazon ECR — Elastic Container Registry

### Core

- Fully managed **OCI / Docker container registry**
- **Private** repositories + **Public Gallery** (`public.ecr.aws/<alias>`)
- Stores OCI artifacts beyond Docker images (Helm charts, AI/ML models)
- Backed by S3 under the hood (you don't see the bucket)
- IAM (identity) + repository policy (resource, supports cross-account)
- **Authentication**: `aws ecr get-login-password | docker login` — token valid **12 hours**; long-running workloads use ECR credential helper or task/instance role for auto-refresh

### Encryption at Rest

- AWS-managed key (default) or **customer-managed KMS key**
- Envelope encryption per image
- **Encryption choice is locked at repository creation** — cannot change later, only re-create
- KMS Grant: `kms:DescribeKey`, `kms:Decrypt`, `kms:GenerateDataKey`, `kms:RetireGrant`

### Image Scanning

| Mode | Engine | Trigger | Scope |
|---|---|---|---|
| **Basic Scanning** | Open-source CVE DB (Clair-based) | On-push or manual | OS packages |
| **Enhanced Scanning** | **Amazon Inspector** | Continuous (initial + re-scan on new CVEs) | OS + language packages (Python, Node, Java, Ruby, Go, .NET) |

- **Scan Filters** — wildcard patterns selecting which repos scan (e.g., `prod*`)
- Findings → Inspector / Security Hub / EventBridge for auto-remediation
- Pair with **EC2 Image Builder** for hardened, scanned golden images

### Lifecycle Policies

- JSON rules per repository
- Common: keep last N tagged images, expire untagged after X days, expire `dev-*` tags
- Evaluated **daily**

### Immutable Tags

- Once enabled, a tag (e.g., `v1.2.3`) cannot be overwritten
- Prevents production drift; recommended for release tags

### Cross-Region Replication

- Automatic replication to multiple Regions (same- or cross-account)
- Use cases: low-latency regional pulls, DR, multi-Region active/active

### Pull-Through Cache

- ECR caches upstream images on first pull, serves subsequent pulls locally
- Supported: **Docker Hub** (auth via Secrets Manager), **ECR Public**, **Quay**, **`registry.k8s.io`**, **`ghcr.io`**, **`mcr.microsoft.com`**, **GitLab Container Registry**
- Benefits: avoid Docker Hub rate limits / outages, Inspector scanning of upstream images, KMS-encrypted local copy

### VPC Endpoints (private subnet usage)

You need **all three**:
- `com.amazonaws.<region>.ecr.api` — API calls
- `com.amazonaws.<region>.ecr.dkr` — image push/pull
- **S3 gateway endpoint** — underlying layer storage

Missing any of the three → "image pull failed in private subnet" exam scenario.

### Signing & Verification

- **AWS Signer Container** + **cosign** for image signature verification
- Combine with admission controllers (Kyverno, OPA Gatekeeper on EKS) for "only signed images run"
- Supply-chain integrity (SLSA)

---

## EKS — Elastic Kubernetes Service (high level)

### Highlights

- Managed Kubernetes **control plane** (multi-AZ); **$0.10/hour per cluster** (~$73/month)
- Compute options: **EC2 self-managed**, **EC2 managed node groups**, **Fargate**, **EKS Auto Mode** (Karpenter-managed compute, similar to Fargate experience)
- **EKS Anywhere** for on-prem
- **EKS Distro** (open source) — same Kubernetes distribution you can run anywhere
- Scales up to **100,000 nodes per cluster**
- **Kubernetes 1.35** is current; AWS supports several versions in parallel with predictable EOL dates
- **AWS Ingress NGINX retiring March 2026** → migrate to **Gateway API**

### EKS Auto Mode

- AWS manages worker nodes, scaling, patching, networking, monitoring
- Built on Karpenter for just-in-time provisioning
- Closest experience to "Fargate for EKS" but supports DaemonSets and more pod types

### Networking & IAM

- **VPC CNI** — pod gets a VPC IP (large IP usage; use prefix delegation or alternatives like Cilium for IP efficiency)
- **IAM Roles for Service Accounts (IRSA)** — pods assume IAM roles via OIDC federation
- **Pod Identity** — simpler alternative to IRSA, agent on the node distributes credentials
- **AWS PrivateLink for the EKS API server endpoint** — private cluster
- **Calico / Cilium** as alternate CNI for network policies, eBPF

### Security & Compliance

- **EKS Pod Identity** / IRSA for least-privilege pod IAM
- **AWS GuardDuty EKS Protection** — runtime + audit-log threat detection
- **Inspector** scans EKS workloads (via ECR + runtime)
- **Security Hub controls** for EKS
- **OPA Gatekeeper / Kyverno** admission controllers for policy enforcement

---

## ECS vs EKS Decision

| Factor | ECS | EKS |
|---|---|---|
| Orchestrator | AWS proprietary | Kubernetes (open source) |
| Learning curve | Lower | Higher (K8s expertise required) |
| Portability | AWS-only | Multi-cloud / hybrid |
| Ecosystem | AWS-native | Massive K8s ecosystem (Helm, Istio, etc.) |
| Control plane cost | Free | $0.10/hour per cluster (~$73/month) |
| Best for | AWS-native, simpler microservices | K8s-skilled orgs, multi-cloud strategy, complex orchestration |

---

## App Runner & App Mesh (retiring)

- **App Runner** — stops accepting new customers **April 30, 2026** → use **ECS Express Mode**
- **App Mesh** — discontinued **September 30, 2026** → use **ECS Service Connect** (ECS) or **Istio / Envoy / Linkerd on EKS** (EKS)

---

## Exam Tips

- "Least operational overhead for containers, no K8s requirement" → **ECS + Fargate**
- "Kubernetes" or "K8s" → **EKS**
- "Run containers on-premises, manage from AWS" → **ECS Anywhere** (or EKS Anywhere for K8s)
- "Service-to-service comms in ECS" → **ECS Service Connect** (App Mesh is retiring)
- "Simplest container deploy, no infra config" → **ECS Express Mode** (App Runner is retiring)
- "Dynamic port mapping" → ALB + ECS bridge mode OR awsvpc mode
- "Task-level security groups" → **awsvpc** networking mode (required for Fargate)
- "Scale tasks based on ALB request count" → Service Auto Scaling with `ALBRequestCountPerTarget`
- "Cluster scales to match task demand" → **Capacity Providers** (not manual ASG policies)
- "Cost optimization on EC2 launch type" → **binpack** placement strategy
- "Save 70% on fault-tolerant containers" → **Fargate Spot** (or EC2 Spot + Capacity Provider)
- "Continuous CVE scanning + language packages" → ECR **Enhanced Scanning** (Inspector)
- "Docker Hub rate limits / outage protection" → ECR **Pull-Through Cache**
- "Prevent prod tag overwrite" → ECR **immutable tags**
- "Auto-cleanup of old images" → ECR **lifecycle policies**
- "Low-latency container pulls across Regions" → ECR **cross-Region replication**
- "Pulling images in a private subnet" → 2× ECR interface VPC endpoints (`ecr.api` + `ecr.dkr`) + S3 gateway endpoint
- "K8s pods need IAM permissions" → **IRSA** or **Pod Identity** (Pod Identity is simpler and current default)
- "Container vulnerability + supply chain" → Image Builder → ECR + Enhanced Scanning + Signer/cosign + admission controller

---

## Exam Traps

- **ECS Anywhere does NOT support Fargate** — external instances are customer-managed
- **App Runner & App Mesh are retiring** — never the right answer for a new architecture
- **Capacity Providers ≠ Service Auto Scaling** — Capacity Providers scale the **cluster infrastructure**; Service Auto Scaling adjusts **task count**
- **Task placement strategies (binpack/spread/random) only apply to EC2 launch type**, NOT Fargate
- **awsvpc is the ONLY networking mode supported by Fargate** — if a question mentions `bridge` or `host`, it must be EC2 launch type
- **ECS control plane is free; EKS is $0.10/hour per cluster** — cost optimization questions exploit this
- **Fargate Spot interruption 2-minute warning** — not suitable for stateful / time-critical workloads
- **ECS Service Connect is single-Region** — cross-Region service mesh needs different tools
- **Task IAM role ≠ Task execution role** — Task role = app permissions; execution role = ECS agent permissions (pull image, fetch secrets, push logs)
- **ECR encryption choice is one-shot** — locked at repository creation; re-create the repo to change
- **VPC endpoints for ECR**: need BOTH `ecr.api` and `ecr.dkr` + S3 gateway. Missing any → pull failure
- **ECR Basic Scanning ≠ Enhanced Scanning** — Basic = OS packages only on-push; Enhanced = Inspector, continuous, OS + language
- **ECR lifecycle policies are per-repository**, NOT registry-wide
- **Immutable tags don't prevent `latest` from moving** unless you apply immutability to that specific tag
- **EKS pod IPs come from the VPC CIDR** — IP exhaustion is real; plan with prefix delegation or alternate CNI
- **AWS Ingress NGINX retires March 2026** — migrate to **Gateway API**
- **Pull-through cache only caches on first pull** — changes to upstream tags require re-pull or expiry
- **ECR replication is opt-in, not automatic** — multi-Region resiliency requires explicit configuration
