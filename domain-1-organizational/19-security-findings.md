# Security Findings: GuardDuty, Inspector, Detective, Security Hub

> **Four services that work together: GuardDuty detects threats, Inspector finds vulnerabilities, Detective investigates incidents, Security Hub aggregates and prioritizes everything. Know which one answers each kind of question on the exam.**

Maps to: **Domain 1.2 — Prescribe security controls** (threat detection, vulnerability management, posture management)

---

## The Four Services at a Glance

| Service | What it does | Input | Output |
|---|---|---|---|
| **GuardDuty** | **Threat detection** — finds malicious / unauthorized activity | CloudTrail, VPC Flow Logs, DNS logs, EKS audit logs, S3 data events, RDS login, EBS volumes, Lambda network logs | Findings (severity 0.1-8+) |
| **Inspector** | **Vulnerability management** — finds CVEs, network exposure | EC2 instances, ECR container images, Lambda function code & deps | Findings with CVE IDs + risk scores |
| **Detective** | **Investigation** — root cause analysis on graphs | GuardDuty findings + CloudTrail + VPC Flow Logs + EKS audit + Route 53 query + Sign-In events | Visual behavior graph, 365-day timeline |
| **Security Hub** | **Posture & aggregation** — control checks + central findings | AWS Config + GuardDuty + Inspector + Macie + IAM Access Analyzer + Firewall Manager + Health + 3rd-party | Compliance scores, prioritized findings, automation hooks |

**Mental model**: Detection (GuardDuty / Inspector) → Aggregation + scoring (Security Hub) → Investigation (Detective) → Remediation (Automation runbooks / EventBridge / Lambda).

---

## Amazon GuardDuty

### What it is
**Threat detection service** that uses **ML** to continuously monitor for malicious behavior across AWS accounts.

- 30-day free trial
- Detects:
  - Unusual API calls, calls from known malicious IPs
  - Attempts to disable CloudTrail
  - Unauthorized deployments, compromised instances
  - Port scanning, failed logins
  - Reconnaissance behavior
  - **CryptoCurrency attacks** — dedicated finding type

### Data sources (continuously analyzed)
- **CloudTrail Events** — management events + S3 data events
- **VPC Flow Logs** — unusual internal traffic, unusual IPs
- **DNS logs** — exfiltration via DNS queries (**default VPC DNS resolver only**)
- **Optional features**: EKS audit logs, RDS & Aurora login activity, EBS volumes (malware scans), Lambda network logs, S3 data events, **Runtime Monitoring** (EC2 / ECS / EKS / Fargate)

### Finding format
- Severity 0.1 to 8+ (High / Medium / Low)
- Naming: `ThreatPurpose:ResourceTypeAffected/ThreatFamilyName.DetectionMechanism!Artifact`
- Example: `CryptoCurrency:EC2/BitcoinTool.B!DNS`, `UnauthorizedAccess:EC2/SSHBruteForce`

### Lists
- **Trusted IP lists** — IPs/CIDRs you trust (no findings generated). Public IPs only
- **Threat IP lists** — known malicious IPs/CIDRs (always generate findings); can be 3rd-party or custom
- In a multi-account setup, **only the GuardDuty administrator account can manage these lists**

### Suppression rules
- Automatically filter and archive new findings
- For low-value or false-positive findings
- **Suppressed findings are NOT forwarded to Security Hub, S3, Detective, or EventBridge** — but they can still be viewed in the Archive

### Multi-account
- Associate member accounts via AWS Organizations or invitation
- **Delegated administrator** is recommended (use the dedicated security tooling account, not management)
- Administrator manages findings, suppression rules, trusted/threat lists across all accounts

### Automation
- Findings → EventBridge → Lambda / SNS / SSM Automation → remediate
- Suspending: stops monitoring; findings preserved
- Disabling: stops monitoring; **findings and config are deleted**, irreversible

---

## Amazon Inspector

### What it is
**Vulnerability management** — automated security assessment for **EC2, ECR container images, Lambda functions**.

- Continuous scanning when needed (not periodic)
- Findings include **CVE IDs** with risk scores

### What it scans
| Target | Trigger | Checks |
|---|---|---|
| **EC2 instances** | SSM Agent installed + Inspector enabled | OS package vulnerabilities (CVE) + network reachability |
| **ECR container images** | On image push (and re-scans periodically) | Container image package CVEs |
| **Lambda functions** | On deployment | Function code + package dependency CVEs (standard and code scanning) |

### Network reachability (EC2)
- Analyzes paths from internet / VPC peering to instances
- Flags inadvertent public exposure (SG / NACL / RT misconfigs)

### CVE database
- Inspector uses **CVE (Common Vulnerabilities and Exposures)** as the source of truth
- Each finding has a **risk score** for prioritization

### Integrations
- **Security Hub** — sends findings (in ASFF for CSPM / OCSF for new Security Hub)
- **EventBridge** — automate response
- **Systems Manager** — required for EC2 scanning (SSM Agent)

### Pricing
- **Free 15-day trial**; billed after that based on resource counts and scanning frequency

---

## Amazon Detective

### What it is
**Investigation / forensics** — root cause analysis using ML + graph analytics.

- GuardDuty / Inspector / Security Hub tell you *that* something happened — Detective tells you *what really happened*, *who/what is involved*, and *the timeline*
- Continuously ingests **365 days** of historical data
- **Regional** service
- **Paid** (per GB ingested per account per Region) — NOT free tier

### Data sources
- VPC Flow Logs
- CloudTrail (management + data events)
- GuardDuty findings
- EKS audit logs
- Route 53 query logs
- AWS Sign-In events

### What it does
- Builds a **behavior graph** linking resources, identities, IPs, and findings
- Investigators **start from a GuardDuty / Security Hub finding** and **pivot** through the graph
- Click an IAM role / EC2 instance / IP / finding to see related activity and timeline
- Surfaces anomalies vs. learned baselines

### Multi-account
- AWS Organizations integration + **delegated administrator**
- Best practice: same security tooling account as GuardDuty / Security Hub / Macie

---

## AWS Security Hub

> **Rebrand**: what you knew as "Security Hub" is now **AWS Security Hub CSPM** (Cloud Security Posture Management) — still aggregates findings, runs control checks, uses ASFF (v1). AWS launched a new **"Security Hub"** on top that **correlates and prioritizes findings** from GuardDuty / Inspector / Macie / Security Hub CSPM in **OCSF** (Open Cybersecurity Schema Framework) format, with risk scoring, exposure analysis, and a Summary dashboard with up to 1 year of trend data. **v1 (CSPM) APIs and v2 APIs coexist**.

### What it is
**Central hub for security alerts** — aggregates findings from GuardDuty, Inspector, Macie, IAM Access Analyzer, Config, Firewall Manager, Systems Manager, Health, and AWS Partner / 3rd-party tools.

### Prerequisite
- **Must enable AWS Config first** — Security Hub uses Config to perform its security checks. If Config is off, controls return `NO_DATA`. Security Hub does NOT manage Config for you.

### Standards
- **AWS Foundational Security Best Practices (FSBP)** — AWS-authored, broadest coverage, default-on
- **CIS AWS Foundations Benchmark** — v1.2.0, v1.4.0, v3.0.0, and **v5.0**
- **PCI DSS v3.2.1** and **PCI DSS v4.0**
- **NIST SP 800-53 Rev. 5**
- **NIST Cybersecurity Framework (CSF) v1.1**
- **AWS Resource Tagging Standard**
- **Service-Managed Standard: AWS Control Tower** (Control Tower-managed accounts only)
- Enable/disable per standard and per control

### Main features
- **Cross-Region aggregation** — pick a **home/aggregation Region** to consolidate findings, insights, and security scores
- **AWS Organizations integration** — auto-detect new member accounts; Security Hub administrator
- **Automation Rules** — update finding fields automatically (severity, workflow status, suppress, custom fields) **without EventBridge**. Must be created in the **delegated administrator** account in the home Region + unlinked Regions; member accounts can't create their own
- **Central Configuration** — define Security Hub policies (standards, controls, enablement) in the delegated admin and attach to root / OUs / accounts; new accounts auto-inherit
- **Consolidated Control Findings** — single finding per control across multiple standards (less noise)
- **Delegated Administrator** — designate any member account (not management) for org-wide Security Hub management

### Findings format
- **CSPM (v1)**: AWS Security Finding Format (**ASFF**)
- **New Security Hub (v2)**: **OCSF** (Open Cybersecurity Schema Framework) for normalized cross-tool correlation
- Auto-update / auto-delete; **findings older than 90 days deleted** (CSPM)
- GuardDuty findings flow into Security Hub typically within **5 minutes**
- **Archiving in GuardDuty does NOT update the finding in Security Hub** — you must update workflow status separately or via Automation Rule

### Insights
- Collection of related findings highlighting a security area
- Built-in managed insights + custom insights (Group By + filters)

### Custom Actions
- Send selected findings to EventBridge with `Security Hub Findings — Custom Action` event type
- Use for invoking Lambda / SSM Automation / SNS / Step Functions / ticketing

---

## Decision Matrix

| Question / Scenario | Answer |
|---|---|
| Detect anomalous API calls, brute force, unusual traffic | **GuardDuty** |
| Detect OS / library CVEs in EC2, ECR, Lambda | **Inspector** |
| Find sensitive data (PII, credit cards) in S3 | **Macie** (separate service) |
| Detect external access to resources | **IAM Access Analyzer** (separate) |
| Aggregate and prioritize all findings, compliance posture | **Security Hub** |
| Investigate root cause / blast radius of an incident | **Detective** |
| Long-term audit log of every API call | **CloudTrail** (separate) |
| Visual SQL-queryable event store with retention up to 10 years | **CloudTrail Lake** (separate) |
| Generate compliance evidence packages for SOC/PCI/HIPAA reports | **Audit Manager** (separate) |

---

## Multi-Account Pattern (applies to all four)

1. Set up **AWS Organizations** with all features
2. Use a dedicated **Security Tooling** account (Control Tower default)
3. Designate it as the **delegated administrator** for **GuardDuty, Security Hub, Inspector, Detective, Macie, IAM Access Analyzer** (and Firewall Manager)
4. Best practice: **NOT** the Organizations management account
5. Findings consolidate in the security tooling account
6. Automation Rules in Security Hub triage incoming findings
7. Custom Actions / EventBridge route critical findings to SSM Automation, Lambda, or ticketing

---

## Exam Tips

- **GuardDuty = detection**, **Inspector = vulnerabilities**, **Detective = investigation**, **Security Hub = aggregation + posture**
- **GuardDuty data sources to memorize**: CloudTrail, VPC Flow Logs, DNS logs, EKS audit, S3 data events, RDS login, EBS volumes, Lambda network
- **GuardDuty DNS findings only work with the default VPC DNS resolver** — heavy distractor
- **Inspector covers EC2 (via SSM Agent), ECR images, Lambda functions** — NOT databases or networking
- **Detective starts from a GuardDuty/Security Hub finding** — investigation tool, not detection
- **Security Hub requires AWS Config** to be enabled first
- **Cross-Region aggregation** in Security Hub — pick a home Region; Automation Rules must be in the home Region + unlinked Regions
- **Multi-account** pattern: delegated admin in security tooling account + Organizations integration
- **CSPM (ASFF v1) vs new Security Hub (OCSF v2)** — both coexist
- **Automation Rules** in Security Hub = EventBridge-free finding remediation logic
- **Suppression rules** in GuardDuty stop findings from going downstream (Security Hub / Detective / EventBridge)
- **CIS v5.0**, **FSBP**, **PCI DSS v4.0**, **NIST 800-53 r5** are the modern standard set
- For "detect cryptocurrency mining on EC2" → GuardDuty (dedicated finding type)
- For "find CVE in container image before deploy" → Inspector on ECR
- For "compliance dashboard across many accounts and Regions" → Security Hub Central Configuration + AWS Organizations + delegated admin
- For "investigate which IAM principals an attacker touched after compromise" → Detective behavior graph

---

## Exam Traps

- **GuardDuty ≠ Inspector**. GuardDuty = behavioral threat detection (logs + ML); Inspector = static vulnerability scan (CVEs)
- **Security Hub ≠ Detective**. Security Hub = aggregator + posture; Detective = investigation graph
- **Security Hub does NOT enable AWS Config for you** — Config must be on first; otherwise controls return NO_DATA
- **Archiving a GuardDuty finding does NOT update Security Hub** — workflow status must be set separately or via Automation Rule
- **Automation Rules cannot be created from member accounts** — only from the delegated admin in the home Region (and unlinked Regions)
- **Findings older than 90 days are deleted from Security Hub CSPM** — for long-term audit use S3 export or CloudTrail / Audit Manager
- **Inspector does NOT scan databases, S3 content, or networking** — only EC2, ECR images, Lambda functions
- **GuardDuty DNS findings require the default VPC DNS resolver** — custom resolvers (BIND on EC2, Route 53 Resolver) won't generate DNS findings
- **Detective is paid** — not free tier; can be expensive at scale
- **Detective is Regional** — enable per Region; not transparent across Regions
- **Suspending vs disabling GuardDuty**: suspending preserves findings; disabling deletes them irrevocably
- **Delegated admin must NOT be the Organizations management account** — best practice is a dedicated security tooling account
- **Security Hub Central Configuration requires AWS Organizations** — if accounts aren't in an Org you must enable per-account / per-Region (script via StackSets or invitation)
- **The new "Security Hub" (v2, OCSF)** is separate from **Security Hub CSPM** — both coexist; CSPM still runs the control checks
- **Trusted IP lists in GuardDuty only work for public IPs** — they don't apply to private/internal addresses
