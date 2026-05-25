# Podcast 16 — Containers

**Length target**: 15 min
**Repo references**: `domain-2-new-solutions/05-containers.md`

## Topic & Scope

ECS, EKS, Fargate, ECR. Container choice is a frequent SAP-C02 scenario. The deciding factor is usually "do you want Kubernetes API" + "do you want to manage infrastructure".

## Service Coverage Depth

**Deep**:
- ECS — task definitions, services, Fargate vs EC2 launch type, capacity providers
- EKS — control plane managed by AWS, node groups (managed / self-managed / Fargate), networking (VPC CNI)
- Fargate — serverless compute for ECS + EKS
- ECR — image registry, replication, scanning

**Brief**:
- ECS Anywhere + EKS Anywhere (on-prem)
- App Mesh (mention as retiring service)

## Structured Outline

1. **Open (30s)** — "Two orchestrators, one serverless compute, one registry. The exam tests which combination fits."
2. **ECS (4 min)** — task definitions (containers, CPU/mem, IAM role, secrets), services (desired count, load balancer), launch types: Fargate (serverless) vs EC2 (you manage), capacity providers
3. **EKS (4 min)** — managed Kubernetes control plane, node groups (managed / self-managed / Fargate), VPC CNI, IAM Roles for Service Accounts (IRSA), Cluster Autoscaler vs Karpenter
4. **Fargate (2 min)** — no EC2 management, per-task billing, works with both ECS and EKS, ideal for variable / spiky workloads
5. **ECR (2 min)** — private + public registry, image scanning, cross-region replication, lifecycle policies, IAM-based access
6. **ECS vs EKS Decision (1.5 min)** — Kubernetes API vs AWS-native
7. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- ECS task definition: JSON spec for containers, CPU, memory, IAM role, secrets, networking
- ECS task IAM role: container-level credentials via metadata service
- ECS execution role: pull image + push logs (Fargate / awslogs driver)
- ECS launch types: Fargate (serverless) vs EC2 (you manage instances)
- ECS Capacity Providers: Fargate, Fargate Spot, EC2-backed ASG
- EKS control plane: AWS-managed, $0.10/hour per cluster
- EKS node options: managed node groups (AWS handles AMI + lifecycle), self-managed nodes, Fargate profiles (per-pod serverless)
- IRSA (IAM Roles for Service Accounts): pod-level AWS credentials via OIDC
- Karpenter: AWS-native cluster autoscaler, faster + smarter than Cluster Autoscaler
- ECR: image scanning (basic Clair-based, enhanced via Inspector v2), cross-region replication, lifecycle policies
- ECS Anywhere: run ECS tasks on on-prem servers; EKS Anywhere: managed EKS on customer hardware
- Fargate Spot: up to 70% off ECS Fargate tasks (interruptible)

## Must-Mention Exam Traps

- EXAM TRAP: ECS ≠ EKS — ECS is AWS-native; EKS is Kubernetes
- EXAM TRAP: ECS task IAM role ≠ execution role — task role is for the app; execution role is for ECS infrastructure (pull image, send logs)
- EXAM TRAP: Fargate scales per-task pricing — not per-instance; no shared compute
- EXAM TRAP: EKS control plane has hourly cost ($0.10/hr per cluster) — ECS control plane is free
- EXAM TRAP: EKS Fargate runs ONE pod per Fargate task — no DaemonSets on Fargate
- EXAM TRAP: IRSA requires OIDC identity provider associated with cluster — not on by default
- EXAM TRAP: ECS Service Discovery uses Cloud Map (private DNS) — not Route 53 alone
- EXAM TRAP: ECR replication is configured per repo + destination region — not by default
- EXAM TRAP: ECR image scanning has two tiers: basic (free, weak) vs enhanced (Inspector v2, paid, deep)
- EXAM TRAP: Fargate tasks have no SSH/exec by default — enable ECS Exec for shell access
- EXAM TRAP: App Mesh is being retired — don't pick for new designs; use VPC Lattice
- EXAM TRAP: ALB-to-target on ECS Fargate uses target type "IP" — not "instance"

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| AWS-native containers, simplest | ECS on Fargate |
| Kubernetes API required (CI/CD, GitOps, third-party tools) | EKS |
| Serverless containers, per-task pricing | Fargate (with ECS or EKS) |
| Cost-optimized batch containers | Fargate Spot |
| Container registry with cross-region replication | ECR |
| Vulnerability scanning of images deeply | ECR Enhanced + Inspector v2 |
| Run ECS tasks on customer hardware | ECS Anywhere |
| Managed EKS on customer hardware | EKS Anywhere (or EKS Distro for self-managed) |
| Pod-level AWS permissions | EKS + IRSA |
| Cluster autoscaling with launch flexibility | Karpenter |

## Tone & Style

- Frame: "AWS-native vs Kubernetes, serverless vs you-manage"
- Repeat task role vs execution role distinction
- Mention Fargate cost model ("per task, not per instance")

## Rapid-Fire Closer

"Kubernetes API?" — "EKS." "AWS-native simple?" — "ECS." "Serverless containers?" — "Fargate." "Container registry?" — "ECR." "Pod-level IAM?" — "IRSA." "EKS control plane free?" — "NO, $0.10/hr."
