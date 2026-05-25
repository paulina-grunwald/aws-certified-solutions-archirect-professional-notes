# AWS Directory Service

> **Family of managed directory services for using Microsoft Active Directory (and AD-compatible identities) with AWS resources, on-premises systems, and AD-aware applications.**

Maps to: **Domain 1.2 — Prescribe security controls** (multi-account identity, federation)

---

## AD vs Non-AD Compatible Services

| AD compatible (use AWS Directory Service)                           | Not AD compatible (separate services)                                     |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| **AWS Managed Microsoft AD** — full Microsoft AD on Windows servers | **Amazon Cloud Directory** — hierarchical multi-schema directory for apps |
| **AD Connector** — proxy to on-prem AD (no directory in AWS)        | **Amazon Cognito User Pools** — sign-up/in for SaaS / mobile apps         |
| **Simple AD** — Samba-based standalone (≤5,000 users)               | **AWS IAM Identity Center** — federation hub on top of identity sources   |

---

## Overview

- Family of **managed services** that connect AWS to existing on-premises Microsoft Active Directory, host a standalone directory in the cloud, or proxy to an on-prem directory
- Lets you **use existing corporate credentials** to access AWS resources and sign in to AWS apps
- Enables **SSO to domain-joined EC2 instances** (Windows / Linux)
- Each directory is created in a **VPC** (2 subnets in different AZs for HA); reachable from on-prem via VPN / Direct Connect
- Integrates with directory-aware AWS services: WorkSpaces, WorkMail, WorkDocs, Chime, Connect, QuickSight, RDS for SQL Server, FSx for Windows File Server, IAM Identity Center, AWS Management Console

---

## AWS Managed Microsoft AD — Three Editions

Managed service that provides **AD domain controllers (DCs) running on Windows servers** in AWS.

- AWS creates a **minimum of two DCs** across two AZs for HA
- You can add additional DCs for HA and performance
- You have **exclusive access** to those DCs (administer via standard AD tools — RSAT, ADUC, etc.)
- Supports **Group Policy**, **schema extension**, **trusts**, **LDAPS**, **MFA via RADIUS**

### Standard Edition

- Up to **5,000 employees** / **30,000 directory objects**
- Small-to-midsize businesses
- Optimized cost

### Enterprise Edition

- Up to **500,000 directory objects**
- Large enterprises
- Higher cost; larger storage

### Hybrid Edition

- AWS Managed Microsoft AD DCs **join your existing on-prem AD forest** — they become part of your existing AD
- **Unified directory experience** — single forest, single set of users/groups across on-prem + AWS
- No need to set up forest trusts
- Best for "lift and extend" scenarios where you want AWS to act as additional DCs in your existing forest

> 💡 **API-driven edition upgrades**: upgrade from Standard → Enterprise via API without downtime. Downgrade is not supported.

**Extending AD on-prem (without Hybrid Edition)**: use **AD Trust** (forest trust over VPN / Direct Connect) if you want to keep separate forests but allow cross-forest authentication.

---

## Trust Types — Critical for SAP-C02

- **Forest trust** (recommended) — trust between two AD forests; fully supports Kerberos without caveats. Use this for AWS Managed Microsoft AD ↔ on-prem AD.
- **External trust** — trust between two domains in different forests; more limited than forest trust.

### One-way vs Two-way trust

| One-way trust                                                                              | Two-way trust                                                                                                                        |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| Users in trusted domain can access resources in trusting domain (not vice versa)           | Both sides can authenticate each other                                                                                               |
| Works with: **EC2, RDS for SQL Server, FSx for Windows File Server**                       | **Required for**: IAM Identity Center, WorkSpaces, WorkMail, WorkDocs, Chime, Connect, QuickSight, AWS Management Console federation |
| Common: AWS trusts on-prem (one-way outgoing from AWS); on-prem users access AWS resources | These services must look up users/groups in your on-prem AD                                                                          |

> ⚠️ **Exam pattern**: scenario says "WorkSpaces / Identity Center / WorkMail with on-prem AD users" → **two-way forest trust**. Scenario says "EC2 / RDS / FSx domain join with on-prem users" → **one-way trust** is sufficient.

---

## AD Connector

- **Directory gateway / proxy** to your existing on-premises AD
- **No directory data stored in AWS** — all authentication, LDAP, and Kerberos queries are forwarded to your on-prem DCs over VPN / Direct Connect
- AD Connector handles only the proxy — your on-prem AD remains the system of record
- **Two sizes**: Small (up to 500 users) and Large (up to 5,000 users)

### Use cases

- WorkSpaces, WorkDocs, QuickSight when you want to keep auth on-prem
- Apply existing on-prem AD security policies (password, MFA) to AWS workloads
- AWS Management Console federation via on-prem AD

### Limitations

- Does NOT support trusts (it's a proxy, not a directory)
- Does NOT support schema extensions
- Cannot host AD-aware AWS services that require a local directory (e.g., AWS Managed Microsoft AD-only features like Group Policy at the AWS DC level)
- Requires reliable, low-latency network to on-prem (VPN / DX)

> 💡 **Exam pick**: "use on-prem AD with AWS apps without storing any directory data in AWS" → **AD Connector**.

---

## Simple AD

- **Samba 4-based** standalone managed directory in AWS (NOT a full Microsoft AD)
- Basic AD features: user accounts, groups, group memberships, domain-joining EC2
- **Two sizes**: Small (≤500 users) and Large (≤5,000 users)
- **Easier to manage** for EC2 domain-joining and Linux workloads needing LDAP
- Does **NOT support trusts** (cannot join on-premises AD)
- Does NOT support: schema extensions, MFA, AD-aware AWS apps requiring full AD (e.g., Office 365, ADFS)

### Use cases

- Greenfield AWS-only deployment with simple directory needs
- Linux workloads needing LDAP
- Cost-conscious EC2 domain-join scenarios

> ⚠️ **Anti-pattern**: don't pick Simple AD for hybrid scenarios with on-prem AD — it can't trust. Use AWS Managed Microsoft AD or AD Connector instead.

---

## Amazon Cloud Directory

- **NOT an Active Directory** — despite the name, it's a different service
- **Hierarchical multi-schema cloud-native directory** for application data
- Stores hierarchical data (org charts, device registries, course catalogs, network topologies) with multiple schemas per directory
- Highly scalable: hundreds of millions of objects
- Used by application developers, NOT for user authentication
- Different from AD: no users / groups / computers, no Kerberos / LDAP, no domain join

**When to use**: building an app that needs a hierarchical data store (e.g., a multi-tenant SaaS with org hierarchies). When the scenario says "directory for users" → NOT Cloud Directory.

---

## Decision Matrix — Pick the Right Directory Service

| Scenario                                                            | Best Service                                                       |
| ------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Run full Microsoft AD in AWS, no on-prem                            | AWS Managed Microsoft AD (Standard or Enterprise)                  |
| Extend existing on-prem AD into AWS as additional DCs (same forest) | AWS Managed Microsoft AD (**Hybrid Edition**)                      |
| Keep on-prem AD as source of truth, no directory in AWS             | AD Connector                                                       |
| Simple AD-compatible directory in AWS, no on-prem                   | Simple AD                                                          |
| WorkSpaces / WorkMail / Identity Center with on-prem AD             | Managed Microsoft AD + **two-way forest trust**                    |
| EC2 / RDS domain join with on-prem AD users                         | Managed Microsoft AD + **one-way trust** OR AD Connector           |
| Application directory (hierarchical, non-user)                      | Amazon Cloud Directory                                             |
| Mobile / SaaS app sign-up + sign-in                                 | Amazon Cognito User Pools                                          |
| Federation hub for AWS console + apps                               | AWS IAM Identity Center (uses one of the above as identity source) |

---

## Integration with AWS Services

**AWS services that can use Directory Service**:

- **EC2** — Windows or Linux domain join; seamless via SSM
- **RDS for SQL Server, PostgreSQL, Oracle, MySQL** — Windows Authentication / Kerberos
- **FSx for Windows File Server** — file shares with AD permissions
- **WorkSpaces, WorkMail, WorkDocs, Chime, Connect, QuickSight** — user authentication
- **AWS IAM Identity Center** — use Directory Service as identity source
- **AWS Management Console** — federate console access via AD (via Identity Center or direct SAML)
- **License Manager** — track licensing by AD users
- **Amazon Q (Business)** — federate via Directory Service

### Cross-account sharing

- Share an AWS Managed Microsoft AD directory across accounts via **AWS RAM**
- Common pattern: centralize AD in a "shared services" account, share to workload accounts

---

## Backup, Monitoring, Maintenance

- **Automatic daily snapshots** of AWS Managed Microsoft AD
- Can take **manual snapshots** (up to 5 per directory)
- Restore is in-place (replaces the current directory state)
- **CloudWatch Logs integration** for security / event logs
- **CloudTrail** logs all Directory Service API calls
- AWS handles patching, OS updates, and DC software updates
- You manage AD-level configuration (users, groups, GPOs, schema, trusts)

---

## Exam Tips

- **AWS Managed Microsoft AD** is the most exam-relevant flavor — know the three editions (Standard 5K users, Enterprise 500K objects, Hybrid for same-forest extension)
- **Hybrid Edition** — when the scenario says "extend on-prem AD into AWS as part of the same forest with unified experience," it's Hybrid Edition (not a trust)
- **Two-way forest trust** is required for AWS apps that look up users from on-prem (Identity Center, WorkSpaces, WorkMail, WorkDocs, Chime, Connect, QuickSight, console federation)
- **One-way trust** is sufficient for EC2, RDS, FSx domain join
- **AD Connector** = proxy only; no directory data stored in AWS. Use when on-prem AD must remain system of record.
- **Simple AD** is Samba-based; no trusts, no schema extension, no MFA, no advanced AD features
- **Cloud Directory ≠ Active Directory** — it's a hierarchical data store, not for user auth
- Share AWS Managed Microsoft AD across accounts with **AWS RAM**
- **Domain controllers run in your VPC** — accessible via VPN / DX from on-prem
- **AWS handles HA**: minimum 2 DCs across AZs automatically
- AWS Managed Microsoft AD supports **MFA via RADIUS**, integrates with most third-party MFA providers
- For modern human federation, layer **AWS IAM Identity Center** on top of Directory Service (Identity Center can use Directory Service as its identity source)

---

## Exam Traps

- Simple AD is **Samba-based** with no trusts, no schema extension, and no MFA — if the scenario needs full AD features, the answer is AWS Managed Microsoft AD
- AD Connector **stores no directory data in AWS** — if the scenario requires the directory to be available when on-prem is down, AD Connector is wrong. Use AWS Managed Microsoft AD with trust
- Cloud Directory is a **hierarchical data store**, not a user directory — don't confuse it with Microsoft AD
- WorkSpaces, Identity Center, WorkMail, and similar services require a **two-way trust** — a one-way trust is insufficient because these services need to look up users in your AD
- AWS Managed Microsoft AD domain controllers are **managed instances** — administer via standard AD tools (RSAT, ADUC), not SSH / RDP. There is no OS-level access
- AWS Managed Microsoft AD is **Region-scoped after creation** — the Region cannot be changed. For multi-Region, use cross-Region replication
- Direct SAML 2.0 federation to the AWS Console is the **legacy approach** — AWS IAM Identity Center is the modern replacement
- AD Connector is a **proxy only** — it cannot host AWS apps that require a directory in AWS. Use AWS Managed Microsoft AD for those
- Simple AD has **limited service compatibility** for seamless domain join — check compatibility before choosing it
- Cognito User Pools is **app-level sign-in (B2C)** — it is not a directory service and lives outside Directory Service entirely
- AD Connector requires a **reliable, low-latency network** to on-prem — high latency or outages break authentication for AWS resources
- AWS Managed Microsoft AD editions can only be **upgraded** (Standard to Enterprise via API) — downgrade is not supported
