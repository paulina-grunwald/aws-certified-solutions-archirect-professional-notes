# AWS Service Catalog

> **Curated catalog of pre-approved, IT-vetted IaC products that end users can self-serve without admin privileges. Admins author CloudFormation / Terraform templates as "products", group them in "portfolios", and grant access to IAM principals / Organizations. End users launch through Service Catalog using a "launch role" — they get the resources without needing IAM permissions to the underlying services. Solves shadow IT + drift + compliance in one move. Pairs with AppRegistry for application metadata + Well-Architected Tool linkage.**

Maps to: **Domain 1.4 — Multi-account governance**, **Domain 3.6 — Operational excellence**, **Domain 4.4 — Modernization**

---

## Overview

- **Self-service catalog** of pre-approved cloud resources
- Admins define **products** (CloudFormation / Terraform templates)
- Group into **portfolios** with **constraints** (tags, IAM, launch role)
- Share portfolios with IAM users / groups / roles / Organization OUs
- End users launch products through Service Catalog UI / API — no direct IAM permission to underlying services needed
- Solves: shadow IT, drift, compliance, repeatable provisioning

## Core Concepts

| Concept | Meaning |
|---|---|
| **Product** | A CloudFormation / Terraform template defining one provisionable resource set (e.g., "Standard VPC", "Approved EC2 instance") |
| **Provisioned Product** | A running instance of a product (a deployed stack) |
| **Portfolio** | Group of products + constraints + access permissions |
| **Constraint** | Rule applied to a product within a portfolio (launch role, template, notification, stack set) |
| **Launch Role** | IAM role Service Catalog assumes when launching — user inherits permissions only for that launch |
| **TagOption** | Pre-defined tag values applied to resources at launch |

## Product Types

- **CloudFormation product** — YAML/JSON template
- **CloudFormation StackSets product** — deploy to many accounts / regions
- **Terraform Open Source / Terraform Cloud product** — Terraform configs
- Templates uploaded to S3; product references S3 URL or git

## Versioning

- Each product has multiple **versions** (template revisions)
- Users can launch any allowed version
- Admins can deprecate / hide old versions
- Update existing provisioned products to a newer version

## Constraints

### Launch Constraint
- Specifies the **IAM role** Service Catalog uses to launch
- User needs NO permissions on underlying services
- Role can have broad CloudFormation + service permissions

### Notification Constraint
- SNS topic for stack create / update / delete events

### Template Constraint
- Restrict allowed parameter values (e.g., instance type must be from approved list)

### Stack Set Constraint
- Deploy product to multiple accounts / regions automatically

### Tag Update Constraint
- Allow / block user updating tags post-launch

## Sharing Portfolios

- **Within account** — grant IAM user / role / group access
- **Across accounts** — share portfolio with another account; admin in recipient account adds principals
- **Organization-wide** — share with OU; access propagated to all accounts in the OU
- Recipients can launch but cannot modify the portfolio

## End-User Experience

1. User logs in → Service Catalog console
2. Sees portfolios they have access to
3. Selects product → fills in parameters → launches
4. Service Catalog assumes launch role → creates CloudFormation stack
5. User sees "provisioned product" status; can update / terminate

User **never gets direct access** to underlying services. Compliance enforced through the template + launch role.

## Integration with AWS Config

- Detect drift on provisioned products
- Compliance status visible in Service Catalog console
- Re-sync drift via update or terminate-and-recreate

## Integration with Organizations

- **Hub-and-spoke**: management account hosts portfolios; member accounts use them
- Portfolios shared via Organization OU → automatic for new accounts in OU
- Centralized authoring; distributed consumption

## Integration with AppRegistry

- Tag provisioned products as part of an AppRegistry application
- Link to Well-Architected Tool review for that application
- Single source of truth: app → resources → review

## Common Patterns

### Self-service VPC
- Network team authors "Standard VPC" product (3 AZs, public+private subnets, NAT, flow logs, defaults)
- Shared with every dev account
- Dev launches via UI → gets compliant VPC without IAM permission to create one
- Network team owns updates centrally

### Approved EC2 instance
- Product: EC2 with hardened AMI, mandatory tags, IAM role attached, SSM agent
- Template constraint: instance type from approved list (t3.medium, m5.large, c5.large)
- Dev launches what they need; security baseline enforced

### Multi-account database
- Stack Set product: RDS Aurora cluster deployed to dev + staging + prod
- Single provision action → 3 stacks in 3 accounts

### Marketplace + Service Catalog
- Subscribe to a Marketplace AMI / SaaS
- Wrap as Service Catalog product
- Distribute to authorized teams without sharing Marketplace billing setup

## Service Catalog vs CloudFormation / IaC

| | Service Catalog | Direct CloudFormation |
|---|---|---|
| Self-service for non-admins | **Yes** | No (need IAM) |
| Version control | Built-in | Manual (git) |
| Access via Organization OU | **Yes** | Via IAM only |
| User permissions needed | Just Service Catalog | Full CloudFormation + services |
| Best for | Curated, governed self-service | Direct IaC workflows |

Often used together: CloudFormation **authors** templates → uploaded as Service Catalog **products**.

## Service Catalog vs Proton

| | Service Catalog | AWS Proton |
|---|---|---|
| Audience | All teams (including non-engineers) | Platform engineering teams |
| Template format | CloudFormation / Terraform | CloudFormation / Terraform with environment + service templates |
| Lifecycle | Provision, update, terminate | Provision + ongoing platform updates (rolls forward) |
| Best for | Pre-approved one-off products | Continuous platform-as-a-product |

Proton is **opinionated platform engineering**; Service Catalog is **curated self-service catalog**.

## Pricing

- **Free** — Service Catalog itself
- Underlying CloudFormation / Terraform-provisioned resources billed normally
- Marketplace products billed via subscription

## Exam Tips

- "Self-service catalog of pre-approved infrastructure" → **AWS Service Catalog**
- "Users without IAM permissions launch infrastructure" → **Service Catalog launch role**
- "Distribute compliant templates across Organization" → **Portfolio shared with OU**
- "Multi-account deployment of one product" → **Service Catalog with Stack Set constraint**
- Supports **CloudFormation + Terraform OSS + Terraform Cloud**
- Free service
- Pairs with **AppRegistry + Well-Architected Tool**

## Exam Traps

- **Service Catalog ≠ AWS Marketplace** — Catalog = your internal curated products; Marketplace = third-party software you can buy
- **Service Catalog ≠ Proton** — Proton is opinionated platform tooling; Catalog is curated self-service
- **Launch role is critical** — without it, user needs underlying IAM permission (defeating the point)
- **Stack Set products are different** — they need stack-set-specific IAM + admin role
- **Portfolio sharing is one-way** — recipient can use but not modify (shared model is read-only)
- **Drift detection requires AWS Config integration** — not on by default
- **Terraform support requires extra setup** — Terraform engine in your account
- **Provisioned product termination requires explicit permissions** — users can update but not always delete
- **Cross-account sharing doesn't grant the launch role automatically** — recipient account admin must wire it
