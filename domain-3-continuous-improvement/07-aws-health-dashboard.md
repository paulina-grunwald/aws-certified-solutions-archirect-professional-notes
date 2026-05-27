# AWS Health Dashboard (+ AWS Health API)

> **Personalized view of AWS service health for your account + Organization. Two scopes: Service Health Dashboard (public, global service status) and Your Account Health Dashboard (events affecting YOUR resources). AWS Health API provides programmatic access — events can be routed via EventBridge for automated remediation. Common SAP-C02 scenarios: react to scheduled maintenance, EC2 retirement events, certificate expiry, account-level issues. Used to be split into "Service Health Dashboard" + "Personal Health Dashboard" — unified into "AWS Health Dashboard" (2022).**

Maps to: **Domain 3.1 — Reliability**, **Domain 3.6 — Operational excellence**, **Domain 1.4 — Multi-account governance**

---

## Two Views

### Service Health Dashboard (Public)
- **status.aws.amazon.com** — public page anyone can view
- **Region-level** + service-level status
- General health of AWS services (not personalized)

### Your Account Health Dashboard
- Events that **affect your AWS resources specifically**
- Personalized — only events for your accounts, regions, services
- Three event categories:
  - **Open issues** — current AWS-side problems affecting your resources
  - **Scheduled changes** — upcoming AWS maintenance
  - **Other notifications** — account/security/billing notices

## Event Categories

| Category | Examples |
|---|---|
| **Issue** | Service degradation in a Region |
| **Scheduled change** | EC2 hardware retirement, EBS volume retirement, RDS maintenance window |
| **Account notification** | Compromised credentials, account-level changes, certificate expiry, security advisories |

## AWS Health API

- Programmatic access to your account's health events
- **Requires Business, Enterprise On-Ramp, or Enterprise Support** for API access
- Operations: `DescribeEvents`, `DescribeAffectedEntities`, `DescribeEventDetails`
- **Aggregate view across an Organization** — see all member-account events from the management account (Organization-wide Health enabled)

## EventBridge Integration

- Health events are published to **default EventBridge bus** as events
- Detail types: `AWS Health Event`, `AWS Health Abuse Event`
- Build automated reactions:
  - Notify Slack / PagerDuty via SNS + Lambda
  - Auto-snapshot EBS volumes before scheduled retirement
  - Migrate EC2 instances ahead of underlying hardware retirement
  - Open a Jira ticket for the on-call team

This is the **key automation pattern** — Health Dashboard alone is passive; EventBridge makes it actionable.

## Organization View

- Enable **Organization-wide AWS Health** from the management account
- Aggregates events across all member accounts
- Single pane for central ops / cloud platform teams
- Requires `EnableHealthServiceAccessForOrganization` API call

## Cross-Account / Cross-Region Considerations

- Events are surfaced **per affected Region**
- Multi-region workloads need to monitor health events in each Region
- Use EventBridge multi-region rules or aggregate to a central account

## Common Event Patterns

### EC2 Scheduled Retirement
- AWS notifies that underlying hardware is being retired
- Action: stop/start the instance (moves to new hardware) before deadline
- Automate via EventBridge → Lambda → API call

### RDS Maintenance Window
- AWS notifies of upcoming RDS patching
- Action: confirm Multi-AZ failover will absorb downtime, or schedule app-side maintenance window

### ACM Certificate Expiry
- AWS notifies that certificate is approaching expiry
- For non-AWS-imported certs that don't auto-renew
- Action: renew before expiry; replace ACM cert

### Compromised IAM Credentials
- AWS notifies that an access key was found in public repo / data leak
- Action: rotate / disable key immediately

### Abuse Reports
- AWS notifies of abuse reports against your resources (e.g., port scans, spam)
- Action: investigate and respond within deadline (else account suspension risk)

## Support Plan Requirements

| Feature | Required Plan |
|---|---|
| Account Health Dashboard (UI) | All plans (Basic included) |
| AWS Health API | Business / Enterprise On-Ramp / Enterprise |
| Organization-wide Health | Business+ |
| Real-time event delivery | All plans |

## Pricing

- **Free** — the dashboard + EventBridge events
- Costs only if you build downstream automation (Lambda, SNS, etc.)

## Common Patterns

### Auto-remediate hardware retirement
- EventBridge rule: `detail-type = "AWS Health Event" AND service = "EC2" AND eventTypeCategory = "scheduledChange"`
- Target: Lambda that stops/starts the affected instance during off-hours

### Centralized health monitoring
- Enable Org-wide Health in management account
- EventBridge rule in management account → SNS topic
- All accounts' events visible to platform team

### Compliance evidence
- Stream health events to S3 via EventBridge → Kinesis Firehose
- Auditable record of AWS-side incidents affecting workloads

## Exam Tips

- "React to scheduled AWS maintenance" → **AWS Health Dashboard + EventBridge**
- "Notify operators of EC2 hardware retirement" → **EventBridge rule on Health event**
- "Centralize health visibility across the Organization" → **Org-wide Health from management account**
- **Personal Health Dashboard** is the old name (renamed)
- Events are categorized: **issue / scheduledChange / accountNotification**
- API requires **Business+ Support**
- EventBridge is the path to automated remediation

## Exam Traps

- **Health Dashboard ≠ CloudWatch alarms** — Health is AWS-side events; CloudWatch is your resource metrics
- **Health Dashboard ≠ Trusted Advisor** — Health surfaces AWS-initiated events; TA surfaces account best-practice gaps
- **Service Health Dashboard (status.aws.amazon.com) is public + generic** — your Account Health Dashboard is personalized
- **Health Dashboard doesn't fix anything** — you build the automation via EventBridge
- **API access requires Business Support** — Basic plan = dashboard UI only
- **Org-wide Health requires explicit enablement** — `EnableHealthServiceAccessForOrganization` API
- **Events are eventually consistent across Regions** — assume up to a few minutes delay
