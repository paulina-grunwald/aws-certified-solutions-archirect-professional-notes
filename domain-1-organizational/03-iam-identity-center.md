# IAM Identity Center (formerly AWS Single Sign-On)

> **Central place to create or connect workforce users and manage their access across all AWS accounts in AWS Organizations + cloud apps. The modern, recommended approach for human access to AWS at scale.**

Maps to: **Domain 1.2 — Prescribe security controls** (multi-account identity, federation)

---

## Overview

- **One login** (single sign-on) for:
  - **AWS accounts** in AWS Organizations
  - **Cloud business applications** (Salesforce, Box, Microsoft 365, Slack, etc.)
  - **SAML 2.0**–enabled applications
  - **EC2 Windows instances** (via WorkSpaces integration)
  - AWS analytics services via **Trusted Identity Propagation** (Redshift, QuickSight, EMR, Lake Formation, S3 Access Grants, Athena, Q Business)
- Provides an **access portal** (per-account dashboard) where users see only the AWS accounts and apps they've been assigned
- Replaces the legacy AWS SSO service; existing SSO configurations migrate automatically

---

## Identity Sources (pick ONE per Identity Center instance)

### 1. Built-in identity store (Identity Center Directory)

- Created and managed in Identity Center directly
- Users and groups stored locally in AWS
- Quick start for small orgs

### 2. AWS Directory Service

- **AWS Managed Microsoft AD** or **AD Connector** (proxy to on-prem AD)
- Useful when you already run AD on-prem or want full AD features

### 3. External Identity Provider (IdP) via SAML 2.0 + SCIM

- **Microsoft Entra ID** (formerly Azure AD)
- **Okta** (Universal Directory)
- **Google Workspace** (users only via SCIM)
- **Ping Identity** (PingFederate full; PingOne users only)
- **JumpCloud**
- **OneLogin**
- Any SAML 2.0 + SCIM-compliant IdP

> ⚠️ **Identity source is one-way**: you pick ONE identity source per Identity Center instance. Switching loses existing assignments.

---

## SCIM v2.0 Automatic Provisioning

**System for Cross-domain Identity Management** v2.0 — open standard for automated user/group provisioning.

- Propagates **joiner / leaver / mover** events from external IdP into Identity Center automatically
- Provisions and **de-provisions** users + groups without manual sync
- Required for production federations at enterprise scale

### Coverage

- **Full SCIM (users + groups)**: Entra ID, Okta, OneLogin, PingFederate
- **Users only via SCIM**: Google Workspace, PingOne
- Setup: enable provisioning in Identity Center → get SCIM endpoint + token → configure the same in the IdP

---

## Multi-Region Replication (2024)

- Replicate Identity Center instance across multiple Regions for **resilience + low latency** for globally distributed workforce
- **One primary Region**, multiple **read-only replicas**
- Writes happen in the primary Region; reads can happen in any replica
- Improves access portal sign-in performance for users far from the primary Region
- Failover is not automatic; primary Region promotion is manual

---

## Permission Sets

**Permission sets** = reusable templates defining what someone can do in an AWS account.

- A permission set contains **IAM managed policies + customer managed policies + inline policy + permissions boundary**
- Assigned to **users or groups** against **AWS account + permission set** pairs
- Identity Center materializes the permission set as an **IAM role** in the target account when first assigned (role name pattern: `AWSReservedSSO_<PermissionSetName>_<hash>`)

### Two kinds

- **AWS-managed permission sets** (predefined): `AdministratorAccess`, `PowerUserAccess`, `ReadOnlyAccess`, `ViewOnlyAccess`, etc.
- **Customer-managed permission sets**: your own definitions; can reference customer-managed IAM policies that you create in advance in the target accounts

### Other Properties

- **Session duration**: configurable per permission set, 1–12 hours (default 1 hour)
- **Block usage via SCP**: as of June 2025, you can use an Organizations SCP to **block specific permission sets** from being used in member accounts

---

## Trusted Identity Propagation (TIP)

Propagate a user's **workforce identity** from the IdP all the way to data and analytics services.

- Built on **OAuth 2.0** Authorization Framework
- Applications access data **on behalf of a specific user**, without sharing credentials
- Permissions are enforced **per-user** in the target service (not per-role)

### Supported AWS services with TIP

- **Amazon Q Business** (workforce assistant)
- **Amazon QuickSight** (BI dashboards)
- **Amazon Redshift** (data warehouse — including Redshift Data API as of March 2025)
- **Amazon EMR** (big data)
- **AWS Lake Formation** (fine-grained data lake permissions)
- **Amazon S3 Access Grants** (per-user S3 access)
- **Amazon Athena** (federated queries)
- **AWS Glue** (catalog access)

> 💡 **Why TIP matters**: previously, analytics queries ran as a shared IAM role → audit logs showed the role, not the user. With TIP, audit logs show the actual user identity, and fine-grained per-user permissions can be enforced down to row/column level in Lake Formation.

**Trusted Token Issuer**: configures Identity Center to trust an external IdP's tokens (OIDC) for TIP flows. Useful when you want to use your existing IdP rather than Identity Center as the token issuer.

---

## Application Assignments

### SAML 2.0 business applications

- Pre-integrated SaaS apps: Salesforce, Box, Microsoft 365, Slack, Zoom, GitHub Enterprise, ServiceNow, Workday, Tableau Cloud, etc.
- Custom SAML 2.0 apps: configure via metadata + ACS URL
- Identity Center acts as the SAML IdP for the app

### AWS-managed applications

- Built-in AWS analytics services (Redshift, QuickSight, EMR, Lake Formation, S3 Access Grants, Q Business) — connected via TIP
- No SAML config needed; the app is "AWS-aware"

---

## Attribute-Based Access Control (ABAC)

- Fine-grained permissions based on **user attributes** stored in Identity Center identity store (or synced from IdP)
- Common attributes: cost center, title, department, locale, region, business unit
- Permission set policies reference attributes via **`aws:PrincipalTag/<key>`**
- **Use case**: define permissions once, then modify AWS access by changing the user's attributes in the IdP — no IAM policy changes

**Example**: permission set allows S3 access only if `${aws:PrincipalTag/CostCenter}` matches the bucket's `CostCenter` tag.

---

## Customizable Access Portal URL

- Default access portal URL: `https://d-xxxxxxxxxx.awsapps.com/start`
- Can configure a **custom subdomain**: `https://yourcompany.awsapps.com/start`
- **Vanity domain support** (2024+): use your own DNS domain (e.g., `https://signin.yourcompany.com`) via the Identity Center custom domain feature
- Improves user experience and brand consistency

---

## CLI v2 and SDK Integration

- **`aws configure sso`** (CLI v2) creates a profile that uses Identity Center for credentials
- CLI v2 launches a browser to the access portal for sign-in, then caches short-lived credentials
- No long-lived access keys on developer machines
- Programmatic access via the `sso` and `identitystore` APIs
- Supports **CLI session profile** for scripts and CI / CD pipelines (via OIDC device authorization grant)

---

## MFA Enforcement

- MFA can be **required at the Identity Center identity store level** OR delegated to the external IdP
- Supported MFA methods (Identity Center identity store):
  - **FIDO2 security keys / passkeys** (recommended, phishing-resistant)
  - **Virtual MFA (TOTP)** (Authenticator apps)
  - **Built-in authenticators** (Touch ID, Windows Hello)
- **Conditional MFA**: require MFA only when sign-in is suspicious (new device, new location)
- When using external IdP, MFA enforcement is delegated to the IdP (Entra ID Conditional Access, Okta MFA, etc.)

---

## Audit and CloudTrail

- All Identity Center API calls logged to **CloudTrail**
- Permission set assignments, identity source changes, application assignments — all logged
- For sign-in events: CloudTrail captures the federation events; the IdP captures the actual authentication
- Combined with TIP: CloudTrail in target service (Redshift, QuickSight, etc.) shows the **actual user identity**, not just the role

---

## Identity Center vs Federation Directly to IAM

| Aspect                | Identity Center              | Direct IAM Federation (SAML/OIDC)            |
| --------------------- | ---------------------------- | -------------------------------------------- |
| Scope                 | All accounts in Organization | Single account at a time                     |
| Permission management | Permission sets (reusable)   | IAM roles per account                        |
| Multi-account access  | Native, via assignments      | Manual role + trust per account              |
| App SSO               | Yes (SAML + AWS-managed)     | No (would need separate SAML config per app) |
| Workforce identity    | Yes (TIP for analytics)      | No                                           |
| Modernity             | Recommended                  | Legacy / Cognito Identity Pool use cases     |

---

## Exam Tips

- **IAM Identity Center is the modern recommended approach** for human access to AWS at scale — federate via Identity Center, don't create long-lived IAM users
- **One identity source per instance** — pick built-in, AWS Directory Service, or external IdP
- **SCIM v2.0** automatic provisioning is required for enterprise federation — joiner/leaver/mover events propagate automatically
- **Microsoft Entra ID**, **Okta**, **OneLogin**, **PingFederate** support full SCIM (users + groups); **Google Workspace** and **PingOne** support users only via SCIM
- **Permission sets** become IAM roles in target accounts (`AWSReservedSSO_*`) when assigned
- **AWS-managed permission sets**: AdministratorAccess, PowerUserAccess, ReadOnlyAccess, ViewOnlyAccess
- **Trusted Identity Propagation (TIP)** = workforce identity flows all the way to analytics services (Redshift, QuickSight, EMR, Lake Formation, S3 Access Grants, Q Business)
- TIP enables **per-user audit trails** in analytics services (CloudTrail shows user, not role)
- **Customer-managed permission sets** reference IAM policies you create in target accounts (must exist before assignment)
- **Identity Center supports MFA** including FIDO2 passkeys, TOTP, and conditional MFA
- **ABAC with `aws:PrincipalTag`** lets you change AWS permissions by changing IdP attributes — no policy changes needed
- **Multi-Region replication** provides resilience + low latency for global workforce
- **`aws configure sso`** (CLI v2) for short-lived credentials on developer machines
- **SCPs can block specific permission sets** from being used
- For **WorkSpaces, WorkMail, WorkDocs**, etc. with on-prem AD — Identity Center is layered on top of AWS Managed Microsoft AD (with two-way trust to on-prem)

---

## Exam Traps

- The current name is **AWS IAM Identity Center** (renamed 2022) — "AWS SSO" is the old name. If the question mentions AWS SSO, it means IAM Identity Center
- You pick **one identity source per instance** — switching loses existing assignments. Multi-IdP per instance is not supported
- Identity Center is for **workforce users (B2E)**; Cognito is for **app users (B2C)** — they serve different audiences
- Permission sets are **reusable templates** — when assigned to a new account, Identity Center automatically creates the corresponding IAM role with the `AWSReservedSSO_*` prefix. They are not unique roles
- Referenced IAM policies must **exist in the target accounts before assignment** — otherwise customer-managed permission set assignment fails
- Identity Center sessions last **1-12 hours** — longer sessions require re-authentication
- AWS-managed permission sets are **read-only** — to customize them, clone as customer-managed
- TIP is for **specific AWS-managed apps** only (Q, QuickSight, Redshift, EMR, Lake Formation, S3 Access Grants, Athena, Glue) — it is not for general AWS API access
- TIP requires **application-level support** — you cannot bolt TIP onto an arbitrary service
- MFA enforcement in Identity Center applies only with the **built-in identity store** — with an external IdP, MFA is enforced at the IdP
- Google Workspace and PingOne support SCIM for **users only** — groups must be managed manually in Identity Center
- Identity Center is **region-scoped at the instance level** — multi-Region replication (one primary, read-only replicas) mitigates this
- SCPs are **org-wide guardrails**; permission sets are **role definitions for assigned users** — Identity Center does not replace SCPs
- Account assignments are **per-account, per-permission-set** — assigning a user to one account does not give them access to other accounts
- Direct SAML 2.0 federation to IAM is the **legacy approach** that creates IAM roles manually per account — Identity Center is the modern replacement
