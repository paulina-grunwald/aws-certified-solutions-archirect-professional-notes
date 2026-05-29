# AWS WAF (Web Application Firewall)

> **L7 web application firewall. Inspect HTTP/S requests and Allow / Block / Count / CAPTCHA / Challenge. Deploy on CloudFront, ALB, API Gateway (REST + HTTP), AppSync, Cognito user pools, App Runner. Managed Rule Groups (190+), Bot Control, Fraud Control (ATP, ACFP), rate-based rules. Per-Region for ALB/API GW; global for CloudFront.**

Maps to: **Domain 1.2 — Security controls**, **Domain 2.3 — New workload security**, **Domain 1.1 — Edge security**

---

## Overview

- **L7 Web Application Firewall** — inspects HTTP/S requests
- Blocks web exploits + bots before they hit your app
- Actions: **Allow, Block, Count, CAPTCHA, Challenge**
- **Not for DDoS** (use **Shield**)
- Returns **HTTP 403** on block (or custom response)

---

## Deployment Targets

- **CloudFront** — global
- **ALB** — regional
- **API Gateway** (REST + HTTP) — regional
- **AppSync** (GraphQL) — regional
- **Cognito user pools** — regional, for sign-up / sign-in protection
- **AWS App Runner** — regional
- **Verified Access**

> **NOT supported on CLB or NLB.**
> WAF is **regional for ALB / API Gateway**; **global for CloudFront**.

---

## Web ACL Components

- **Rules** — conditions + action; evaluated in priority order
- **Rule Groups** — bundles (AWS-managed, Marketplace, custom)
- **Default action** — Allow or Block when no rule matches
- **Web ACL Capacity Units (WCU)** — 1,500 default

---

## Rule Statement Types

- **IP set match** — /8, /16, /24, /32 (IPv4); /16, /32, /48, /64, /128 (IPv6)
- **Geo match** — by country
- **String / regex match** — body, headers, URI, query, cookies, JSON body
- **Size constraint**
- **SQL injection / XSS** — pattern recognition
- **Rate-based** — per source IP / forwarded IP over **5-minute window**
- **Label match** — composable rules
- **Custom request / response** — modify status / headers
- **Logical operators** — AND, OR, NOT

---

## Managed Rule Groups

- **190+** AWS-managed + Marketplace
- **Baseline**: `AWSManagedRulesCommonRuleSet`, `AdminProtectionRuleSet`
- **Use-case**: `SQLiRuleSet`, `WindowsRuleSet`, `PHPRuleSet`, `WordPressRuleSet`, `LinuxRuleSet`, `POSIXRuleSet`
- **IP Reputation**: `AmazonIpReputationList`, `AnonymousIpList` (Tor + proxies)
- **Bot Control** (paid): `BotControlRuleSet`
- **Fraud Control** (paid): ATP + ACFP

---

## Bot Control

- Identifies + categorizes bots (verified, sneaker, scraper, AI scrapers)
- **Targeted Bot Control** — paid; advanced detection + JS challenge + CAPTCHA

---

## Fraud Control (paid add-ons)

### Account Takeover Prevention (ATP)

- Protect login endpoints from credential stuffing + brute force
- Detects stolen creds, ATO IPs, automation
- Block + custom response

### Account Creation Fraud Prevention (ACFP)

- Protect sign-up flows from fake accounts / bots / mass registration
- Disposable email / VPN abuse detection

---

## CAPTCHA & Challenge

- **CAPTCHA** — user puzzle (8-hour token)
- **Challenge** — silent JS challenge (15-min token); transparent to legit users
- Use as rule actions (e.g., rate-based → CAPTCHA above threshold)

---

## Logging Destinations

- **CloudWatch Logs** (~5 MB/sec ingestion)
- **S3** (~5-min intervals)
- **Amazon Data Firehose** (Redshift, OpenSearch, Splunk, partners)
- Field redaction for sensitive headers / cookies

---

## Pricing

- **~$5/month per Web ACL**
- **~$1/month per rule**
- **~$0.60 per million requests**
- Managed rule groups + Bot Control + ATP / ACFP have **additional fees**
- CAPTCHA / Challenge — per-challenge served

---

## Common Patterns

### CloudFront + WAF + Shield Advanced

Standard edge perimeter — caching + L7 filtering + DDoS protection.

### ALB + WAF for regional apps

Protect HTTP / API apps behind ALB without CloudFront. WAF must be in the same Region.

### CloudFront → Origin via Secret Header

- WAF rule on ALB checks for a CloudFront-set secret header
- Origin only accepts requests with the header → prevents direct ALB bypass
- **Secrets Manager** rotates the secret

---

## WAF Classic vs WAFv2

- **WAF Classic is legacy** — WAFv2 is the default
- Don't propose Classic for new designs

---

## Exam Tips

- "Block malicious IPs / SQLi / XSS at L7" → **AWS WAF**
- **L7 only** — for L3/L4 use SGs / NACLs; for DDoS use **Shield**
- **WAF deploys on**: CloudFront, ALB, API Gateway, AppSync, Cognito, App Runner — **NOT CLB, NOT NLB**
- **Global for CloudFront**; **regional for ALB / API Gateway**
- **190+ Managed Rule Groups**
- **Bot Control + Fraud Control (ATP, ACFP)** are paid add-ons
- **Rate-based rules** — 5-min window per source IP
- **CAPTCHA + Challenge** for human verification
- **Allow / Block / Count / CAPTCHA / Challenge** actions
- **Custom response** for HTTP status / headers
- **Logging** to CloudWatch Logs / S3 / Firehose
- **CloudFront + WAF + Secret Header → ALB origin** (with Secrets Manager rotation)

---

## Exam Traps

- **WAF is NOT for DDoS** — use **Shield Standard** (free) + **Shield Advanced** (paid, WAF included)
- **NO CLB / NLB integration** — only CloudFront / ALB / API Gateway / AppSync / Cognito / App Runner
- **WAF Classic is legacy** — propose WAFv2
- **Rules evaluate in priority order** — first match wins
- **Default action** is configurable (Allow or Block)
- **IP set matches start at /8** — broader ranges not supported
- **Bot Control + ATP / ACFP are paid extras** — not included by default
- **Rate-based window is 5 minutes** — not configurable
- **Body inspection** is opt-in (max 64 KB on ALB / 8 KB on CloudFront)
