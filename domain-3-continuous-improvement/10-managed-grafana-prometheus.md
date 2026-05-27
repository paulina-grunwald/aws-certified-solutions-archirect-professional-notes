# Amazon Managed Grafana + Amazon Managed Service for Prometheus

> **Two managed open-source observability services. Managed Grafana = fully managed Grafana dashboards + multi-source data visualization (CloudWatch, Prometheus, Loki, X-Ray, Athena, OpenSearch, Timestream, RDS). Managed Service for Prometheus (AMP) = managed Prometheus-compatible time-series metric store for container / Kubernetes workloads. AMP + Grafana together = AWS-native equivalent of self-hosted Prometheus + Grafana stack. Common SAP-C02 scenarios: lift-and-shift existing Prometheus dashboards, hybrid/multi-cloud observability, EKS monitoring at scale.**

Maps to: **Domain 3.6 — Operational excellence**, **Domain 3.2 — Performance**, **Domain 4.3 — Modernization**

---

## Amazon Managed Grafana

### Overview

- **Fully managed Grafana service** (AWS hosts + scales + patches)
- Connects to **20+ data sources** including:
  - **AWS**: CloudWatch, Prometheus (AMP), X-Ray, OpenSearch, Athena, Timestream, RDS, Redshift, Aurora, IoT SiteWise, IoT TwinMaker
  - **External**: Self-hosted Prometheus, InfluxDB, Loki, Tempo, PostgreSQL, MySQL, Snowflake, GitHub, Datadog, New Relic, Splunk
- Build **dashboards + alerts** across sources in a single UI
- **SAML / IAM Identity Center** for auth (not Grafana-native users)
- Enterprise plugins (Datadog, ServiceNow, Snowflake) on Enterprise tier

### Workspaces

- A **workspace** = a managed Grafana environment
- Each workspace has its own URL, dashboards, data sources, alerts
- Permissions per workspace — Admin / Editor / Viewer

### Authentication

- **IAM Identity Center (preferred)** — federated SSO via Org
- **SAML 2.0** — bring your own IdP (Okta, Azure AD, OneLogin)
- **No local users** — must federate

### Data Source Access

- **Service-managed permissions** — Managed Grafana provisions IAM roles for AWS data sources
- **Cross-account access** via IAM role assumption
- For external sources (on-prem Prometheus): VPC connector + Direct Connect / VPN

### Versions

- **Grafana 9 / 10** supported
- AWS handles upgrades; you can pin a version

### Pricing

- **$9 per active Editor or Admin user / month**
- **$5 per active Viewer / month**
- **Enterprise plugins**: $45 per Editor/Admin, $25 per Viewer
- "Active" = logged in during the month

### Use Cases

- Cross-account / cross-Region observability dashboard
- Hybrid cloud (AWS + on-prem) unified view
- EKS monitoring (with AMP as data source)
- Replace self-hosted Grafana cluster ops burden

## Amazon Managed Service for Prometheus (AMP)

### Overview

- **Managed Prometheus-compatible** metric ingestion + storage + query
- AWS handles scaling, replication, durability — no Prometheus servers to operate
- Compatible with **PromQL** queries + Prometheus remote_write protocol
- Built for **EKS / ECS / EC2 / on-prem** containerized workloads
- **150 days** default metric retention

### Workspaces

- A **workspace** = a multi-tenant AMP environment for your account
- Has a unique remote_write + query endpoint
- Per-workspace IAM auth (SigV4)

### Ingestion

- Use **Prometheus agent mode** or **OpenTelemetry collector** to remote_write to AMP
- **ADOT (AWS Distro for OpenTelemetry)** is the AWS-recommended collector
- For EKS: deploy AMP-targeting ADOT collector as DaemonSet / Sidecar
- IAM auth via SigV4 — no API keys

### Querying

- PromQL via the workspace's `/api/v1/query` endpoint
- Use Managed Grafana as the visualization layer (most common)
- Or query directly from any Prometheus-compatible tool (Grafana OSS, Thanos, etc.)

### Alerting

- **Alerting rules** + **recording rules** defined per workspace (Prometheus alerting YAML)
- Alerts sent to **Alertmanager** (managed, AWS-hosted)
- Alertmanager routes to SNS → downstream (PagerDuty, Slack, email)

### Cross-Account

- Workspace can be accessed from other accounts via IAM role assumption
- Sources in account A → remote_write to AMP workspace in account B → Grafana in account C

### Pricing

- **$0.90 per 10M metric samples ingested** (first 2B samples)
- **$0.03 per GB stored / month**
- **$0.10 per query GB processed**
- Alertmanager + recording rules included

## AMP + Managed Grafana Pattern

**Standard EKS observability stack:**

1. ADOT collector in EKS sends metrics → **AMP**
2. **Managed Grafana** with AMP data source
3. Engineers query / dashboard in Grafana via PromQL
4. AMP alerting rules → Alertmanager → SNS → PagerDuty

This is the **default modern AWS observability pattern** for Kubernetes workloads.

## Comparison with CloudWatch

| Need | AMP | CloudWatch |
|---|---|---|
| Existing Prometheus dashboards / queries | **AMP** | — |
| Native AWS service metrics (EC2, RDS, Lambda) | — | **CloudWatch** |
| EKS / containerized custom metrics | **AMP** (or CloudWatch Container Insights) | CloudWatch with Container Insights |
| PromQL | **AMP** | — (CloudWatch is its own query language) |
| Cross-cloud / hybrid | **AMP** | — (AWS-only) |
| Built-in alarming on AWS service metrics | — | **CloudWatch** |
| Long-term retention without manual export | **AMP** (150d default) | CloudWatch (15 months) |

**Use both**: CloudWatch for AWS service metrics, AMP for app-level / Prometheus-instrumented metrics, Managed Grafana unifies both.

## Common Patterns

### Lift-and-shift Prometheus from on-prem
- Existing apps already export `/metrics` endpoints
- Deploy ADOT collector, point at AMP
- Existing Grafana dashboards work as-is on Managed Grafana with AMP source

### Multi-cluster EKS observability
- 5 EKS clusters in 3 Regions
- Each cluster's ADOT collector remote_writes to a **single AMP workspace**
- Managed Grafana queries the workspace → unified view

### Hybrid CloudWatch + Prometheus dashboard
- Managed Grafana dashboard with two data sources: CloudWatch + AMP
- Single panel: AWS service metrics + app metrics side by side

### Long-term Prometheus retention
- AMP retains 150 days vs. self-hosted Prometheus typically days/weeks
- No object-storage / Thanos / Cortex ops burden

## Exam Tips

- "Managed Prometheus-compatible metric store" → **AMP**
- "Managed Grafana dashboards across AWS + on-prem sources" → **Managed Grafana**
- "EKS metrics with PromQL" → **AMP + ADOT collector**
- AMP retention: **150 days** default
- Managed Grafana auth: **IAM Identity Center or SAML** — no local users
- Pricing: AMP per-ingestion + storage; Grafana per active user
- ADOT (AWS Distro for OpenTelemetry) is the recommended collector for AMP

## Exam Traps

- **AMP is NOT a general-purpose TSDB** — only Prometheus remote_write; doesn't accept StatsD, Graphite, OTLP-metrics natively (ADOT bridges OTLP → Prometheus)
- **Managed Grafana has no local user accounts** — must use IAM Identity Center or SAML
- **AMP doesn't run Prometheus scraping itself** — you run a collector (ADOT or Prometheus agent) that remote_writes
- **AMP alerts use Alertmanager, not CloudWatch alarms** — different alerting path
- **Managed Grafana Enterprise plugins cost extra** — Datadog / Splunk / ServiceNow plugins are Enterprise-tier only
- **Don't pick AMP for AWS service metrics** — CloudWatch is native + cheaper for that
- **Grafana version pinning** — AWS supports versions for limited windows; plan for upgrades
- **AMP query cost is per GB processed** — broad / expensive PromQL queries add up
- **Self-hosted Prometheus on EKS ≠ AMP** — different ops profile; the question often steers you to AMP when "managed" / "no ops burden" is mentioned
