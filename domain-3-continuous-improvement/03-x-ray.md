# AWS X-Ray (+ OpenTelemetry)

> **Distributed tracing service for serverless + containerized + EC2 apps. Records traces (request paths), segments (per service), subsegments (per operation). Service Map visualizes service-to-service dependencies + latencies. Sampling rules control trace cost. Native SDKs + agents; OpenTelemetry support via ADOT. Integrates with API Gateway, Lambda, ECS, EKS, EC2, App Mesh, SNS, SQS, DynamoDB, Step Functions. Pairs with CloudWatch ServiceLens for unified observability.**

Maps to: **Domain 3.2 — Improve performance**, **Domain 4.3 — Modernization observability**

---

## Overview

- **Distributed tracing** — follow a request across many services
- **Identify bottlenecks**, latency hotspots, errors
- **Service Map** — visual graph of services + connections + latencies
- **Trace** — end-to-end record of a single request
- **Segment** — work done by one service / component
- **Subsegment** — finer-grained units (DB calls, HTTP calls, custom)
- **Annotations** (indexed) + **Metadata** (non-indexed) for filtering

## How X-Ray Works

1. App instrumented with **X-Ray SDK** (Node.js, Python, Java, .NET, Ruby, Go) or **OpenTelemetry**
2. SDK sends segments to **X-Ray daemon** (local UDP) or directly via API
3. **X-Ray service** assembles traces from segments
4. **Service Map** + **Analytics** built from traces

## Supported Integrations

- **API Gateway** — set tracing in stage settings; traces include API GW segment
- **Lambda** — enable **Active Tracing**; X-Ray SDK for downstream span instrumentation
- **ECS / EKS / Fargate** — sidecar daemon or OpenTelemetry collector
- **EC2** — install X-Ray daemon
- **App Mesh** — Envoy sends tracing automatically
- **SNS, SQS, DynamoDB, S3, Kinesis** — service segments visible via SDK
- **Step Functions** — execution-level tracing

## Sampling Rules

- Control which requests get traced (cost)
- **Default**: 1 trace/second + 5% of requests
- **Custom rules** by service, URL pattern, HTTP method
- Configure **reservoir** (guaranteed traces) + **fixed rate** (% of remaining)

## OpenTelemetry (AWS Distro for OpenTelemetry / ADOT)

- AWS-managed OTel SDK / collector
- **Vendor-neutral** instrumentation
- Send traces to X-Ray + metrics to CloudWatch + logs to CloudWatch / S3
- **Cross-tool compatibility** — same traces to Datadog, Honeycomb, etc.

## Trace Query + Analytics

- **X-Ray Analytics** — query traces by annotation, service, error rate, latency percentile
- **Filter expressions** — `service("api-gateway") AND fault`
- **Trace summary** — duration, services touched, errors, top operations

## CloudWatch ServiceLens

- Unified view of **traces + metrics + logs**
- Combines X-Ray Service Map with CloudWatch metrics + Application Insights
- Click into a service → trace + metrics + log group entries
- One pane for observability triage

## Trace Sampling Cost Strategy

- High-volume services: sample 1% + reservoir of 5/sec
- Low-volume services: sample 100%
- Error / slow requests: **always sample** (custom rule on response status code or latency)

## Common Patterns

### API Gateway → Lambda → DynamoDB tracing

- Enable tracing on API Gateway stage
- Lambda Active Tracing ON
- DynamoDB calls auto-instrumented via SDK
- See full trace: API GW → Lambda → DynamoDB

### Microservices on ECS / EKS

- Sidecar X-Ray daemon container OR OpenTelemetry collector
- Each service publishes segments
- Service Map shows the full dependency graph

### Diagnosing latency

- Query traces with latency > 5s
- Identify slow downstream call (DB? external API?)
- Look at subsegments for the bottleneck

## Security & Compliance

- Encrypted at rest with KMS (AWS-owned or CMK)
- IAM controls access to traces
- **Tracing data is NOT for sensitive content** — annotations / metadata visible to ops
- Trace retention: **30 days**

## Pricing

- **First 100,000 traces recorded per month free**
- $5 per 1M traces recorded
- $0.50 per 1M traces retrieved or scanned

## Exam Tips

- "Distributed tracing across services" → **X-Ray**
- "Service dependency + latency visualization" → **X-Ray Service Map**
- "Trace a request through API Gateway → Lambda → DynamoDB" → **X-Ray with Active Tracing**
- **OpenTelemetry support via ADOT** for vendor-neutral instrumentation
- **Sampling rules** for cost control
- **Annotations vs Metadata** — annotations are indexed (queryable); metadata is not
- **CloudWatch ServiceLens** unifies X-Ray + CloudWatch metrics + Application Insights
- **30-day retention** for traces
- **First 100k traces/month free**

## Exam Traps

- **X-Ray is NOT for log aggregation** — that's CloudWatch Logs
- **X-Ray traces are NOT metrics** — use CloudWatch metrics for time-series KPIs
- **Lambda Active Tracing is opt-in** — must enable per function
- **X-Ray SDK ≠ CloudWatch agent** — different agents / SDKs
- **Sampling default is 1/s + 5%** — too low for low-traffic services; tune up
- **Trace retention is 30 days** — for longer retention export to S3
- **X-Ray doesn't trace HTTP requests outside instrumented apps** — must instrument the SDK or use OpenTelemetry / service-level tracing
- **OpenTelemetry via ADOT** is the modern path — pure X-Ray SDK is still supported but ADOT is more flexible
