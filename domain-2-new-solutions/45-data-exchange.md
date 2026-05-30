# AWS Data Exchange

> **Marketplace for third-party data. Subscribe to data products from 300+ providers (financial data, weather, location, healthcare, demographics, market research, etc.) and receive the data via S3, Redshift, Lake Formation, or API. AWS handles entitlement + delivery + billing. Common SAP-C02 use: "ingest third-party data without operating custom ETL" — AppFlow is for SaaS connectors; Data Exchange is for purchased datasets.**

Maps to: **Domain 4.4 — Modernization opportunities**, **Domain 2.5 — Analytics performance**

---

## Overview

- **Marketplace** for third-party datasets
- 300+ data providers (Reuters, Foursquare, Nielsen, FactSet, Acxiom, Equifax, weather companies, etc.)
- Data delivered via:
  - **S3** (most common — files dropped in your bucket)
  - **Redshift** (data shares — query without copying)
  - **Lake Formation** (governed access)
  - **API** (REST endpoints)
- Subscribe → get continuous updates → use in your analytics
- AWS handles **entitlement, delivery, billing** — no custom integration

## Three Delivery Types

### File-based (S3)
- Provider publishes new datasets as files
- AWS copies to your S3 bucket on schedule
- You read via Athena, Glue, EMR, SageMaker, anything

### Redshift data shares
- Provider exposes Redshift data
- You query directly in your Redshift cluster via data share
- No data copy — live query

### API-based
- Provider hosts REST endpoint
- Data Exchange manages auth + entitlement
- Your apps call API; AWS handles provider relationship

## Subscription Model

- **Free** datasets and **paid** subscriptions
- Pricing varies: monthly subscription, per-request, free
- **Bring Your Own Subscription (BYOS)** — pre-existing contracts with provider mapped into Data Exchange
- **Trial** datasets for evaluation

## Use Cases

- Financial services: market data, alt data, ratings
- Insurance: weather, demographics, risk data
- Retail: location intelligence, foot traffic, demographics
- Healthcare / life sciences: clinical data, drug data
- Public sector: census, geospatial
- ML training: pre-trained datasets

## Data Exchange Components

### Data Sets
- A logical container for revisions

### Revisions
- New versions of the dataset
- Subscribers automatically get new revisions

### Assets
- Individual files / API endpoints inside a revision

### Jobs
- Async operations: import to S3, copy to provider, etc.

## Producer Side

Companies can also **publish** their data on Data Exchange to monetize:
- Define dataset + pricing
- AWS handles billing + entitlement
- Get paid via AWS Marketplace mechanism

## Comparison with AppFlow + Glue

| Tool | When |
|---|---|
| **Data Exchange** | Subscribe to third-party purchased data |
| **AppFlow** | Connect to SaaS apps (Salesforce, Slack, etc.) |
| **Glue** | ETL between AWS sources |
| **DMS** | Database migration / CDC |

These are **complementary**, not competing.

## Integration Patterns

### Data Exchange → S3 → Athena
- Subscribe to weather data
- Lands in your S3 bucket daily
- Glue crawler catalogs it
- Athena queries combine weather + your data
- Insurance underwriting model

### Data Exchange → Redshift data share
- Subscribe to FactSet financial data
- Add as Redshift data share
- Join with internal trading data
- No data copy / ETL

### Data Exchange → SageMaker
- Subscribe to pre-trained NLP datasets
- Mount in SageMaker training job
- Combine with your data for fine-tuning

## Pricing

- **Subscriptions** vary per provider — some free, some thousands per month
- AWS adds Marketplace handling fee
- **AWS Data Exchange itself is free** (no fees beyond subscription)

## Common Patterns

### Replace custom data licensing pipeline
- Old: contract with provider, custom FTP/API ingestion, ETL job
- New: Data Exchange subscription, S3 delivery, Athena queries
- Removes ops burden of data licensing integration

### Free public datasets
- Open Data on AWS via Data Exchange (NOAA weather, US Census, etc.)
- No subscription cost, just standard AWS storage / query

### Provider monetization
- Your company publishes proprietary data
- AWS handles entitlement + billing
- Sell to enterprise customers without building marketplace

## Exam Tips

- "Subscribe to third-party data (financial, weather, demographics)" → **AWS Data Exchange**
- "No-code ingestion of purchased datasets" → **Data Exchange**
- Delivery options: **S3, Redshift data share, Lake Formation, API**
- AWS handles **entitlement + delivery + billing**
- 300+ data providers
- Free public datasets (Open Data on AWS) available

## Exam Traps

- **Data Exchange ≠ AppFlow** — Data Exchange is purchased datasets; AppFlow is SaaS app connectors
- **Data Exchange ≠ S3 Glacier** — Data Exchange is sourcing data; Glacier is archival storage
- **Data Exchange ≠ Glue Data Catalog** — Catalog organizes your data; Data Exchange brings in third-party data
- **Pricing varies wildly** — some free, some thousands/month — check before assuming cost
- **Not all data is free** — Open Data subset is, but most provider products are paid
- **Revisions update automatically** — your pipeline must handle data versioning
- **Provider may end product** — plan for fallback
- **Data Exchange itself is free** — only the subscription costs money
