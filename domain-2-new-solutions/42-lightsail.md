# Amazon Lightsail

> **Simplified VPS-style AWS offering for small workloads. Fixed monthly pricing (starts $3.50/mo), pre-configured bundles (compute + storage + data transfer), one-click templates (WordPress, LAMP, Plesk, etc.). Designed for developers / small businesses who want predictable pricing without sizing EC2 + EBS + ELB + Route 53 separately. Common SAP-C02 use: distractor "wrong answer" for enterprise scenarios — Lightsail is rarely the correct answer when EC2 / containers / Lambda would be. Know when to NOT pick it.**

Maps to: **Domain 4.3 — Modernization (lower complexity)**, **Domain 2.5 — Performance/cost**

---

## Overview

- **Pre-bundled** virtual private server (compute + storage + data transfer)
- **Fixed monthly pricing** starting at $3.50/month
- Built on EC2 + EBS + Route 53 + ELB — but abstracted
- One-click application templates: WordPress, LAMP, Magento, Joomla, Drupal, Plesk, Ghost
- Managed databases (MySQL, PostgreSQL)
- Lightsail load balancer, CDN, object storage, containers

## Use Cases

- **Personal blog / WordPress** site
- **Small business website**
- **Dev / test environments** for cheap fixed cost
- **Simple production app** under predictable load
- **Lift-and-shift from shared hosting** (GoDaddy, Bluehost)

## What's Included Per Bundle

| Tier | RAM | CPU | SSD | Transfer | Monthly |
|---|---|---|---|---|---|
| $3.50 | 512 MB | 2 vCPU | 20 GB | 1 TB | $3.50 |
| $5 | 1 GB | 2 vCPU | 40 GB | 2 TB | $5 |
| $10 | 2 GB | 2 vCPU | 60 GB | 3 TB | $10 |
| $20 | 4 GB | 2 vCPU | 80 GB | 4 TB | $20 |
| $40 | 8 GB | 2 vCPU | 160 GB | 5 TB | $40 |
| ... up to $160/mo for 32 GB RAM / 8 vCPU

- **Data transfer overage** charged per GB if exceeded

## Lightsail Components

- **Instances** — VPS (compute)
- **Databases** — managed MySQL / PostgreSQL
- **Load Balancers** — simple HTTP/HTTPS LB
- **Object Storage** — S3-style buckets (Lightsail buckets)
- **CDN Distributions** — CloudFront-backed
- **Container Services** — managed container deployment (similar to App Runner)
- **Static IPs** — free with attached instance, $0.005/h when unattached
- **DNS Zones** — Route 53-like

## Networking

- Lightsail VPC is **separate** from your standard AWS VPC
- Can **peer Lightsail VPC to default AWS VPC** in same region (single-step toggle)
- Enables: connect Lightsail instance to RDS / EC2 in regular VPC
- Limitations: no Transit Gateway, no Site-to-Site VPN, no Direct Connect

## Lightsail vs EC2

| | Lightsail | EC2 |
|---|---|---|
| Pricing | Fixed monthly bundles | Per-hour + add-ons |
| Setup complexity | Minimal | Full configuration |
| Networking | Limited VPC peering | Full VPC features |
| Scaling | Vertical (upgrade bundle) | Horizontal (ASG) + Vertical |
| IAM integration | Limited | Full |
| Best for | < 5 instances, predictable | Enterprise, complex, scaling |

## Lightsail vs Elastic Beanstalk vs App Runner

| | Lightsail | Beanstalk | App Runner |
|---|---|---|---|
| Compute | VPS | EC2 (managed) | Container |
| Pricing | Fixed monthly | EC2 hourly | Request-based |
| Best for | Personal / SMB sites | Mid-size web apps | Container web apps |

## Migration Path

- Lightsail → EC2: **snapshot + export** an instance to AMI in regular EC2
- Useful when outgrowing Lightsail constraints
- One-way path; doesn't migrate back

## Limitations

- **No native IAM** — Lightsail accounts have their own permissions model
- **No Auto Scaling** (vertical only — upgrade bundle)
- **Limited region coverage** — fewer Regions than EC2
- **No CloudFormation native support** (limited Terraform support)
- **No Reserved / Savings Plans** — fixed pricing is its own pricing model

## Pricing

- **Bundle includes**: compute + SSD + free data transfer up to limit
- **Add-ons**: extra block storage, snapshots, static IP overage, CDN, LB
- **Database**: $15+/mo for managed MySQL/PG
- **Container service**: $7+/mo per service
- **Bucket storage**: $1+/mo per bucket

## Common Patterns

### WordPress site
- Lightsail $5/mo bundle, one-click WordPress
- Lightsail CDN distribution
- Custom domain via Lightsail DNS
- Total ~$8/mo for full hosting

### Staging / sandbox
- Lightsail VPS for dev environment
- Predictable budget for non-prod

### Migrate to EC2 later
- Start on Lightsail
- Snapshot → export AMI → launch EC2
- Continue with regular VPC + ASG when growth requires

## Exam Tips

- "Cheap predictable monthly pricing for a small website / app" → **Lightsail**
- "WordPress / LAMP / personal blog on AWS" → **Lightsail**
- "Beginner-friendly AWS VPS" → **Lightsail**
- Built-in **load balancer, CDN, managed DB, object storage, containers**
- Can **peer to default VPC** for hybrid Lightsail+EC2 patterns

## Exam Traps

- **Lightsail is RARELY the right answer for enterprise SAP-C02 scenarios** — it's usually a distractor
- **No Auto Scaling Group** — vertical scale only
- **Separate VPC from main AWS** — must peer to access regular VPC resources
- **No IAM integration** for AWS service access from Lightsail instance — use static credentials
- **No Savings Plans / RI** — already fixed pricing
- **Data transfer overage charged** if you exceed the bundle allowance
- **Limited region coverage** — not in all AWS Regions
- **Not for highly-available production at scale** — use EC2 + ASG + ELB instead
- **Pick Lightsail for "small dev/test/personal" hints** — pick EC2/ECS/Lambda for enterprise / scale / compliance hints
