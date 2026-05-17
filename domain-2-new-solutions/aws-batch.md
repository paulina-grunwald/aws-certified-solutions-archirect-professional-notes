# AWS Batch

> **Fully managed service that runs hundreds of thousands of batch computing jobs on AWS. Dynamically provisions the optimal quantity and type of compute (CPU / memory / GPU) based on job requirements.**

Maps to: **Domain 2.5 — Design high-performance architectures** (compute selection for batch / HPC workloads)

---

## Overview

- Optimized for batch computing — workloads that scale through the execution of **many jobs in parallel**
- Plans, schedules, and executes batch workloads across the full range of AWS compute services: **Fargate, EC2 (On-Demand and Spot), EKS**
- Dynamically provisions **optimal quantity and type** of compute (CPU / memory / GPU optimized) based on submitted jobs
- **You pay only for the underlying compute** (EC2 / Fargate / EKS) — Batch orchestration itself is free
- Integrates with **Amazon EventBridge** (formerly CloudWatch Events) for asynchronous job state changes — eliminates polling

---

## Use Cases

- Deep learning training and inference
- Genomics and life sciences analysis
- Financial risk modeling, Monte Carlo simulations
- Animation rendering, media transcoding
- Image processing pipelines
- Engineering and scientific simulations (HPC)
- ETL and large-scale data preprocessing

---

## Components

- **Jobs** — unit of work executed by Batch. Containerized; can reference an executable, a script, or a full Docker image. Runs on EC2, Fargate, or EKS.
- **Job Definitions** — blueprint for a job: container image, vCPU / memory, environment variables, IAM role, retry strategy, timeout, mount points, etc. Versioned (like ECS task definitions).
- **Job Queues** — where jobs wait to be scheduled. Multiple queues per Region with **different priorities** (higher priority drains first). Each queue maps to one or more compute environments with **CE ordering**.
- **Job Scheduling** — Batch evaluates the queue and dispatches jobs based on resource requirements, priorities, and dependencies.
- **Compute Environment** — pool of compute resources (EC2 / Fargate / EKS) where jobs actually run. Managed or unmanaged.

### Job State Lifecycle

```
SUBMITTED → PENDING → RUNNABLE → STARTING → RUNNING → SUCCEEDED | FAILED
```

- `PENDING` — waiting on a dependency
- `RUNNABLE` — ready to run, waiting for resources
- `STARTING` — scheduled and starting up

---

## Compute Environments

### Managed Compute Environment
- AWS Batch manages the capacity and instance types within the environment
- Choose **EC2 On-Demand** or **EC2 Spot** Instances
- Choose **Fargate** On-Demand or **Fargate Spot**
- Choose **EKS** (Batch on EKS, GA since May 2022) for Kubernetes-native batch
- Set a maximum vCPUs and (for Spot) a maximum bid price
- Launched within your own VPC
  - Private subnet must have access to **ECS service** (NAT GW or **VPC Endpoints for ECS** + ECR + Logs + S3)
  - Fargate jobs also need access to ECR, Logs, S3 endpoints

### Unmanaged Compute Environment
- You control and manage EC2 instance configuration, provisioning, and scaling
- Useful when you need custom AMIs, special instance configurations, or your own ASG
- Less common — managed is usually preferred

---

## Allocation Strategies

When using EC2 in a managed compute environment, Batch picks instances based on the allocation strategy:

- **BEST_FIT** (default for legacy) — picks the least expensive instance type that fits the job. May queue jobs if that instance type isn't available. Beginning April 2024, newly-created BEST_FIT CEs default to launch templates.
- **BEST_FIT_PROGRESSIVE** — picks the least expensive instances; if not available, progressively tries other instance types. May temporarily exceed your specified vCPU max.
- **SPOT_CAPACITY_OPTIMIZED** — chooses instances from the **deepest Spot capacity pools**. Best for Spot, minimizes interruption risk.
- **SPOT_PRICE_CAPACITY_OPTIMIZED** (August 2023) — balances Spot price and capacity depth. Recommended over SPOT_CAPACITY_OPTIMIZED for cost-conscious Spot workloads.

> 💡 **Exam pick**: scenario says "minimize Spot interruptions for batch jobs" → **SPOT_CAPACITY_OPTIMIZED** or **SPOT_PRICE_CAPACITY_OPTIMIZED**. Default BEST_FIT can stall if cheapest instance is unavailable.

---

## Job Queues — Priority and CE Ordering

- Multiple queues per Region; each queue has a **priority** (higher value = higher priority)
- Each queue maps to **one or more compute environments**, evaluated in order
- **Use case**: a high-priority production queue draining first; a low-priority research queue draining when capacity is available
- Within a queue, jobs are FIFO by default (first-in-first-out) unless using **fair-share scheduling**

---

## Fair-Share Scheduling

- Scheduling policy that lets you manage **many users or workloads in a single job queue** with fair compute allocation
- Each job tagged with a `shareIdentifier` (e.g., team or user)
- Each share identifier has a **weightFactor** (default 1) — defines the ratio of compute capacity allocated
- Configurable **fairness-over-time** parameter (lookback window for historical usage)
- **Compute reservation** — guarantee a minimum share for specific identifiers
- **Queue and Share Utilization Visibility** (Feb 2026) — Batch surfaces capacity used by FIFO + fair-share queues per allocation, in job queue snapshots

> 💡 **Exam pattern**: scenario says "multiple teams sharing one queue, ensure fair allocation over time" → **Fair-Share Scheduling Policy**.

---

## Job Dependencies

Submit jobs that depend on other jobs — Batch waits for predecessors to succeed before scheduling.

- **Sequential dependency**: `JobB` depends on `JobA` → JobB stays `PENDING` until JobA succeeds
- **N-to-N array dependency**: each index of an array job depends on the same index of another array (useful for ETL pipelines)
- **Aggregate dependency**: all child jobs of an array must complete before successor runs
- Max depth: 20 jobs in a chain

---

## Array Jobs

- Submit **a single job that spawns N parallel children** (each child has a unique `AWS_BATCH_JOB_ARRAY_INDEX`)
- Sizes: **2 to 10,000** child jobs per array
- Each child is a separate scheduled job, but submitted / managed as one
- Great for embarrassingly parallel workloads: process 10,000 files, run 10,000 simulations with different parameters
- Child jobs can have **N-to-N dependencies** on another array (parallel pipeline stages)

---

## Retries and Timeouts

- **Retry strategy**: define `attempts` (1–10) in job definition; Batch retries failed attempts automatically
- **Evaluate retry conditions**: retry on specific exit codes, container reasons (e.g., Spot interruption), or status reasons
- **Job timeout** (`attemptDurationSeconds`): max time the job runs before Batch terminates it (minimum 60 seconds)
- Common pattern: retry on Spot interruption (`reason: SpotInterruption`) but not on application errors

---

## Multi-Node Parallel Jobs (large-scale HPC)

- Single job that **spans multiple EC2 instances** for tightly-coupled workloads
- 1 **main node** + multiple **child nodes**
- Supports **Elastic Fabric Adapter (EFA)** — low-latency network for high inter-node communication (MPI workloads)
- Better performance with EC2 launch template using a **placement group ("cluster" type)**
- **Does NOT work with Spot Instances** (requires guaranteed nodes)
- Use cases: distributed training, MPI simulations, gang scheduling (incl. on EKS)

---

## Batch on EKS (May 2022)

- Run Batch jobs on Amazon EKS clusters instead of ECS / Fargate / standalone EC2
- Standardize batch workloads using Kubernetes — jobs run as **K8s Pods** in your EKS cluster
- Batch creates a separate namespace for its jobs
- Uses **CoreDNS, kube-proxy, VPC CNI** — your existing EKS networking
- **Gang scheduling** supported for multi-node parallel jobs
- Useful when you already have EKS and want consistent tooling for online and batch workloads

---

## GPU and Specialty Instance Support

- Specify GPU requirements in job definition via `resourceRequirements`
- Supports `nvidia.com/gpu` resource type on EC2 with NVIDIA GPUs (G4, G5, G6, P3, P4, P5, P5e instances)
- AWS Deep Learning AMI and DLC (Deep Learning Containers) for ML jobs
- Use placement groups + EFA for distributed training (multi-node parallel)

---

## IAM Roles for Batch Jobs

### Execution Role (ECS task execution role)
- Used by ECS to pull images from ECR, push logs to CloudWatch Logs
- Standard managed policy: `AmazonECSTaskExecutionRolePolicy`

### Job Role (task role)
- Used by the **job code itself** to call AWS APIs
- Scoped to least-privilege for what the job does (e.g., S3 read of input bucket, S3 write of output bucket)
- Different from execution role — keep them separate

### Compute Environment Role (`AWSBatchServiceRole`)
- Used by AWS Batch to manage EC2 / Fargate resources, scale ASGs, register / deregister instances

---

## Logging and Monitoring

- **CloudWatch Logs** — container stdout / stderr automatically captured to log group `/aws/batch/job`
- **EventBridge** (formerly CloudWatch Events) — emits events for every job state transition (`SUBMITTED → PENDING → RUNNABLE → ... → SUCCEEDED / FAILED`)
- Route EventBridge events to **Lambda, SNS, SQS, Step Functions, Kinesis Data Streams** for downstream automation
- **Container Insights** — collect metrics and traces from ECS containers running Batch jobs
- **AWS X-Ray** — distributed tracing if the job code is instrumented
- **Queue Snapshots** + **Share Utilization Visibility** (2026) — for analyzing queue throughput

---

## Batch vs Lambda vs ECS vs Step Functions for Batch Workloads

| Tool | Best for | Limitations |
|---|---|---|
| **AWS Batch** | Long-running batch jobs, HPC, ML training, simulations | Container-based; cold start ~minutes |
| **AWS Lambda** | Short batch tasks (<15 min), event-driven processing | 15-min limit, 10 GB ephemeral disk, 10 GB memory |
| **ECS Service** | Always-on container services | Not optimized for batch queueing / scheduling |
| **Step Functions** | Orchestrate complex workflows of mixed services | Not a compute engine; orchestrates other services |
| **EMR / EMR Serverless** | Big data (Spark, Hadoop, Hive) | Specialized for big-data frameworks |

**Use Step Functions + Batch together**: SFN orchestrates the workflow (decision logic, branching, error handling); Batch runs the heavy compute steps.

---

## Pricing

- **No charge for AWS Batch itself** — you only pay for the underlying compute (EC2, Fargate, EKS, EBS, networking) that Batch provisions
- This makes Batch cheaper than rolling your own ECS / EC2 batch system because there's no orchestration cost
- **Spot Instances** are the biggest cost lever — up to 90% discount vs On-Demand
- For predictable workloads, use **Savings Plans** on the EC2 fleet

---

## Exam Tips

- **AWS Batch is the answer** for: "managed batch processing at scale," "long-running jobs," "HPC," "many parallel containerized jobs," "ML training pipeline orchestration"
- **Pick Fargate** when you want fully managed, no instance config; **EC2** for specific instance types (GPU, large memory, EFA); **EKS** when you already use Kubernetes
- **Allocation strategies**: SPOT_CAPACITY_OPTIMIZED or SPOT_PRICE_CAPACITY_OPTIMIZED for Spot workloads (minimize interruption); BEST_FIT_PROGRESSIVE for flexible On-Demand
- **Multi-Node Parallel Jobs** = the answer for tightly-coupled HPC with EFA (MPI workloads). Does NOT work with Spot.
- **Fair-Share Scheduling** = the answer for multi-team / multi-workload sharing one queue with fair allocation over time
- **Array Jobs** = the answer for "process 10,000 files in parallel from one submission"
- **Job dependencies** = the answer for batch pipelines with sequential stages
- **EventBridge** (not CloudWatch Events) emits job state events — route to Lambda / SQS / SNS / SFN
- **You pay only for compute**, not for Batch orchestration — Batch is "free middleware"
- For **batch + workflow orchestration**, combine **Step Functions** (decision logic) + **AWS Batch** (compute)
- **Use Spot** with retry-on-interruption strategy for huge cost savings on stateless jobs
- For **GPU workloads**, specify GPU resource requirements in job definition; pick GPU instance families (G / P)
- **Logging** automatically to CloudWatch Logs; integrate with Container Insights and X-Ray

---

## Exam Traps

- **Multi-Node Parallel Jobs don't support Spot Instances** — they need guaranteed nodes; pick On-Demand
- **AWS Batch is NOT Lambda for jobs** — Lambda has a 15-min hard limit; Batch has no time limit and supports any container
- **BEST_FIT default can stall the queue** if cheapest instance is unavailable; use BEST_FIT_PROGRESSIVE or SPOT_CAPACITY_OPTIMIZED for resilience
- **Fargate has limits**: max 16 vCPU, 120 GB memory per task as of 2024; for larger jobs use EC2
- **Job definitions are versioned** — updating creates a new revision; jobs reference a specific version
- **Job role ≠ execution role** — execution role lets ECS pull images and push logs; job role lets your code call AWS APIs
- **CloudWatch Events is the old name** — it's now **EventBridge**; same service, new branding
- **Array job indexes are passed via env var** `AWS_BATCH_JOB_ARRAY_INDEX` — don't pass them as command-line args
- **You can't change a job's queue after submission** — must terminate and resubmit
- **Compute environments are region-scoped** — for multi-region batch you need separate environments per Region
- **Private subnets must have a route to ECS** (NAT GW or VPC Endpoints for ECS + ECR + Logs + S3) — common networking pitfall
- **Batch is not for real-time / synchronous request-response** — it's queue-based; use Lambda, App Runner, or ECS Service for that
- **Don't confuse Batch with Step Functions** — Batch runs the jobs; SFN orchestrates the workflow. They're complementary.
- **Job retries are not infinite** — max 10 attempts; design for idempotency
- **Spot interruption is not failure** by default — configure retry strategy to retry on `SpotInterruption` reason but not on application errors
