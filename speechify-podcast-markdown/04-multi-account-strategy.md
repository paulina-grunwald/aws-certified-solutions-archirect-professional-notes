# Podcast 04 — Multi-Account Strategy

**Length target**: 15 min (~2,200 spoken words)
**Repo references**: `domain-1-organizational/05-aws-organizations.md`, `06-aws-control-tower.md`, `07-aws-ram.md`, `48-service-catalog.md`

## Topic & Scope

Domain 1 weighting is heavy on multi-account governance. This podcast covers Organizations, SCPs, Control Tower, RAM, and Service Catalog — the four pillars of AWS landing zones.

## Service Coverage Depth

**Deep**:
- AWS Organizations — OUs, SCPs, consolidated billing
- AWS Control Tower — landing zone, account factory, guardrails
- Service Control Policies — the most-tested governance mechanism
- AWS RAM — resource sharing across accounts
- Service Catalog — self-service vending

**Brief**:
- AppRegistry (mentioned in podcast 12)
- Account Factory for Terraform (AFT)

## Structured Outline

1. **Open (30s)** — "Multi-account is how AWS expects you to architect. Organizations + Control Tower + RAM + Service Catalog = the landing zone toolkit."
2. **Organizations & OUs (3 min)** — management account vs member, OU hierarchy, consolidated billing
3. **SCPs Deep Dive (3 min)** — apply at root/OU/account, deny-list vs allow-list, FullAWSAccess implicit, common deny patterns
4. **Control Tower (3 min)** — opinionated landing zone, mandatory + strongly-recommended + elective guardrails, Account Factory
5. **RAM (2.5 min)** — share VPC subnets, Transit Gateway, License Manager configs, Resolver rules
6. **Service Catalog (2 min)** — vending compliant products to dev teams via launch role
7. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- Organizations: management account creates Org; member accounts join
- Consolidated billing aggregates Org-wide; RI/SP sharing on by default
- SCPs attach to root, OU, or account
- SCPs are a CEILING — they don't grant, they restrict
- Default `FullAWSAccess` SCP attaches to everything — replace it for deny-list-only, otherwise allow-list-only enforced
- Control Tower = landing zone (Org + Identity Center + log archive + audit + guardrails baked in)
- Control Tower guardrails: mandatory (always on), strongly-recommended, elective
- Account Factory provisions new accounts with baseline applied
- RAM shares: VPC subnets, Transit Gateway, License Manager, Resolver rules, Outposts, ACM private certs, more
- RAM-shared resources show up natively in recipient accounts (not just IAM grant)
- Service Catalog launch role: user launches product without underlying IAM permission

## Must-Mention Exam Traps

- EXAM TRAP: SCPs do NOT grant permissions — only restrict
- EXAM TRAP: SCPs do NOT apply to the management account by default
- EXAM TRAP: Control Tower deploys via CloudFormation StackSets — modifying outside Control Tower causes drift
- EXAM TRAP: Removing the FullAWSAccess SCP without replacing it BLOCKS everything in that account
- EXAM TRAP: RAM ≠ resource-based policy — RAM shares natively; resource policy is per-service
- EXAM TRAP: Service Catalog ≠ AWS Marketplace — Catalog is internal curated; Marketplace is third-party
- EXAM TRAP: Service Catalog launch role is the trick — without it, user still needs underlying IAM
- EXAM TRAP: Organizations consolidates billing whether SCPs are enabled or not — "all features" mode is needed for SCPs
- EXAM TRAP: AWS Config aggregator for org-wide compliance needs explicit setup
- EXAM TRAP: Some services bypass SCPs (signed URLs, support APIs) — check docs for edge cases

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| Restrict every account in an OU from creating internet gateways | SCP |
| Auto-provision new accounts with baseline | Control Tower Account Factory |
| Share VPC subnets with other accounts | RAM |
| Vending compliant infrastructure to dev teams | Service Catalog |
| Consolidate billing across accounts | AWS Organizations consolidated billing |
| Centralize CloudTrail across all accounts | Organizations trail (multi-account) |

## Tone & Style

- Frame the whole multi-account world as a layered cake: Organizations is the plate, Control Tower the icing, SCPs the wire frame, RAM the bridges
- Repeat "SCPs do NOT grant" three times
- For Control Tower, name the four foundational OUs (Security, Sandbox, Workloads, Suspended)

## Rapid-Fire Closer

"SCP grants?" — "NO." "FullAWSAccess removed = ?" — "Everything denied." "Share subnets with other accounts?" — "RAM." "Vending compliant templates?" — "Service Catalog." "Control Tower deploys via?" — "StackSets."
