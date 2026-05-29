# AWS Lambda

> **Serverless compute. Run code in response to events without managing servers. Pay only for what you consume. The default answer to "I want to glue services together", "I want an event-driven workflow", or "I want a small HTTP backend". Hard timeout: 15 minutes — never propose Lambda alone for longer jobs.**

Maps to: **Domain 2.5 — High-performance architectures**, **Domain 2.4 — Decoupling / event-driven**, **Domain 4.3 — Serverless modernization**

---

## Overview & Core Concepts

- Serverless compute — run code without provisioning servers
- **Runtimes**: Node.js, Python, Java, .NET, Go, Ruby, custom (Runtime API), **container images up to 10 GB**
- **Timeout**: max **15 minutes** (900s). For longer jobs use **Step Functions**, **Fargate**, **Batch**, or chain via SQS
- **Memory**: 128 MB → **10,240 MB** in 1 MB steps; CPU + network scale proportionally
- **Architecture**: `x86_64` or **`arm64`** (Graviton2) — arm64 is ~20% cheaper and often faster for most workloads
- **Lambda Managed Instances**: Lambda on EC2 compute with serverless simplicity — up to 32 GB / 16 vCPUs with configurable memory-to-vCPU ratios (2:1, 4:1, 8:1)
- **Ephemeral `/tmp`**: 512 MB → 10,240 MB
- **Deployment package**: ZIP up to 50 MB compressed / 250 MB uncompressed; container images up to 10 GB
- **Environment variables**: 4 KB total
- **Stateless** — no instance affinity; each invocation may hit a fresh execution environment
- **Billing**: per request + GB-second compute (rounded to nearest ms); INIT phase now billed across all configurations for ZIP packages with managed runtimes
- **Free tier**: 1M requests / 400k GB-s per month

---

## Invocation Models

- **Synchronous (RequestResponse)** — caller waits; used by API Gateway, ALB, CloudFront (Lambda@Edge), Function URL, direct SDK
- **Asynchronous (Event)** — Lambda queues internally and returns immediately. Built-in retry (2 retries, backoff). Used by S3, SNS, EventBridge
  - **DLQ** (SQS / SNS) — capture failed events after retries
  - **Destinations** (preferred over DLQ) — route success / failure to SQS, SNS, Lambda, or EventBridge with richer metadata
  - **Maximum Event Age**: 60s to 6h (default 6h); older events discarded
  - **Max Retry Attempts**: 0 to 2 (default 2)
- **Event Source Mapping (Poll-based)** — Lambda polls the source: SQS, Kinesis, DynamoDB Streams, Kafka (MSK / self-managed), Amazon MQ
  - Lambda manages the polling infrastructure
  - **Provisioned Mode for SQS ESM** — fine-tune throughput with min/max event pollers for sudden spikes
  - Batch window + batch size configurable per source
  - On failure: bisect-on-error (Kinesis / DynamoDB), DLQ, or Destinations

---

## Lambda Function URLs

- Built-in HTTPS endpoint — no API Gateway needed
- URL: `https://<url-id>.lambda-url.<region>.on.aws`
- Auth: **AWS_IAM** or **NONE** (no API keys, no custom authorizers)
- CORS, response streaming via `InvokeWithResponseStream`
- Bound to a function or a specific alias
- **Use for**: simple webhooks, single-function microservices, prototyping
- **Don't use for**: multi-route APIs, throttling, usage plans, request validation, WAF integration — those need API Gateway

---

## Response Streaming

- Stream payloads progressively as data becomes available
- API: `InvokeWithResponseStream`
- Max payload: **20 MB** (streaming) vs 6 MB (buffered)
- Reduces Time-to-First-Byte for large responses
- Available via Function URLs and the invoke API — NOT API Gateway REST APIs
- **Use cases**: SSR, large file generation, real-time LLM responses, progressive data delivery
- Handler: Node.js `awslambda.streamifyResponse`, or response-streaming interfaces in other runtimes

---

## Lambda SnapStart

- Reduces cold start latency from seconds to **sub-second** (up to 10× improvement)
- Mechanism: Firecracker microVM snapshot taken after INIT, restored on invocation
- **Supported runtimes**: Java 11+, Python 3.12+, .NET 8+
- **NOT supported with**:
  - Provisioned concurrency (mutually exclusive)
  - Amazon EFS
  - Ephemeral storage > 512 MB
  - arm64 architecture (x86_64 only)
- **Uniqueness gotcha**: snapshot replay duplicates random seeds and IDs set during INIT — use `afterRestore` hooks for re-initialization
- Best fit: Java / Python / .NET with heavy initialization (framework startup, DI containers, large class loading)
- **Free** (no per-request snapshot fee on supported runtimes)

---

## Concurrency & Scaling

- **Reserved Concurrency**: guaranteed slots carved from account pool, also acts as a hard max. No extra cost. Setting to **0 disables the function**.
- **Provisioned Concurrency**: pre-initialized envs — zero cold start; billed whether used or not. Pair with **Application Auto Scaling** for scheduled / target-tracking
- **Account-level unreserved concurrency**: 1,000 default (soft, raisable to tens of thousands)
- **Burst capacity**: immediate +500 to +3,000 (region-dependent), then +500/min
- Throttled invocations: `429 TooManyRequestsException` (sync), retried (async / ESM)
- **SnapStart vs Provisioned Concurrency**: SnapStart = cheap, runtime-restricted, near-zero cold start. Provisioned = all runtimes, zero cold start, costs even when idle. **Cannot combine on the same function version.**

---

## Lambda Durable Functions

- Build reliable multi-step apps + AI workflows directly in Lambda
- Automatically **checkpoint progress** and **suspend** for up to **1 year**
- Recover from failures without bespoke state management
- No extra infra to manage
- Available in 14+ Regions (US East/West, EU Ireland/Frankfurt, several APAC)
- **Use cases**: AI/ML pipelines, human-in-the-loop approvals, long-running orchestrations
- Complements Step Functions — Durable Functions are **code-first**, Step Functions are **visual / declarative**

---

## Lambda Tenant Isolation

- Run invocations in **separate execution environments per end-user / tenant**
- Simplifies multi-tenant SaaS — no shared runtime state between tenants
- Use case: SaaS platforms with strict data-isolation requirements

---

## Container Image Support

- Deploy as a container image up to **10 GB** (ECR same-account, same-region)
- Must implement the **Lambda Runtime Interface Client (RIC)** — AWS base images ship with it
- **Runtime Interface Emulator (RIE)** for local testing
- Images are immutable — version via ECR tags
- Cold start scales with image size — mitigate with SnapStart (where supported) or Provisioned Concurrency
- **Use**: ML inference with large models, heavy native deps, consistent dev/prod parity

---

## EFS Integration

- Mount EFS for **persistent, shared storage** across invocations
- Requires VPC connectivity (EFS mount targets live in subnets)
- EFS Access Point provides POSIX user/group + path entrypoint
- Throughput scales with concurrent executions; up to **25,000 concurrent connections**
- **Use cases**: shared reference data, ML models, `/tmp` overflow
- **Not compatible with SnapStart**
- Latency higher than `/tmp` (network) — use `/tmp` for scratch, EFS for shared / persistent

---

## VPC Connectivity

- Attach Lambda to a VPC to reach private resources (RDS, ElastiCache, internal services)
- Uses **Hyperplane ENIs** shared across functions in the same subnet + SG combo — no cold-start ENI penalty
- VPC-attached functions **lose direct internet** by default — add **NAT Gateway**, **VPC endpoints**, or **IPv6** for outbound
- **IPv6**: eliminates need for NAT when reaching internet or AWS services from VPC
- **Best practice**: only attach to VPC when you actually need private resources

---

## Lambda@Edge vs CloudFront Functions

| Feature | **CloudFront Functions** | **Lambda@Edge** |
|---|---|---|
| Runtime | JavaScript only | Node.js, Python |
| Execution time | < 1 ms | Up to 5s (viewer) / 30s (origin) |
| Memory | 2 MB | 128 MB (viewer) / 3,008 MB (origin) |
| Triggers | Viewer request/response only | Viewer + Origin request/response |
| Network access | No | Yes |
| File system | No | Yes |
| Package size | 10 KB | 1 MB (viewer) / 50 MB (origin) |
| Pricing | ~1/6 of Lambda@Edge | Higher |
| Scale | Millions req/sec | Thousands/sec per Region |
| Isolation | Process-based | VM-based |

- **CloudFront Functions on the same viewer event as Lambda@Edge is NOT allowed** — pick one (combining on origin events is fine)
- **CloudFront Functions**: header manipulation, URL rewrites/redirects, cache-key normalization, simple A/B, lightweight JWT validation
- **Lambda@Edge**: origin selection, complex auth, image resize, SSR, A/B with external data, SEO bots

---

## Event Source Mappings (Deep Dive)

- **SQS** — batch 1–10,000; long polling. On failure → DLQ on source queue or Destinations. Scales to **1,000 concurrent batches (standard)** / 5 (FIFO). **Provisioned Mode** for min/max event pollers
- **Kinesis / DynamoDB Streams** — batch 1–10,000; **parallelization factor 1–10**. On failure: bisect-on-error, retry-with-backoff, DLQ, Destinations. **Tumbling windows** up to 15 min for aggregation
- **Kafka (MSK / self-managed)** — SASL/SCRAM, mTLS; VPC connectivity for self-managed
- **Amazon MQ (ActiveMQ / RabbitMQ)** — VPC required; basic auth via Secrets Manager
- **Event filtering** — apply criteria at the ESM to skip non-matching events (cheaper than filtering in code)
- **Partial batch failure reporting** — return failed item identifiers so only failed records retry (SQS, Kinesis, DynamoDB)

---

## Step Functions Integration

- **Standard Workflows**: up to 1 year, exactly-once, up to 25,000 events in history, priced per state transition
- **Express Workflows**: up to 5 min, at-least-once, high-volume, priced per execution + duration
- Lambda integration patterns:
  - `arn:aws:states:::lambda:invoke` — fire-and-forget
  - `.sync` — synchronous wait
  - `.waitForTaskToken` — pause until token returned (human approval, async external system)
- **Map state** — parallel iteration: Inline 40 concurrent / Distributed up to **10,000 concurrent**
- Error handling: Catch / Retry per state; Lambda errors surface as `States.TaskFailed`
- **Choose Step Functions over Lambda chaining** when: branching logic, visual monitoring, per-step error handling, > 15 min total, human approvals
- **Durable Functions vs Step Functions**: Durable = code-first async/await; Step Functions = declarative JSON/YAML + visual designer, broader service integrations

---

## Aliases & Versions

- **Version** — immutable snapshot of code + config, auto-numbered; `$LATEST` is mutable and points to the most recent unpublished
- **Alias** — named pointer to a version; can be updated without changing callers
- **Weighted alias (traffic shifting)** — split traffic between two versions (90/10) for canary / linear deployments
- **CodeDeploy integration** — `Canary`, `Linear`, `AllAtOnce` with pre/post hooks and CloudWatch-alarm rollback
- Event source mappings target an **alias** — flip the alias to roll forward / back without touching the mapping

---

## Layers & Extensions

- **Layers** — package shared deps separately from function code
  - Up to **5 layers** per function; total unzipped (function + layers) under 250 MB
  - Versioned, immutable; shareable across functions / accounts (resource policy)
  - Extracted to `/opt/`
- **Lambda Extensions** — long-running processes alongside your function (Datadog, New Relic, Secrets cache, Parameters & Secrets Lambda Extension, observability sidecars)
  - Run in the same execution environment; share lifecycle events (init, invoke, shutdown)
  - Configure via Layers
- **Lambda Powertools (AWS-published)** — common utilities for logging, tracing, metrics, idempotency, parameters; available for Python, Node.js, Java, .NET

---

## Security & Permissions

- **Execution role** — what the function can do (DynamoDB, S3, ENI for VPC)
- **Resource-based policy** — who can invoke the function (API Gateway, S3, cross-account)
- **Environment variable encryption** — KMS (AWS-managed or CMK) at rest; helpers for in-transit
- **Code signing** — AWS Signer ensures only trusted artifacts deploy
- **Secrets Manager / Parameter Store** — fetch secrets at runtime; cache in execution environment (use Lambda Extensions for managed caching)
- **VPC isolation** when reaching private resources

---

## Monitoring & Observability

- **CloudWatch Logs** — auto log group `/aws/lambda/<function-name>`; role needs `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents`
- **CloudWatch Metrics** — `Invocations`, `Duration`, `Errors`, `Throttles`, `ConcurrentExecutions`, `UnreservedConcurrentExecutions`, `IteratorAge` (streams)
- **X-Ray** — distributed tracing, enable **Active Tracing**; X-Ray SDK for downstream spans
- **Lambda Insights** — enhanced per-invocation CPU, memory, disk, network metrics
- **Telemetry API** — extensions can subscribe to platform/function/extension telemetry streams
- **GenAI Observability** — CloudWatch surfaces token usage, model invocation metrics for Bedrock-backed Lambdas

---

## Limits Summary

| Resource | Limit |
|---|---|
| Timeout | 900 s (15 min) |
| Memory | 128 MB – 10,240 MB |
| Ephemeral `/tmp` | 512 MB – 10,240 MB |
| ZIP package (compressed / uncompressed) | 50 MB / 250 MB |
| Container image | 10 GB |
| Environment variables | 4 KB total |
| Concurrent executions (account) | 1,000 default (soft) |
| Burst concurrency | 500–3,000 (region) |
| Layers per function | 5 |
| Sync response payload | 6 MB |
| Streaming response payload | 20 MB |
| Async invocation payload | 256 KB |
| Lambda@Edge viewer | 1 MB pkg, 128 MB mem, 5 s |
| Lambda@Edge origin | 50 MB pkg, 3,008 MB mem, 30 s |

---

## Exam Tips

- **15-minute timeout** is the critical constraint — if work exceeds 15 min, the answer is **Step Functions**, **Fargate**, **Batch**, or chaining via SQS — not Lambda alone
- **Lambda + SQS + DLQ** is the textbook reliable async pattern
- **SnapStart vs Provisioned Concurrency**: SnapStart = cheap, Java/Python/.NET only, near-zero cold start. Provisioned = all runtimes, zero cold start, costs whether used or not
- **Function URLs**: "simple HTTP endpoint, no API Gateway" → Function URL. If they need **WAF, throttling, API keys, multi-route, request validation** → API Gateway
- **Response streaming** — large payloads (> 6 MB up to 20 MB) or reducing TTFB → Function URL response streaming
- **CloudFront Functions vs Lambda@Edge**: lightweight JS edge transforms → CloudFront Functions; network access / heavier compute / origin events → Lambda@Edge
- **arm64 (Graviton2)** — ~20% cheaper, often faster for most workloads; switch unless dep doesn't compile
- **VPC attachment only when needed** — Lambda → S3 / DynamoDB / public AWS services does NOT require VPC; just call the public endpoint or use a VPC endpoint
- **Event filtering on ESMs** — "process only certain events" → ESM filter criteria (cheaper than filtering in code)
- **EFS + Lambda** for shared persistent storage > 10 GB; VPC required; **not with SnapStart**
- **Weighted aliases + CodeDeploy** = canary / linear / all-at-once Lambda deployments with auto-rollback
- **Durable Functions** — code-first alternative to Step Functions for multi-step + AI workflows; can suspend for up to 1 year
- **Lambda Powertools + Extensions** — surface in answers about consistent logging/tracing/metrics or secrets caching across many functions

---

## Exam Traps

- **SnapStart + Provisioned Concurrency together** — not allowed on the same version. Any answer combining them is wrong
- **SnapStart + EFS** — not supported. If shared storage + fast cold start are both required → Provisioned Concurrency
- **SnapStart + arm64** — not supported. SnapStart is x86_64 only
- **Lambda@Edge viewer limits** — 5 s, 128 MB. Don't confuse with origin (30 s, 3 GB)
- **CloudFront Functions + Lambda@Edge on the same viewer event** — not allowed
- **VPC-attached Lambda loses internet** — needs NAT Gateway, VPC endpoints, or IPv6 to reach the public internet / public AWS endpoints
- **Sync throttling = 429 to caller; async throttling = retry + DLQ** — different behavior, exam loves the distinction
- **Reserved Concurrency = 0 disables the function** — a valid pattern to stop a function without deleting it
- **Kinesis / DynamoDB Streams stuck shard** — failed batch blocks the shard by default; configure bisect-on-error, max retry attempts, max record age, or destinations
- **Container image cold starts** scale with image size — a 10 GB image is NOT as fast as a 50 MB ZIP
- **15-min timeout is per invocation, not per workflow** — Step Functions can orchestrate hours of work, but each Lambda task in it is still capped at 15 min
- **Function URLs do not support multi-route, usage plans, API keys, request validation, or WAF** — if any of those appear in the requirements, the answer is API Gateway
- **`$LATEST` aliases**: an alias can point to a version OR `$LATEST`, but not both simultaneously
- **Lambda Layers count against the 250 MB unzipped limit** combined with the function package
