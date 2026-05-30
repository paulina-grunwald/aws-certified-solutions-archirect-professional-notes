# Amazon QuickSight

> **Serverless, ML-powered, cloud-native BI service. Interactive dashboards, paginated reports, embedded analytics, GenAI experiences via Amazon Q in QuickSight. SPICE in-memory engine. Per-session pricing for readers. Two editions: Standard and Enterprise (Q add-on for GenAI).**

Maps to: **Domain 2.5 — Analytics / BI**, **Domain 1.2 — Fine-grained access control**

---

## Core

- **Serverless** — auto-scaling, no infrastructure
- **ML-powered**: anomalies, forecasting, auto-narratives
- Fast, embeddable; per-session pricing for readers (Enterprise)
- **SPICE** — in-memory calculation engine (Super-fast, Parallel, In-memory Calculation Engine) for imported data
- **Direct Query** mode — query data sources directly without import (huge datasets / fresh data)

---

## Data Sources

- **AWS-native**: RDS, Aurora, Athena, Redshift, S3, OpenSearch, Timestream, IoT Analytics, Lake Formation
- **Databases**: PostgreSQL, MySQL, MariaDB, SQL Server, Oracle, Teradata, Snowflake
- **SaaS**: Salesforce, ServiceNow, Adobe Analytics, Jira, GitHub
- **Files**: CSV, Excel, JSON, Parquet (direct upload or S3)
- **APIs**: REST connector

---

## Editions

| Feature | Standard | Enterprise |
|---|---|---|
| Dashboards / analyses | ✅ | ✅ |
| SPICE | ✅ | ✅ |
| Users + Groups | Users only | Users + **Groups** |
| **Row-Level Security (RLS)** | ❌ | ✅ |
| **Column-Level Security (CLS)** | ❌ | ✅ |
| **Tagged RLS** (multi-tenant embedded) | ❌ | ✅ |
| **VPC connection** | ❌ | ✅ |
| **Active Directory / SAML / IAM Identity Center SSO** | ❌ | ✅ |
| **Email reports + Paginated Reports** | Limited | ✅ |
| **Amazon Q in QuickSight (Generative BI)** | ❌ | ✅ (add-on) |
| **Embedded analytics** | Limited | ✅ |

> Users & Groups in QuickSight are **separate from IAM** — they live within QuickSight (can federate from IAM Identity Center, AD, SAML).

---

## Amazon Q in QuickSight (Generative BI)

- **Natural language Q&A** — "what was revenue last quarter by region?" in plain English
- **Auto-generated narratives** — automatically explain dashboards / charts
- **Generative BI** — describe what you want, Q builds the visual
- **Topic-based** — admins curate "topics" mapping business terms to fields
- **Stories** — AI-generated narrative analyses combining multiple visuals + text
- Powered by Bedrock under the hood
- Enterprise edition only, add-on subscription

---

## Paginated Reports

- Pixel-perfect, multi-page reports (think operational PDFs)
- Authored alongside interactive dashboards in QuickSight
- Schedule + email distribution
- Replaces standalone tools like SSRS / Crystal Reports for AWS-native shops

---

## Dashboards & Analyses

- **Analysis** — editable workspace where you build visualizations
- **Dashboard** — read-only snapshot of an analysis you publish + share
  - Preserves filtering, parameters, controls, sort
  - Viewers see underlying data unless RLS / CLS restricts it
- **Sheets** — pages within a dashboard
- **Themes** — corporate branding
- **Anomaly insights** — auto-detect outliers
- **Forecasting** — ML-based future predictions on time-series

---

## Embedded Analytics

- **Embedded dashboards** in your own application (iframe or SDK)
- Two modes:
  - **Anonymous embedding** — public-facing apps, no QuickSight user identity
  - **Identity-aware embedding** — per-user with row/column-level security based on the logged-in user (tagged RLS)
- Embed visuals, dashboards, Q search bar, or the full console
- Use cases: SaaS apps with customer-specific analytics, internal portals

---

## Security

- **Row-Level Security (RLS)** — restrict data rows visible to user / group via permissions dataset
- **Tagged RLS** — pass session tags at embed time for multi-tenant SaaS without per-tenant rules
- **Column-Level Security (CLS)** — hide specific columns from specific users / groups
- **VPC connection** — reach private data sources without internet
- **SAML 2.0 / IAM Identity Center / Active Directory** SSO (Enterprise)
- **KMS encryption** at rest for SPICE + Direct Query metadata
- **CloudTrail** logs control-plane API calls
- **AWS Lake Formation** integration — column / row / cell-level access governed centrally

---

## Pricing

- **Authors** — monthly per-author flat fee (Standard or Enterprise)
- **Readers** — **per-session** pricing in Enterprise (pay only when someone opens a dashboard)
- **SPICE** — first 10 GB free per Enterprise author; additional GB billed
- **Amazon Q in QuickSight** — add-on per-user fee
- **Paginated reports** — capacity-based pricing

---

## Comparison with Related Services

| Service | Role |
|---|---|
| **QuickSight** | AWS-native BI, dashboards, embedded analytics, GenAI Q |
| **Tableau, Power BI, Looker** | Third-party BI tools; can connect to AWS data sources |
| **Athena** | Query engine, not BI tool; pair with QuickSight for visuals |
| **Redshift Spectrum / Aurora / RDS** | Data sources for QuickSight |
| **OpenSearch Dashboards** | Visualization for OpenSearch data; not general BI |
| **Grafana (Managed)** | Operational dashboards + time-series; not analytical BI |

---

## Exam Tips

- "Serverless BI on AWS data" → **QuickSight**
- "Embed dashboards in a SaaS app, per-customer data isolation" → **QuickSight Enterprise + Embedded + Tagged RLS**
- "Natural language BI Q&A on AWS data" → **Amazon Q in QuickSight**
- "Pixel-perfect PDF reports on a schedule" → **QuickSight Paginated Reports**
- "BI on Athena results" → QuickSight + Athena; QuickSight IAM role needs `s3:GetObject` + `kms:Decrypt` on result location
- "Column- or row-level access control on dashboards" → **Enterprise edition** with **CLS / RLS**
- "Connect QuickSight to private RDS" → **VPC connection** (Enterprise)
- "Reduce dashboard load latency for large datasets" → **import into SPICE** (in-memory)
- **SPICE auto-refresh** — schedule incremental refreshes to keep dashboards fresh
- **Anomaly detection + forecasting** are built-in ML features

---

## Exam Traps

- **QuickSight users / groups ≠ IAM users / groups** — managed within QuickSight (federation from IAM Identity Center, AD, SAML works)
- **CLS and RLS require Enterprise edition** — Standard cannot do them
- **Dashboard viewers see underlying data by default** — apply RLS / CLS to restrict
- **SPICE has capacity limits** — large datasets may need Direct Query mode
- **VPC connections are Enterprise-only** — needed for private RDS / Redshift
- **Anonymous embedding ≠ unauthenticated public dashboards** — anonymous still requires session-token generation by your backend
- **Q in QuickSight requires topic definitions** — bad topics give bad answers
- **QuickSight + Athena + KMS-encrypted S3**: QuickSight IAM role needs `kms:Decrypt` on the key (common troubleshooting question)
- **Paginated reports ≠ scheduled dashboard email** — paginated = pixel-perfect / multi-page; emailed dashboards = PDF snapshots
- **Per-session pricing** is Enterprise only; Standard has fixed per-user pricing
