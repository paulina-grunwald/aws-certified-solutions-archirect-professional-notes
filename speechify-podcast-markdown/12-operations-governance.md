# Podcast 12 — Operations, Governance & Compliance

**Length target**: 15 min
**Repo references**: `domain-1-organizational/20-aws-config.md`, `21-cloudtrail.md`, `25-aws-systems-manager.md`, `47-license-manager.md`, `49-service-quotas.md`, `domain-3-continuous-improvement/06-well-architected-tool.md`

## Topic & Scope

The "what's running, who changed it, is it compliant" services: Config, CloudTrail, Systems Manager, License Manager, Service Quotas, Well-Architected Tool. Heavy SAP-C02 material.

## Service Coverage Depth

**Deep**:
- AWS Config — resource inventory + compliance rules + remediation
- CloudTrail — API audit logs, organization trails
- Systems Manager — Session Manager, Patch Manager, Run Command, Automation, OpsCenter
- Well-Architected Tool — six pillars + lenses

**Brief**:
- License Manager (BYOL for Windows/Oracle)
- Service Quotas (limit management)

## Structured Outline

1. **Open (30s)** — "These services answer: what's running, who changed it, is it compliant, can we fix it. Operations 101 for SAP-C02."
2. **AWS Config (3 min)** — configuration history, rules (managed + custom Lambda), conformance packs, aggregator across Org, remediation actions
3. **CloudTrail (3 min)** — management events (default) vs data events (opt-in), Organization trail, integrity validation, CloudWatch Logs integration
4. **Systems Manager Deep (4 min)** — Session Manager (no SSH/bastion), Patch Manager, Run Command, Automation, OpsCenter, Inventory, Maintenance Windows, Hybrid Activations for on-prem
5. **Well-Architected Tool (2 min)** — six pillars (Op Ex / Security / Reliability / Perf / Cost / Sustainability), lenses, HRIs, milestones
6. **License Manager + Service Quotas (1.5 min)** — License Manager tracks BYOL Windows/Oracle/SQL; Service Quotas manages limits + auto-request templates
7. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- AWS Config records configuration changes — history + relationships per resource
- Config rules: managed (AWS-provided) or custom (Lambda function), check compliance against desired state
- Conformance packs: bundles of rules for HIPAA / PCI / NIST
- Aggregator: cross-account + cross-Region compliance view
- Remediation: SSM Automation documents triggered on non-compliance
- CloudTrail records API calls — management events on by default, data events (S3 object, Lambda invoke) opt-in
- Organization trail: management-account-defined, applied to all member accounts
- CloudTrail logs can integrate with CloudWatch Logs for real-time alerts + S3 for retention
- Log file integrity validation: optional SHA-256 hashing
- Systems Manager Session Manager: SSH/RDP-free access to EC2 + on-prem (via Hybrid Activations), logged to CloudWatch + S3
- Patch Manager: baseline + scan + install patches via maintenance window
- Run Command: ad-hoc scripts on managed instances
- Automation documents: stitched workflows (Lambda + SSM + CFN)
- OpsCenter: ops issue management with auto-correlation
- Hybrid Activations: register on-prem servers as managed instances
- Well-Architected six pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability
- Lenses: Serverless, ML, Foundation Model, SaaS, IoT, FinServ, etc.
- License Manager: track vCPU/core/socket licenses, hard-stop EC2 launch when exceeded
- Service Quotas: centralized limit view + request increases + CloudWatch alarms at 80%

## Must-Mention Exam Traps

- EXAM TRAP: AWS Config records CHANGES — not real-time metrics; CloudWatch does metrics
- EXAM TRAP: CloudTrail data events (S3, Lambda) are NOT on by default + cost extra
- EXAM TRAP: Organization trail is configured from management account — applies to all members
- EXAM TRAP: CloudTrail Insights detects unusual API patterns (paid feature)
- EXAM TRAP: SSM Session Manager doesn't open inbound ports — connection initiated outbound from instance
- EXAM TRAP: SSM Agent must be installed + IAM role attached for SSM to manage an instance
- EXAM TRAP: Patch Manager + Maintenance Window run together — separate Run Command isn't continuous
- EXAM TRAP: Well-Architected Tool is a QUESTIONNAIRE — doesn't scan resources
- EXAM TRAP: Sustainability pillar exists (6th) — don't forget
- EXAM TRAP: Reliability pillar ≠ Resilience Hub — they complement
- EXAM TRAP: License Manager requires Dedicated Hosts for most BYOL Windows / Oracle scenarios
- EXAM TRAP: Service Quotas is per-Region — increase in us-east-1 doesn't increase eu-west-1
- EXAM TRAP: Some limits are NON-adjustable — must architect around (split accounts, etc.)
- EXAM TRAP: CloudTrail logs are NOT immediate — up to 15 min latency to S3
- EXAM TRAP: Config has billing per configuration item — high-churn workloads can drive cost up

## Key Decision Matrix

| Need | Pick |
|---|---|
| Track configuration drift + history | AWS Config |
| Audit who called which API | CloudTrail |
| SSH/RDP-free access to EC2 | SSM Session Manager |
| Patch fleet of EC2 / on-prem | SSM Patch Manager |
| Automated remediation of non-compliant resources | Config + SSM Automation |
| Org-wide CloudTrail | Organization trail |
| Strategic review of architecture | Well-Architected Tool |
| Track Windows / Oracle BYOL | License Manager |
| Increase EC2 vCPU limit | Service Quotas |
| Alarm before hitting service limit | Service Quotas + CloudWatch alarm |
| Continuous compliance evidence for SOC2 | Audit Manager (podcast 11) |

## Tone & Style

- Frame as the "what / who / fix" trio: Config (what), CloudTrail (who), SSM (fix)
- Mention SSM repeatedly with its sub-services
- Repeat the six WA pillars in order

## Rapid-Fire Closer

"What changed?" — "Config." "Who called the API?" — "CloudTrail." "Patch fleet?" — "SSM Patch Manager." "WA pillars count?" — "SIX." "Increase EC2 limit?" — "Service Quotas." "BYOL Windows?" — "License Manager."
