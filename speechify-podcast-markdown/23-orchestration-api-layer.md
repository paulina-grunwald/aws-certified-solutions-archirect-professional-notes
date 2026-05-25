# Podcast 23 — Orchestration & API Layer

**Length target**: 15 min
**Repo references**: `domain-2-new-solutions/20-step-functions.md`, `30-api-gateway.md`, `37-other-services.md` (AppSync)

## Topic & Scope

Step Functions for workflow orchestration; API Gateway + AppSync for client-facing APIs. SAP-C02 tests Standard vs Express workflows, API Gateway types (REST / HTTP / WebSocket / Private), and AppSync GraphQL.

## Service Coverage Depth

**Deep**:
- Step Functions — Standard vs Express, state types, integrations (200+ services), error handling, parallel/Map
- API Gateway — REST API vs HTTP API vs WebSocket vs Private API, stages, authorizers, throttling, caching
- AppSync — GraphQL service, real-time subscriptions, offline data sync

**Brief**:
- API Gateway custom domain + ACM
- Lambda Function URLs (mentioned)

## Structured Outline

1. **Open (30s)** — "Workflow orchestration plus client-facing APIs. Step Functions, API Gateway, AppSync — different jobs, often combined."
2. **Step Functions Standard vs Express (3.5 min)** — Standard (long-running up to 1 year, full history, exactly-once, paid per state transition) vs Express (high-volume, 5-min max, at-least-once or sync, paid per execution + duration), state types (Task, Choice, Parallel, Map, Wait, Pass, Succeed, Fail)
3. **Step Functions Integrations + Error Handling (2.5 min)** — Optimized integrations (Lambda, ECS Run Task, etc.), AWS SDK integrations (200+ services), Retry / Catch, intrinsic functions
4. **API Gateway Types (4 min)** — REST API (full features, throttling, caching, request/response transformations), HTTP API (cheaper, faster, JWT auth, no caching), WebSocket API (bidirectional), Private API (VPC-only via Interface Endpoint)
5. **API Gateway Features (2 min)** — Lambda authorizer, Cognito authorizer, IAM auth, usage plans + API keys, stages, canary deployments, CloudFront fronting
6. **AppSync (2 min)** — managed GraphQL, real-time subscriptions, resolvers (Lambda, DynamoDB, RDS, OpenSearch, HTTP), offline sync via Amplify
7. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- Step Functions Standard: exactly-once, 1-year max duration, full history, $25 per million state transitions
- Step Functions Express: at-least-once, 5-min max duration, paid per execution + duration, cheaper at scale
- Express sync mode: returns result; async mode: fire-and-forget
- State types: Task, Choice, Parallel, Map (iterate over array), Wait, Pass, Succeed, Fail
- Integrations: optimized (Lambda, ECS, SNS, SQS, DynamoDB, etc.) + AWS SDK integration to 200+ services
- Error handling: Retry (with backoff) + Catch (fallback path)
- API Gateway REST API: full features, request validation, transformation, caching, usage plans, API keys
- API Gateway HTTP API: ~70% cheaper than REST, faster, JWT auth, simpler config
- API Gateway WebSocket: bidirectional, $route$, $connect, $disconnect, $default
- API Gateway Private API: accessible only via VPC Interface Endpoint
- Authorizers: Lambda (custom), Cognito User Pool (JWT), IAM (signed requests)
- Usage plans + API keys: throttle + quota per customer
- Canary deployments: split traffic between two stage variants
- AppSync: GraphQL managed, real-time subscriptions over WebSocket, resolvers to DynamoDB/Lambda/RDS/OpenSearch/HTTP
- AppSync caching: per-resolver / per-query cache
- AppSync auth: API key, Cognito, IAM, OIDC, Lambda authorizer

## Must-Mention Exam Traps

- EXAM TRAP: Step Functions Express max 5 min — not for long-running orchestration
- EXAM TRAP: Step Functions Standard at-most $25 per million transitions — Express bills different (per execution + duration)
- EXAM TRAP: Map state vs Parallel — Map iterates over array; Parallel runs fixed branches
- EXAM TRAP: API Gateway REST has caching; HTTP API does NOT
- EXAM TRAP: API Gateway HTTP API lacks request/response transformation, request validation, usage plans
- EXAM TRAP: WebSocket API needs $connect / $disconnect / $default routes
- EXAM TRAP: API Gateway Private API requires VPC Interface Endpoint + endpoint policy
- EXAM TRAP: API Gateway timeout is 29 seconds for integration — long-running needs async + polling
- EXAM TRAP: Lambda authorizer caching reduces invocations — set TTL appropriately
- EXAM TRAP: AppSync ≠ API Gateway — AppSync is GraphQL specifically
- EXAM TRAP: AppSync subscriptions need WebSocket support on client
- EXAM TRAP: Step Functions optimized integrations (.sync) wait for completion; SDK integrations are call-and-continue
- EXAM TRAP: API Gateway has stage variables — use for per-stage Lambda alias or backend URL
- EXAM TRAP: Step Functions activities = long-poll-based external workers (legacy); prefer SDK or callbacks
- EXAM TRAP: AppSync supports OFFLINE sync via Amplify DataStore + delta tables

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| Long-running multi-step workflow with history | Step Functions Standard |
| High-volume short workflows | Step Functions Express |
| RESTful API with throttling + caching | API Gateway REST API |
| Lower-cost simple REST API | API Gateway HTTP API |
| Real-time bidirectional client | API Gateway WebSocket |
| Internal API accessible from VPC only | API Gateway Private API |
| GraphQL API with real-time subscriptions | AppSync |
| Aggregate data from multiple sources for client | AppSync resolvers |
| Step-by-step retry / catch error handling | Step Functions Retry + Catch |
| Iterate over a list with parallelism | Step Functions Map state (Distributed Mode for large arrays) |
| Authentication via JWT (Cognito) | API Gateway Cognito authorizer or HTTP API JWT |

## Tone & Style

- Standard vs Express is the most-tested workflow distinction
- For API GW types, repeat "REST = features, HTTP = cost"
- Frame AppSync as "GraphQL specialist"

## Rapid-Fire Closer

"Long-running workflow?" — "Standard." "High-volume short?" — "Express." "RESTful with caching?" — "REST API." "Cheap simple API?" — "HTTP API." "GraphQL?" — "AppSync." "WebSocket?" — "API Gateway WebSocket."
