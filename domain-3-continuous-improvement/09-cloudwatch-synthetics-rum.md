# CloudWatch Synthetics + CloudWatch RUM

> **Two CloudWatch frontend-monitoring services. Synthetics = synthetic monitoring with scheduled scripted "canaries" (Puppeteer / Selenium) that exercise endpoints from AWS regions. RUM = Real User Monitoring with a JavaScript snippet on your site collecting actual user page load, JS errors, and HTTP latency. Both feed CloudWatch metrics + alarms. Synthetics is proactive (run every N minutes); RUM is reactive (sees what real users see). Use together for end-to-end visibility.**

Maps to: **Domain 3.2 — Improve performance**, **Domain 3.6 — Operational excellence**

---

## CloudWatch Synthetics (Canaries)

### Overview

- **Synthetic monitoring** — scheduled scripts hit your endpoints from outside
- **Canaries** run on Lambda (managed) — write in Node.js / Python
- Use **Puppeteer (headless Chrome)** or **Selenium WebDriver**
- Schedule: every minute up to once per hour, or rate / cron expression
- Detect issues **before users do** — measure availability, latency, broken links, certificate expiry, broken APIs

### Canary Blueprints

Pre-built templates for common patterns:

- **Heartbeat monitor** — load a URL, screenshot, assert 200
- **API canary** — call REST API endpoints, validate response schema
- **Broken link checker** — crawl pages, report 4xx/5xx links
- **Visual monitoring** — screenshot diff vs baseline (detect UI regressions)
- **GUI workflow** — multi-step user journey (login → search → checkout)

### Output

- **CloudWatch metrics** per canary (success %, duration, HTTP status code distribution)
- **CloudWatch alarms** on failure thresholds
- **S3** — screenshots, HAR files, logs per execution
- **CloudWatch Logs** — full canary execution log
- **CloudWatch ServiceLens** integration

### Use Cases

- Validate prod is up from external perspective every minute
- Detect certificate expiry before browser warns users
- Validate critical user journeys (login, checkout) every 5 min
- Region-by-region health monitoring (run canaries in multiple Regions)
- Catch DNS / Route 53 misconfigurations

### Pricing

- **$0.0012 per canary run** + Lambda + S3 + CloudWatch standard charges

## CloudWatch RUM (Real User Monitoring)

### Overview

- **JavaScript snippet** added to your web app
- Collects **real user telemetry**:
  - Page load times, navigation timing
  - HTTP request latency (XHR / Fetch)
  - JavaScript errors + stack traces
  - User session paths
  - **Core Web Vitals** (LCP, FID, CLS, INP)
- Telemetry sent to CloudWatch as metrics + events
- Identify performance issues that **synthetic monitoring misses** (slow ISPs, mobile users, real browsers)

### App Monitor

- The CloudWatch RUM resource = "app monitor"
- Configure: domain, telemetries to collect, sample rate, identity pool for unauth uploads
- Generates JavaScript snippet to embed
- Uses **Cognito Identity Pool** for unauthenticated browser uploads

### Output

- **CloudWatch metrics** — performance percentiles, error counts, session counts
- **Dashboards** in CloudWatch RUM console — page load distribution, error grouping, user journey funnels
- **CloudWatch ServiceLens** — correlate RUM with X-Ray traces + CloudWatch metrics
- **CloudWatch Evidently** integration for A/B testing impact analysis (Evidently is being deprecated 2025)

### Use Cases

- Measure actual user-experienced latency (p50, p90, p99)
- Diagnose JS errors hitting real users
- Correlate frontend perf with backend X-Ray traces (ServiceLens)
- Track Core Web Vitals → SEO impact
- Compare prod performance across geographies / browsers / device classes

### Pricing

- **$1 per 100,000 RUM events** ingested
- Sample rate (10–100%) trades cost vs visibility

## Synthetics vs RUM — When to Use Which

| Need | Pick |
|---|---|
| "Is my site up right now?" — independent monitoring | **Synthetics** |
| "What latency are my actual users seeing?" | **RUM** |
| Detect outages before users notice | **Synthetics** |
| Diagnose performance regressions in prod | **RUM** |
| Certificate expiry warning | **Synthetics** |
| Real-user JS errors | **RUM** |
| Multi-step user journey validation | **Synthetics** (with Puppeteer) |
| Core Web Vitals | **RUM** |

**Use both** for full coverage: Synthetics catches outages immediately; RUM shows real-user impact.

## ServiceLens Integration

Both feed **CloudWatch ServiceLens**:
- RUM data → frontend perf
- Synthetics → endpoint availability
- X-Ray → backend traces
- CloudWatch metrics + logs → infrastructure

One pane for frontend → backend correlation.

## Common Patterns

### Multi-region availability monitoring
- Synthetics canary deployed in 5 Regions
- Each runs every 60s, hits the same prod endpoint
- Region with >1 min failure → alarm + Route 53 health check failover

### Pre-release validation
- Synthetics canary against staging environment
- Run on every CodeDeploy deployment via EventBridge trigger
- Auto-rollback if canary fails

### Real-user performance baseline
- RUM on all pages, sample 100%
- Dashboard: p90 page load by browser × geography
- Identify regressions per release

### Frontend → backend trace
- ServiceLens: RUM session → X-Ray trace
- Click slow page → see exact backend call that caused latency

## Exam Tips

- "Synthetic monitoring of endpoints from external" → **CloudWatch Synthetics canaries**
- "Real-user page load + JS errors" → **CloudWatch RUM**
- Canaries run on **Lambda + Puppeteer / Selenium**
- RUM uses a **JS snippet + Cognito Identity Pool**
- Both feed **CloudWatch metrics + alarms**
- **ServiceLens** combines RUM + Synthetics + X-Ray + metrics
- Use **both together** for end-to-end visibility

## Exam Traps

- **Synthetics ≠ load testing** — canaries are availability/performance probes, not load generators
- **RUM is JS only** — for mobile apps native SDKs needed (or app-side instrumentation)
- **RUM samples by default** — full collection at high traffic gets expensive
- **Synthetics canary failures don't auto-fix anything** — they alarm; you build the remediation
- **Canaries cost per run** — running every minute in 5 Regions = ~$8.6/month per canary minimum
- **RUM requires Cognito Identity Pool** — unauthenticated browsers need temporary creds
- **CloudWatch Evidently being deprecated 2025** — don't pick it for new feature flag use cases; use AWS AppConfig Feature Flags instead
- **Synthetics is a CloudWatch service, not standalone** — billed under CloudWatch
