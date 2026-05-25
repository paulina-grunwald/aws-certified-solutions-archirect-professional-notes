# Podcast 03 — Federated Identity

**Length target**: 15 min (~2,200 spoken words)
**Repo references**: `domain-1-organizational/03-iam-identity-center.md`, `04-aws-directory-service.md`, `40-cognito.md`, `02-iam.md` (federation section)

## Topic & Scope

Three identity systems get confused on the exam: IAM Identity Center (workforce SSO), Directory Service (Active Directory), and Cognito (customer identity). This podcast nails when to pick each.

## Service Coverage Depth

**Deep**:
- IAM Identity Center — workforce SSO, account-level access, permission sets
- Cognito User Pools vs Identity Pools — the most-confused pair on SAP-C02
- Directory Service three flavors: Managed Microsoft AD, AD Connector, Simple AD

**Brief**:
- SAML 2.0 + OIDC concepts
- IAM identity provider for SAML
- Cognito advanced (custom triggers, hosted UI)

## Structured Outline

1. **Open (30s)** — "Workforce vs customers vs Windows — three identity worlds, three AWS services. Don't mix them up."
2. **IAM Identity Center (4 min)** — replacement for SSO, central workforce identity for AWS accounts + SaaS, permission sets, integration with external IdPs (Okta, Azure AD, Entra)
3. **Directory Service (3 min)** — Managed AD (full AD in AWS), AD Connector (proxy to on-prem AD), Simple AD (small/cheap, not a real AD)
4. **Cognito (4 min)** — User Pools (sign-up + sign-in for your app users), Identity Pools (federated AWS credentials), the difference is the whole exam question
5. **Decision Matrix (1.5 min)**
6. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- IAM Identity Center: workforce SSO across multi-account AWS Org + external SaaS apps
- Identity Center integrates with Okta, Entra ID (Azure AD), Google Workspace, OneLogin, JumpCloud
- Identity Center permission sets = templated IAM permissions assigned per account
- Cognito User Pool = identity provider for YOUR app's customer accounts
- Cognito Identity Pool = exchange any token for temporary AWS credentials
- Managed Microsoft AD: full AD features, trusts with on-prem, group policies
- AD Connector: lightweight proxy from AWS to on-prem AD — no AD in AWS
- Simple AD: small workgroups, NOT a real AD (Samba-based)
- For workforce identity to AWS accounts → Identity Center
- For end-user identity to your mobile/web app → Cognito User Pool

## Must-Mention Exam Traps

- EXAM TRAP: Identity Center ≠ Cognito — Identity Center is for your employees; Cognito is for your customers
- EXAM TRAP: Cognito User Pool ≠ Identity Pool — User Pool authenticates; Identity Pool authorizes for AWS access
- EXAM TRAP: AD Connector does NOT store users in AWS — it proxies to on-prem
- EXAM TRAP: Simple AD is NOT a real AD — limited features, can't trust with corporate AD
- EXAM TRAP: IAM Identity Center used to be called "AWS SSO" — questions may still use old name
- EXAM TRAP: SAML federation to IAM uses an IAM Identity Provider object — separate from Identity Center
- EXAM TRAP: Cognito Identity Pool supports unauthenticated guests too — special role
- EXAM TRAP: To trust on-prem AD from AWS workloads → Managed AD with one-way or two-way trust, NOT AD Connector
- EXAM TRAP: WorkSpaces / RDS for SQL Server need a real Directory Service (not Identity Center)

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| Employees sign in to multiple AWS accounts + Salesforce + Slack | IAM Identity Center |
| Customers sign up for your mobile app | Cognito User Pool |
| Mobile app needs temporary AWS creds to upload to S3 | Cognito Identity Pool |
| Domain-joined EC2 Windows instances using on-prem AD | AD Connector or Managed AD with trust |
| Small dev team needs LDAP/AD-like service in AWS | Simple AD (or just Managed AD) |
| Federate corporate Okta/Entra to AWS accounts | IAM Identity Center with external IdP |

## Tone & Style

- Use the "workforce vs customer vs Windows" framing throughout
- Hammer the User Pool vs Identity Pool distinction with concrete examples
- Mention every renamed service name: "AWS SSO is now IAM Identity Center"

## Rapid-Fire Closer

"Employee SSO to AWS?" — "Identity Center." "Customer sign-up?" — "Cognito User Pool." "App needs AWS creds?" — "Cognito Identity Pool." "On-prem AD trust?" — "Managed AD." "Simple AD is a real AD?" — "NO."
