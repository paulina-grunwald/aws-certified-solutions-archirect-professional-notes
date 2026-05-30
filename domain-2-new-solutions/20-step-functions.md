# AWS Step Functions

> **Serverless visual workflow orchestrator. Build state machines that coordinate Lambda, ECS, Batch, Glue, SageMaker, SQS, SNS, DynamoDB, and 220+ AWS services. Standard (up to 1 year, exactly-once) or Express (up to 5 min, at-least-once, high-throughput). Distributed Map for fan-out (10,000 parallel). HTTP Tasks for any HTTPS endpoint. Variables + JSONata. Wait-for-callback for human approval.**

Maps to: **Domain 2.4 — Decoupling / orchestration**, **Domain 4.3 — Modernization / serverless**

---

## Core Concepts

- **State machine** — JSON document (Amazon States Language, ASL) describing a workflow as states + transitions
- **Execution** — single run of a state machine with input → output
- States: **Task, Choice, Wait, Pass, Parallel, Map, Succeed, Fail**
- **Workflow Studio** — visual drag-and-drop designer; generates / edits ASL
- **Max execution time**: 1 year (Standard) / 5 minutes (Express)
- **Built-in error handling**: `Retry`, `Catch` blocks per state
- Every state transition logged for observability

---

## Workflow Types

| Feature | **Standard** | **Express** |
|---|---|---|
| Max duration | 1 year | 5 min |
| Execution semantics | Exactly-once | At-least-once |
| Execution history | Up to **25,000 events** or 1 GiB | Sent to CloudWatch Logs |
| Pricing model | Per state transition | Per execution + duration |
| Use cases | ETL, ML pipelines, long-running orchestration, human approval | Microservice orchestration, IoT, event-driven workloads at scale |

### Express Sub-types

- **Synchronous Express** — caller waits for result (API Gateway → Step Functions → response)
- **Asynchronous Express** — fire-and-forget (high-throughput event processing)

---

## Task Types (how a Task State does work)

- **Lambda Tasks** — invoke a Lambda function
- **Service Tasks** — direct integration with AWS services
  - **Optimized integrations**: Lambda, Batch, ECS/Fargate, DynamoDB, SNS, SQS, EMR, Glue, SageMaker, Step Functions (nested)
  - **AWS SDK Integrations** — call 220+ AWS services via standard SDK API calls
- **HTTP Tasks** — call ANY HTTPS endpoint with EventBridge API Destinations + AWS Secrets Manager for auth
- **Activity Tasks** — long-poll worker model; workers run anywhere (EC2, ECS, on-prem, mobile); not serverless. Worker uses `GetActivityTask` / `SendTaskSuccess` / `SendTaskFailure`

### Wait-for-Callback Pattern

- `.waitForTaskToken` — task sends a token to a worker / SQS / SNS / Lambda; workflow pauses until the worker calls `SendTaskSuccess` with the token
- Use cases: **human approval**, async external integrations, message-driven steps
- Token can wait up to 1 year (Standard)

---

## State Types

| State | Purpose |
|---|---|
| **Pass** | No-op; useful for input shaping / debugging |
| **Task** | Single unit of work (Lambda, service task, HTTP task, activity) |
| **Choice** | Branching logic (if/else, switch) |
| **Wait** | Delay for a duration or until a timestamp |
| **Parallel** | Run multiple branches concurrently; waits for all to finish |
| **Map** | Iterate over an array (see modes below) |
| **Succeed** | Successful end state |
| **Fail** | Failure end state |

### Map State Modes

- **Inline Map** (default) — up to **40 concurrent iterations**; runs within the parent execution context
- **Distributed Map** — up to **10,000 concurrent iterations**; reads input from S3 (CSV / JSON Lines / S3 inventory) at scale; results aggregated to S3
- Distributed Map: process millions of S3 objects, batch jobs, large CSV files

---

## Triggers

- Console / SDK / CLI (`StartExecution`)
- Lambda (`StartExecution`)
- API Gateway (sync via Express; async via Standard)
- EventBridge (scheduled or event-pattern)
- CodePipeline
- Step Functions (nested)

---

## Variables & JSONata

- **Variables** — assign values to a workflow-scoped variable; reference later in any state without passing through input/output paths
- **JSONata** — replace JSONPath (`$.foo.bar`) with **JSONata** expressions (richer than JSONPath; supports transformations, math, string ops)
- Simplifies workflows that previously needed Lambda functions for trivial data manipulation

---

## Error Handling

- **Retry** — per state: error names, interval, max attempts, backoff rate
- **Catch** — per state: route specific errors to a fallback state
- Errors are strings (`States.TaskFailed`, `States.Timeout`, `Lambda.ServiceException`, custom)
- **EventBridge integration** — emit on execution failure for alerting / pages

---

## Cross-Region / Cross-Account

- State machines are **regional**; for multi-Region orchestration, use HTTP Tasks or nested executions via SDK integrations
- **Cross-account invoke** supported via assumed roles

---

## Use Cases

- **Microservice orchestration** (Express + API Gateway)
- **ETL pipelines** (Standard + Glue / Batch / Lambda)
- **ML pipelines** (Standard + SageMaker training/transform/endpoints)
- **Human approval workflows** (Standard + wait-for-callback)
- **Long-running batch jobs** (Standard + Batch / Fargate)
- **Event-driven processing at scale** (Express + EventBridge)
- **Large fan-out** processing S3 inventory / database extracts (Distributed Map)
- **Saga pattern** for distributed transactions across services

---

## Integrations Cheatsheet

| Integration | Use case |
|---|---|
| Lambda | Custom logic |
| Batch | Long-running batch compute |
| ECS / Fargate | Containerized tasks |
| Glue / EMR | ETL + Spark |
| SageMaker | ML training, transform, endpoint deployment |
| DynamoDB | CRUD operations |
| SNS / SQS | Pub-sub / queue messaging |
| EventBridge | Schedule + event-pattern triggers |
| API Gateway | Sync HTTP trigger (Express) |
| HTTP Tasks | Any HTTPS API |
| Step Functions (nested) | Workflow composition |

---

## Decision: Step Functions vs Alternatives

| Need | Service |
|---|---|
| Visual orchestration of multi-step workflows | **Step Functions** |
| Code-first multi-step Lambda with checkpointing | **Lambda Durable Functions** — alternative for code-only teams |
| Simple Lambda chain | Direct Lambda → Lambda; SQS; EventBridge |
| Workflow scheduling + DAG with retries | **MWAA (Managed Airflow)** for batch DAGs |
| Stream processing | **Managed Flink** or Lambda + Kinesis ESM |
| Heavy compute jobs | **AWS Batch** (Step Functions can orchestrate it) |

---

## Exam Tips

- "Visual workflow orchestrator for serverless" → **Step Functions**
- "Long-running orchestration (> 15 min, up to 1 year)" → **Standard Workflow**
- "High-volume, short orchestration (< 5 min)" → **Express Workflow**
- "Synchronous API → workflow → response" → **Express Sync** (low latency)
- "Pause workflow until external system / human responds" → **wait-for-callback (`.waitForTaskToken`)**
- "Fan-out processing of 1M+ items" → **Distributed Map** (S3 input, 10,000 concurrency)
- "Branching logic without writing a Lambda" → **Choice state**
- "Call any HTTPS API from workflow" → **HTTP Tasks**
- "Coordinate microservices with retries / error handling per step" → **Step Functions** > Lambda chains
- "Orchestrate Batch / Fargate / SageMaker job" → **Step Functions Service Task** with `.sync` for waiting
- "Human approval step" → **wait-for-callback** + SNS / email / web link
- "Simplify data transformations between states" → **Variables + JSONata**

---

## Exam Traps

- **Step Functions does NOT integrate natively with Amazon Mechanical Turk** — for human workflows with MTurk use **SWF** (Amazon Simple Workflow Service, legacy)
- **Standard execution history is capped at 25,000 events** — for workflows with thousands of iterations, use **Distributed Map** to split into child executions
- **Express workflows do NOT have execution history persisted in Step Functions** — must enable CloudWatch Logs to debug
- **Standard is exactly-once; Express is at-least-once** — design for idempotency on Express
- **Activity tasks are NOT serverless** — workers run on EC2 / ECS / on-prem; uncommon for new designs
- **Distributed Map results are aggregated to S3** — not returned to the parent execution directly
- **HTTP Tasks require EventBridge API Destinations** to manage credentials — not standalone
- **Cross-Region orchestration is NOT native** — use HTTP Tasks or nested executions across Regions
- **`.sync` task pattern works only with specific integrations** (Batch, ECS, EMR, SageMaker, Glue, Step Functions) — not all SDK integrations
- **Step Functions adds latency** when chaining Lambdas — for tight loops use direct Lambda invocation
- **State machine size limit**: 1 MB ASL document — for large workflows decompose into nested state machines
