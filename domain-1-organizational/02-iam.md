# IAM (Identity and Access Management)

> **Centralized identity, authentication, and authorization for AWS — the backbone of every SAP-C02 security scenario.**

Maps to: **Domain 1.2 — Prescribe security controls**

---

## General Concepts

- **Resources** — user, group, role, policy, and identity provider objects stored in IAM
- **Identities** — IAM users, groups, roles (used to identify and group entities)
- **Entities** — IAM users and roles (used by AWS to authenticate)
- **Principals** — a person or app using AWS to sign in and make requests (the root user, IAM users, IAM roles, or federated users)
- **ARN format**: `arn:partition:service:region:account-id:resource-id`
  - Partitions: `aws` (commercial), `aws-cn` (China), `aws-us-gov` (GovCloud)
- **Access Advisor** — shows last accessed time for services per user/role; use for least-privilege cleanup

---

## STS (Security Token Service)

Returns temporary, limited-privilege credentials.

### API Actions

- **`AssumeRole`** — assume a role in the same or different account
- **`AssumeRoleWithSAML`** — federate via SAML 2.0 IdP
- **`AssumeRoleWithWebIdentity`** — federate via OIDC (Cognito, Google, Facebook, etc.)
- **`GetFederationToken`** — for proxy federation (custom identity broker)
- **`GetSessionToken`** — get temporary credentials for an IAM user (used with MFA enforcement)

### Credential Durations

- 15 minutes minimum to 36 hours maximum (depending on call type and role's max session duration)
- Default 1 hour for `AssumeRole`

### Key Concepts

- **External ID** — required for cross-account roles assumed by **third parties** (SaaS vendors); prevents the **confused deputy** problem. NOT a secret — it's a correlation ID.
- **Session tags** — pass tags during `AssumeRole`; accessible via `aws:PrincipalTag/<key>` in policies. Foundation for ABAC.
- **Source identity** — pass the original user's identity through role chains for audit

### STS Regional Endpoints (Resilience)

AWS strongly recommends **Regional STS endpoints** (`sts.<region>.amazonaws.com`) over the legacy global endpoint (`sts.amazonaws.com`).

- Global endpoint is hosted in **us-east-1 only** — no automatic failover
- Regional endpoints reduce latency and align credential scope with resource scope
- AWS CLI v2 sends STS calls to the Regional endpoint by default
- Set `AWS_STS_REGIONAL_ENDPOINTS=regional` for legacy SDKs
- STS is active in all enabled Regions by default; can be disabled per Region for governance

> ⚠️ **Exam scenario**: "Improve resilience of cross-account role assumption" → switch to **Regional STS endpoints** + build cross-Region failover logic.

---

## Full Policy Evaluation Logic

Critical for SAP-C02. Evaluation order (highest priority first):

1. **Explicit Deny** — anywhere in the chain → denied, full stop. No allow overrides.
2. **Organizations SCPs / RCPs** — SCP filters principals; RCP filters resources
3. **Resource-based policy** — if resource has a policy that allows, AND identity policy also allows
4. **Identity-based policy** — IAM policy attached to the principal
5. **Permissions boundary** — caps maximum permissions; identity ∩ boundary = effective
6. **Session policy** — passed via `AssumeRole`'s `Policy` parameter; further narrows the session

- **Same-account access**: union of identity-based + resource-based → minus boundary → minus session policy
- **Cross-account access**: requires BOTH identity policy in source account AND resource policy (or role trust policy) in target account

> ⚠️ **Explicit deny always wins.** No allow can override it.

### Troubleshooting "Access Denied"

Walk the chain in order:
1. Is there an explicit deny anywhere? (SCP, identity, resource, boundary, session)
2. Does the SCP allow the action?
3. Does an identity or resource policy explicitly allow?
4. Is the boundary capping it?
5. Is a session policy further restricting it?

---

## Policy Types Overview

Six policy types in IAM:

| Type | Attached to | Effect |
|---|---|---|
| **Identity-based** | User, group, role | Grants what the identity can do |
| **Resource-based** | Resource (S3, KMS, Lambda, SNS, SQS, ECR, EFS, Secrets Manager, etc.) | Grants what principals can do to the resource; always has `Principal` field |
| **Permissions boundary** | User or role (never group, never root) | Sets MAXIMUM permissions; never grants alone |
| **Organizations SCP** | OU or account | Guardrail; filters what identity policies can do in member accounts |
| **Organizations RCP** | OU or account | Resource-side guardrail |
| **Session policy** | An STS session | Further restricts a session; passed via `Policy` or `PolicyArns` parameter |

**Trust policy** — a special resource-based policy on **IAM roles**. Defines WHO (principal) can assume the role. Required on every role.

> 💡 **Trust policy vs permissions policy on a role**: trust policy = "who can assume me?"; permissions policy = "what can I do once assumed?"

### Session Policy Use Case

Identity policy allows `s3:*`, but you assume a role passing a session policy that allows only `s3:GetObject` on one bucket → the session can only do that. Useful for delegating short-lived, scoped access.

---

## IAM Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "OptionalIdentifier",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringEquals": {"aws:RequestedRegion": "eu-west-1"}
      }
    }
  ]
}
```

### Managed vs Inline

- **AWS-managed policies** — created and maintained by AWS (e.g., `AdministratorAccess`); versioned by AWS
- **Customer-managed policies** — your own managed policies; up to **5 versions** with `SetDefaultPolicyVersion` for rollback
- **Inline policies** — embedded directly in a single user/role; NO versioning, NO reuse

### NotAction with Allow

`NotAction` matches everything EXCEPT the listed actions. Combined with `Effect: Allow`, it grants everything except those actions — risky pattern, use with care. With `Effect: Deny`, it denies everything except listed actions (more common).

### Principal Options

- `*` — any principal (anonymous)
- `{"AWS": "arn:aws:iam::123:user/Alice"}` — specific IAM entity
- `{"AWS": "123456789012"}` — root of an account (≠ root user; means "delegate to that account's IAM")
- `{"Service": "lambda.amazonaws.com"}` — AWS service
- `{"Federated": "arn:aws:iam::123:saml-provider/Okta"}` — SAML IdP

### Condition Operators

- **String**: `StringEquals`, `StringNotEquals`, `StringLike` (wildcards)
- **Numeric**: `NumericEquals`, `NumericLessThan`, etc.
- **Date**: `DateGreaterThan`, `DateLessThan`
- **Boolean**: `Bool`
- **IpAddress** / **NotIpAddress** — supports CIDR
- **ArnEquals**, **ArnLike**
- **Null** — check if a condition key is present

### Variables and Tags

- `${aws:username}` — current IAM user name
- `${aws:userid}` — unique principal ID
- `${aws:PrincipalTag/<key>}` — tag on the principal (ABAC foundation)
- `${aws:ResourceTag/<key>}` — tag on the resource

---

## SCPs vs Permission Boundaries

| Attribute | SCP | Permission Boundary |
|---|---|---|
| **Scope** | OU or account (org-wide guardrail) | A specific user or role |
| **Attach to root** | Yes (root user in member accounts is affected) | No |
| **Attach to groups** | N/A | No (only users and roles) |
| **Grant permissions** | No (filters only) | No (caps only) |
| **Affects management account** | NO (management account is exempt) | Yes (if attached to its entities) |
| **Affects service-linked roles** | NO | Yes |
| **Combined effect** | Intersection at account level | Intersection at entity level |

> 🔥 **Key exam takeaway**: To restrict the root user in **member accounts** → use SCPs. No other mechanism (IAM policies, permission boundaries, Access Analyzer) can constrain root.
>
> Caveat: SCPs do NOT constrain the **management account root** — only IAM policies attached within that account can.

### Permissions Boundary Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:*", "ec2:Describe*"],
      "Resource": "*"
    }
  ]
}
```

Attached as a permissions boundary, this user/role can do AT MOST `s3:*` + `ec2:Describe*`, regardless of identity policy.

**Use case**: safely delegate role creation to development teams — set a boundary so they can't escalate beyond a specific scope.

---

## IAM Roles vs Resource-Based Policies

| Aspect | Roles | Resource-based policies |
|---|---|---|
| **Mechanism** | Principal assumes role → temporary creds | Principal accesses resource directly |
| **Original permissions** | **Surrendered** during assumption | **Preserved** (original + resource policy) |
| **Cross-account** | Yes (with trust policy) | Yes (with `Principal` field) |
| **Supported services** | All AWS services | S3, SNS, SQS, Lambda, ECR, Backup, EFS, Glacier, Cloud9, Secrets Manager, ACM, KMS, CloudWatch Logs, API Gateway, EventBridge, OpenSearch |

**Cross-account access** ALWAYS requires both: identity policy in source account + trust/resource policy in target account.

---

## MFA and Phishing-Resistant Authentication

### MFA Condition Keys

- **`aws:MultiFactorAuthPresent`** — `true` if MFA was used in the current session
- **`aws:MultiFactorAuthAge`** — seconds since MFA authentication

Example: deny sensitive actions unless MFA was authenticated within last hour:

```json
{
  "Effect": "Deny",
  "Action": "iam:DeleteUser",
  "Resource": "*",
  "Condition": {
    "NumericGreaterThan": {"aws:MultiFactorAuthAge": "3600"}
  }
}
```

### FIDO2 Passkeys (June 2024)

AWS supports **phishing-resistant MFA** via FIDO2.

- **Security keys (device-bound passkeys)** — physical hardware (YubiKey, etc.); single key supports multiple users
- **Synced passkeys** — biometric / credential-manager-backed (Apple iCloud Keychain, Google Password Manager, 1Password, Bitwarden, Dashlane, Microsoft account)
- Up to **8 MFA devices per root or IAM user**
- Phishing-resistant — cryptographic origin binding
- Works for console sign-in (not programmatic API calls)

### Root MFA Enforcement (2024–2025)

- All AWS account types require MFA on the root user for console access
- Member accounts must register MFA within **35 days** of first console sign-in (unless centralized root access is enabled)

> 💡 **Exam tip**: "phishing-resistant MFA" or "passwordless second factor" → **FIDO2 passkeys / security keys**, NOT virtual MFA (TOTP).

---

## Centralized Root Access Management

Launched 2024–2025; part of AWS Organizations.

- **Centrally manage and disable root credentials** across all member accounts from the management account
- **Remove root credentials** from member accounts entirely
- **Central root sessions** for privileged tasks — **max 15 minutes**
- Management account can perform root-only tasks on member accounts without member root credentials

**Supported privileged actions**:
- Deleting S3 bucket policies that deny all access
- Deleting SQS queue policies that deny all access
- Enabling/disabling account-level settings

> ⚠️ **Exam pattern**: "manage root access across many accounts with least operational overhead" → **Centralized Root Access Management**.

---

## IAM Roles Anywhere

Launched 2022. Lets on-premises and hybrid workloads use IAM roles **without long-term access keys**.

Authentication via **X.509 certificates** issued by a Certificate Authority (CA).

### How It Works

1. Create a **Trust Anchor** — references a CA (AWS Private CA or external CA)
2. Create a **Profile** — specifies which IAM roles can be assumed + optional session policies
3. On-prem workload presents its X.509 cert to the Roles Anywhere endpoint
4. Roles Anywhere validates against the Trust Anchor
5. Returns temporary AWS credentials (like STS `AssumeRole`)

### Use Cases

- On-prem servers needing AWS access (S3, DynamoDB, etc.)
- Hybrid architectures
- IoT devices or edge computing
- Replacing distributed access keys with short-lived role credentials

> 💡 **Exam pattern**: on-prem workloads need AWS access **without long-term credentials** → **IAM Roles Anywhere**.

As of 2026, supports post-quantum cryptography (FIPS 204 / ML-DSA).

---

## IAM Access Analyzer

Three analyzer types since 2024.

### 1. External Access Analyzer (original)

- Identifies resources shared with **external principals** (outside your zone of trust)
- Zone of trust = your account or your Organization
- **Free**

### 2. Unused Access Analyzer (2023 GA, expanded 2024)

- Identifies **unused IAM roles, access keys, passwords, and permissions**
- Finds:
  - Unused roles (not assumed within specified period)
  - Unused access keys
  - Unused passwords
  - Unused permissions (granted actions never invoked)
- **Paid per analyzer per month**

### 3. Internal Access Analyzer (2024)

- Identifies resources shared **within your organization** across accounts
- Surfaces unintended internal cross-account access

### Custom Policy Checks

- **`CheckAccessNotGranted`** — confirm a policy does NOT grant a specific action
- **`CheckNoNewAccess`** — confirm a new policy version doesn't expand access vs old
- **`CheckNoPublicAccess`** — confirm resource policy doesn't permit public access
- Used in CI/CD to gate policy changes before deployment

### Other Capabilities

- **Policy validation** — grammar, best-practice warnings, security warnings
- **Policy generation** — generates least-privilege policy from CloudTrail history (90-day lookback)

**14+ supported resource types**: S3, IAM roles, KMS, Lambda functions + layers, SQS, Secrets Manager, SNS, EBS snapshots, RDS DB/Cluster snapshots, ECR, EFS, DynamoDB tables, DynamoDB streams.

> 💡 **Exam pattern**: "find unused roles or permissions for least-privilege" → **Unused Access**. "Validate a policy in CI before deploy" → **Custom Policy Checks**.

---

## Credential Reports & Aged Access Keys

### IAM Credential Report

- Lists every IAM user with: passwords, access keys, MFA status, key rotation dates, last used
- Generated max **every 4 hours**
- APIs: `GenerateCredentialReport`, `GetCredentialReport`

### Managing Aged Access Keys

- **AWS Config managed rule**: `access-keys-rotated` — flags keys not rotated within N days
- Combine with **Systems Manager Automation** for remediation (deactivate, notify, rotate)
- Combine with **EventBridge** for alerting

### AWSCompromisedKeyQuarantine

AWS-managed policy automatically attached to an IAM user when AWS detects their credentials have been exposed publicly (e.g., committed to GitHub). The policy denies sensitive actions until you investigate and rotate.

---

## Policy Versioning

- **Customer-managed policies** — up to **5 versions**; use `SetDefaultPolicyVersion` for rollback
- **AWS-managed policies** — versioned by AWS; can't be rolled back manually
- **Inline policies** — NO versioning

### Policy Simulator

- Test policies before applying to production
- Troubleshoot existing user permissions
- Simulates SCP impact for Organizations
- Useful for "why is this user denied?" investigations

---

## Service-Linked Roles

A unique IAM role linked to a specific AWS service (e.g., `AWSServiceRoleForElasticLoadBalancing`).

- **Predefined by the AWS service** — you don't craft the permissions
- **Trust policy is immutable** — only that service can assume it
- **Permission policy is managed by the service** — can't be edited (AWS updates it as service evolves)

### Creation

- Most services create them automatically on first use
- Some require manual creation via `iam:CreateServiceLinkedRole`

### Deletion

- Can be deleted only after the linked service has no dependent resources
- Service returns a list of dependencies blocking deletion

> ⚠️ **SCPs do NOT restrict service-linked roles.** They bypass SCP restrictions. Critical exam trap.

### Common Services Using Service-Linked Roles

- Auto Scaling, ELB, Lambda (when accessing other services via console)
- Organizations, Config, CloudTrail, GuardDuty, Security Hub, Inspector, Macie
- AWS Backup, Compute Optimizer, Trusted Advisor
- ECS, EKS service operations

---

## IAM Identity Center (formerly AWS SSO)

Modern recommended approach for human access to AWS at scale.

### Identity Sources (pick ONE per instance)

1. **Built-in identity store** — managed in Identity Center directly
2. **AWS Managed Microsoft AD or AD Connector** — sync from on-prem AD
3. **External IdP** via SAML 2.0 + SCIM:
   - **Microsoft Entra ID** (formerly Azure AD)
   - **Okta**
   - **Google Workspace**
   - **Ping Identity**
   - **JumpCloud**
   - **OneLogin**
   - Any SAML 2.0 + SCIM-compliant provider

### SCIM v2.0 Provisioning

- Provisions and de-provisions users + groups from external IdP automatically
- Joiner / leaver / mover events propagate
- Required for production federations at enterprise scale

### Multi-Region Replication (2024)

- Replicate Identity Center instance across multiple Regions for resilience + low latency
- One primary Region, multiple read-only replicas

### Permission Sets

- Reusable templates defining IAM policies + permission boundaries
- Assigned to users/groups against AWS account + role pairs
- Synced as IAM roles into target accounts when first assigned

### Applications Integration

- Pre-integrated SaaS apps (Salesforce, Box, M365, Slack, etc.)
- SAML 2.0 custom apps
- **Trusted token issuer** for AWS analytics (Redshift, Athena, EMR Studio, QuickSight) — federate user identity all the way down to data queries

> 💡 **Identity Center vs federation directly to IAM**: Identity Center is the modern recommendation. Direct SAML/OIDC federation to IAM still works for Cognito Identity Pool use cases and some legacy setups.

---

## Cognito

### User Pools

- Sign-up and sign-in for app users
- Returns **JWT tokens** (ID, Access, Refresh)
- MFA, password policies, user attributes, custom triggers (Lambda)
- Federation with Google, Facebook, Apple, SAML, OIDC

### Identity Pools

- Exchange tokens (from User Pool or external IdP) for **temporary AWS credentials** via STS
- Allow app users to call AWS services directly (e.g., upload to S3)
- Authenticated and unauthenticated (guest) identities

> 💡 **Common combined pattern**: User Pool for sign-in + Identity Pool for AWS resource access.

> ⚠️ Cognito Sync deprecated — migrate to **AWS AppSync** for syncing data across devices.

---

## S3 Bucket Policies, ACLs, Access Points

### Bucket Policies

- Resource-based policy on the bucket
- Cross-account access without sharing IAM users
- Size limit: **20 KB**
- Use over ACLs for almost everything

### Block Public Access (BPA)

- Account-level + bucket-level setting
- Overrides any policy granting public access
- **Default ON** for new buckets

### S3 ACLs

- Legacy; AWS recommends bucket policies + IAM policies
- Disabled by default since April 2023 (Object Ownership = Bucket owner enforced)

### S3 Access Points

- Named endpoints per use case, each with its own policy
- Optional VPC restriction (private-only access points)
- Useful when many apps need different scopes against the same bucket

### Forcing TLS / HTTPS

```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": ["arn:aws:s3:::my-bucket/*", "arn:aws:s3:::my-bucket"],
  "Condition": {"Bool": {"aws:SecureTransport": "false"}}
}
```

### Forcing KMS Encryption

```json
{
  "Effect": "Deny",
  "Action": "s3:PutObject",
  "Condition": {
    "StringNotEquals": {"s3:x-amz-server-side-encryption": "aws:kms"}
  }
}
```

---

## Users, Groups, Roles

### IAM Users

- Long-lived identity (anti-pattern for humans at scale — use Identity Center instead)
- Access keys: 20-char ID + 40-char secret; **max 2 per user**; secret shown only at creation
- Password policy: length, complexity, rotation, history
- Best practice: programmatic access for service accounts; federation for humans

### Groups

- Container of IAM users (NOT roles, NOT other groups)
- A user can belong to **max 10 groups**
- Default limit: **300 groups per account** (raised from 100)
- Used to assign common policies to many users

### Roles

Four common role types:
- **AWS service role** — assumed by an AWS service (e.g., EC2 instance role, Lambda execution role)
- **Service-linked role** — predefined by a service, trust policy immutable
- **Cross-account role** — assumed by principals in another AWS account
- **Identity provider role** — assumed via SAML / OIDC federation

### Access Keys vs EC2 Key Pairs

- **IAM access keys** — for programmatic AWS API authentication (`aws_access_key_id` + `aws_secret_access_key`)
- **EC2 key pairs** — SSH key pair for connecting to Linux instances (PEM file); unrelated to IAM
- **Never store IAM access keys on EC2** — use **instance profiles** (auto-rotated temp credentials via instance metadata)

---

## VPC Endpoint Policies

Resource-based policy on a VPC endpoint that controls which API calls can be made through the endpoint.

- **Gateway Endpoints** (S3, DynamoDB) — endpoint policy filters API calls
- **Interface Endpoints** (PrivateLink) — endpoint policy + security groups
- Layered with: identity policy + resource policy + service control policy
- Common pattern: lock down S3 access through VPC endpoints to a specific bucket + account

---

## Identity Federation Quick Reference

| Provider type | AWS integration |
|---|---|
| **Enterprise (SAML 2.0)** | IAM Identity Center, or `AssumeRoleWithSAML` directly |
| **Active Directory (Windows)** | AWS Managed Microsoft AD, AD Connector, or ADFS → SAML |
| **Web (Google, Facebook, Apple, OIDC)** | Cognito Identity Pool (recommended) or `AssumeRoleWithWebIdentity` directly |
| **Custom Identity Broker** | `GetFederationToken` (only when IdP doesn't speak SAML/OIDC) |

---

## Exam Tips

- **Federate via IAM Identity Center** — don't create long-lived IAM users for humans
- **Use Roles Anywhere** for on-premises / hybrid workloads instead of distributing IAM access keys
- **Always prefer Regional STS endpoints** for cross-account role assumption
- **Access Analyzer Unused Access** finds dormant roles, keys, passwords, and unused permissions
- **External Access** analyzer = free; **Unused Access** = paid per analyzer per month
- **Permissions boundary** caps an entity's permissions (intersection); never grants
- **Session policies** further narrow an assumed role's permissions — useful for delegating short-lived scoped access
- **Phishing-resistant MFA** = **FIDO2 passkeys** or hardware security keys (NOT virtual MFA / TOTP)
- **Centralized Root Access Management** removes root credentials from member accounts entirely; root sessions are max 15 min
- **Cross-account access** = identity policy in source account ∩ trust policy/resource policy in target account
- **`aws:PrincipalTag`** + session tags enable ABAC; scales better than identity-based for large orgs
- **`aws:RequestedRegion`** restricts which Regions an action targets — common compliance pattern
- **STS `External ID`** is required for cross-account roles assumed by third parties; prevents confused deputy
- **Trust policy** (resource-based, on a role) defines WHO can assume; permissions policy defines WHAT it can do
- **Identity Center + Trusted Token Issuer** federates identity into Redshift, Athena, EMR Studio, QuickSight

---

## Exam Traps

- An explicit deny **always wins** — no Allow, resource policy, or boundary can override it
- SCPs **do not apply to service-linked roles** — SCPs alone cannot block a service that uses them
- SCPs **do not apply to the management account root** — restricting it requires IAM policies within that account
- Permissions boundary is **per-entity**; SCP is **org-wide** — they are different mechanisms despite both capping permissions
- Cross-account access via resource-based policies requires **both sides** — a resource policy in the target account AND an identity policy in the source account
- IAM groups can contain **users only** — not roles, and not other groups
- Assuming a role **surrenders original permissions** — the assumed identity has only the role's permissions
- Only customer-managed policies support **5-version rollback** — inline policies have no versioning
- IAM user access keys are **long-lived** — for temporary credentials, use roles + STS instead
- Use **Regional STS endpoints** for production — the global endpoint has a single-Region (us-east-1) failure mode
- Use **instance profiles** for EC2 — never store access keys on instances
- Cognito User Pools handle **sign-in (JWT)**; Identity Pools **exchange tokens for AWS temp creds** — they are different services
- The current name is **AWS IAM Identity Center** (renamed 2022) — "AWS SSO" is the old name
- Virtual MFA (TOTP) is **not phishing-resistant** — use FIDO2 passkeys or hardware security keys instead
- `aws:SourceIp` **does not work for VPC endpoint traffic** — use `aws:VpcSourceIp` or `aws:SourceVpce` instead
- Service-linked role deletion can **fail if dependencies exist** — the service returns a list of blocking dependencies that must be cleaned up first
- External ID is for **confused-deputy prevention**, not authentication — it is not a secret
- Identity Center supports **one identity source per instance** — switching loses existing assignments
- Permissions boundaries attach to **users and roles only** — never groups, never root
- Cross-account role chaining has a **max 1-hour session duration** — this is not the role's MaxSessionDuration setting
