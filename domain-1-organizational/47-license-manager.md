# AWS License Manager

> **Centralized license tracking + enforcement for vendor software (Microsoft, Oracle, SAP, IBM, custom). Define license rules (vCPU / core / socket / instance / user limits) and License Manager enforces them at EC2 launch — hard-stop or soft-warn. Supports BYOL (Bring Your Own License) scenarios + license-included offerings. Integrates with Systems Manager (inventory), Service Catalog (vending), Organizations (cross-account), AWS Marketplace (delivered software), and Dedicated Hosts (Windows / Oracle host affinity). Critical for SAP-C02 compliance scenarios.**

Maps to: **Domain 1.5 — Cost optimization**, **Domain 1.4 — Multi-account governance**, **Domain 4.2 — Migration**

---

## Overview

- **Track + enforce software licenses** across AWS + on-premises
- Common vendors: **Microsoft (Windows, SQL Server), Oracle (DB, Java), SAP, IBM**, custom enterprise software
- Define **license configurations** (rules: # vCPUs, # cores, # sockets, # instances, # users)
- Enforce at **EC2 launch** — block over-commitment (hard) or warn (soft)
- Supports **Dedicated Host** assignment for vendor licensing requirements (Windows / Oracle BYOL often requires dedicated hardware)

## Core Concepts

### License Configuration
- Rule set for one license type
- Counts what matters: **vCPUs / cores / sockets / instances / users**
- Defines whether host requires **Dedicated Host** affinity
- Hard limit (block launch) or soft limit (warn + track)

### License Type Conversion
- Convert between license-included and BYOL on the same EC2 instance
- For Windows Server / SQL Server primarily
- Saves cost when you already own the vendor license

### Self-Managed Licenses
- Track your own license inventory
- Manual count + AWS-side enforcement
- Use for any vendor

### License Manager Subscriptions
- Track third-party software bought through **AWS Marketplace** as managed subscriptions
- Auto-tracked usage; no manual configuration

## Integrations

| Integration | Purpose |
|---|---|
| **EC2 Launch** | Block / warn at launch when limit exceeded |
| **CloudFormation** | License enforcement on stack create |
| **Systems Manager Inventory** | Discover already-deployed licenses on EC2 + on-prem |
| **Service Catalog** | Vending products with license config attached |
| **AWS Marketplace** | Auto-track managed subscriptions |
| **Organizations** | Cross-account license sharing |
| **Dedicated Hosts** | Windows / Oracle host-based licensing |

## Cross-Account Sharing

- **License Manager + Organizations**: share license configs across linked accounts
- Central account creates the config; member accounts use it at launch
- Enforces total org-wide limit, not per-account

## Common Vendor Patterns

### Windows Server BYOL
- **Per-core licensing** model (Microsoft)
- Often requires **Dedicated Host** for compliance
- License config: count physical cores on dedicated hosts
- AWS-provided **License Included** alternative (Windows AMI) — pay AWS per hour, no BYOL hassle

### SQL Server BYOL
- **Per-core** Enterprise / Standard
- Dedicated Host for hardware affinity
- License Mobility through Software Assurance may allow non-dedicated

### Oracle Database
- **Per-core licensing** with Oracle's core factor
- Dedicated Host required for BYOL in most Oracle license contracts
- License Manager tracks cores in use

### SAP Application
- **SAP HANA** + SAP NetWeaver — vCPU / SAPS based tracking
- Use License Manager + Compute Optimizer for sizing

## Dedicated Hosts vs Dedicated Instances

| | Dedicated Host | Dedicated Instance |
|---|---|---|
| Visible to license tracking (sockets / cores) | **Yes** | Less precise |
| BYOL (Windows / Oracle / SQL) | **Required for most** | Not always allowed |
| Affinity to specific host | **Yes** | No (just dedicated hardware) |
| Use case | BYOL compliance | Compliance / isolation |
| Cost | $/hour for the host | $/hour per instance |

License Manager uses **Dedicated Hosts** for precise license enforcement.

## Discovery

- License Manager can **discover existing software** via:
  - **Systems Manager Inventory** (EC2 + on-prem)
  - **AWS Data Exports** for usage rollup
- Useful for **migration assessment** — know what licenses you'd need to bring

## License Manager + Service Catalog

- Service Catalog product references License Manager config
- Users launch product → License Manager validates limits → block if exceeded
- Central governance: licenses enforced through the vending mechanism

## Pricing

- **Free** — License Manager itself
- Pay only for **underlying resources** (EC2, Dedicated Hosts, SSM, Marketplace subscriptions)

## Common Patterns

### Org-wide Microsoft compliance
- Central account: License Manager config for Windows Server (X cores)
- Share across Org via RAM-like sharing
- Member accounts launch — License Manager enforces total org-wide
- Audit trail in CloudTrail

### Oracle DB migration
- Existing on-prem Oracle: 64 cores
- Create License Manager config: 64 cores, Dedicated Hosts required
- Launch on AWS Dedicated Hosts → License Manager tracks
- Compliance team has hard-stop at limit

### Marketplace third-party software
- Buy Splunk / Trend Micro via Marketplace as License Manager managed subscription
- Auto-tracked; no manual config
- Stop / start managed in one place

### Hybrid discovery
- Run SSM on EC2 + on-prem servers
- License Manager discovers installed software inventory
- Identifies optimization opportunities (over-licensed software)

## Exam Tips

- "Track Microsoft / Oracle / SAP licenses across AWS + on-prem" → **AWS License Manager**
- "BYOL Windows Server on Dedicated Hosts" → **License Manager + Dedicated Hosts**
- "Block EC2 launch if license exceeded" → **License Manager hard-enforcement config**
- "Discover what software is installed" → **License Manager + SSM Inventory**
- Free service
- Integrates with Service Catalog for license-aware vending
- Cross-account sharing via Organizations

## Exam Traps

- **License Manager doesn't issue licenses** — it tracks them; you bring or buy from the vendor
- **BYOL Windows / Oracle often requires Dedicated Hosts** — not all BYOL works on shared tenancy
- **Hard enforcement blocks EC2 launch** — can be a self-inflicted outage if mis-configured; start with soft limit
- **License Manager ≠ AWS Marketplace** — Marketplace sells subscriptions; License Manager tracks them
- **License Manager ≠ Resource Access Manager (RAM)** — different services with similar "share across Org" capability
- **Self-managed licenses are manual count** — License Manager enforces but doesn't auto-discover off-AWS use
- **Switching BYOL ↔ License-Included** is a separate License Type Conversion feature — not all combinations supported
- **Dedicated Instance ≠ Dedicated Host** — only Hosts give socket/core visibility for licensing
