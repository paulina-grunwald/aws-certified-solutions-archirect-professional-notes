# AWS Systems Manager Parameter Store

> **Secure, hierarchical, versioned storage for configuration data and secrets. Capability of AWS Systems Manager. Free for the first 10,000 standard parameters per account per Region.**

Maps to: **Domain 1.2 — Prescribe security controls** (secrets / configuration management)

---

## Overview

- Located in **Systems Manager → Application Management → Parameter Store**
- Stores configuration data and secrets — passwords, DB connection strings, license codes, AMI IDs, feature flags
- Values can be **plain text** (`String` / `StringList`) or **encrypted with KMS** (`SecureString`)
- Reference parameters by **name** (and optionally a specific **version** or **label**)
- Integrates with EC2, ECS, Lambda, CloudFormation, CodeBuild / CodeDeploy / CodePipeline, EventBridge, CloudTrail, Config, IAM, KMS

---

## Parameter Types

- **String** — plaintext, single value
- **StringList** — comma-separated list of values
- **SecureString** — encrypted with KMS (use for passwords, secrets, license keys, tokens)

---

## Parameter Tiers

| Aspect | Standard | Advanced |
|---|---|---|
| **Cost** | Free for first 10,000 params/account/Region | $0.05/parameter/month |
| **Max value size** | 4 KB | 8 KB |
| **Max params** | 10,000 per account per Region | 100,000 |
| **Parameter policies** | Not supported | Supported (Expiration, ExpirationNotification, NoChangeNotification) |
| **SecureString encryption** | KMS direct | KMS envelope encryption (AWS Encryption SDK) |

> ⚠️ **Standard → Advanced is one-way.** You cannot revert to Standard (would truncate to 4 KB and drop policies).

**Higher API throughput** is an account-level setting (independent of tier) — raises sustained throughput up to **10,000 TPS** at a per-call charge.

---

## Hierarchies & Versioning

- Organize parameters with `/` paths up to **15 levels** deep (e.g. `/prod/app/db/password`)
- **GetParametersByPath** with `Recursive=true` fetches an entire subtree in one call
- IAM policies can be scoped by path prefix — grant ops `/prod/*` and devs `/dev/*`
- Every change creates a new immutable **version**; address with `name:version`
- **Labels** are mutable aliases that can be moved between versions — use them for `production` / `last-known-good` rollback patterns

---

## Parameter Policies (Advanced tier only)

- **Expiration** — automatically delete the parameter at a timestamp
- **ExpirationNotification** — emit EventBridge event N days before expiration
- **NoChangeNotification** — emit event if a parameter hasn't changed in N days (rotation / audit tracking)
- Max **10 policies per parameter**

---

## Public Parameters (AWS-managed)

AWS publishes read-only parameters under `/aws/service/...`:

- **Latest Amazon Linux 2023 AMI**: `/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64`
- ECS-optimized AMIs, EKS AMIs, RDS engine versions, AWS Region list, AWS service endpoints

**Use in CloudFormation / Launch Templates / Auto Scaling** to always pull current AMI IDs without hardcoding — common answer for "automatically use the latest AMI".

---

## Integrations

### CloudFormation dynamic references
- Plaintext: `{{resolve:ssm:param-name:version}}`
- SecureString: `{{resolve:ssm-secure:param-name:version}}`
- ⚠️ `ssm-secure` is **NOT allowed** in `AWS::CloudFormation::Init`, EC2 `UserData`, or custom resources

### Lambda Extension
- `AWS-Parameters-and-Secrets-Lambda-Extension` — in-process cache layer that reduces API calls and improves cold start
- Works for both Parameter Store and Secrets Manager

### EventBridge
- Emits `Parameter Store Change` events on Create / Update / Delete / LabelParameterVersion
- Drive custom rotation, pipelines, or notifications

### CloudTrail + Config
- CloudTrail logs every API call
- Config for compliance and drift detection

### KMS
- SecureString uses an AWS-managed key (`alias/aws/ssm`) by default, or any CMK you specify

### Cross-account / cross-Region
- **Resource-based policies are NOT supported** — share via IAM roles + `sts:AssumeRole`
- Replicate across Regions with EventBridge + Lambda or CloudFormation StackSets

---

## Parameter Store vs Secrets Manager

| Feature | Parameter Store | Secrets Manager |
|---|---|---|
| Cost | Std free (10K) / Adv $0.05/mo | $0.40/secret/mo + API calls |
| Max value | 4 KB Std / 8 KB Adv | 64 KB |
| Automatic rotation | No (build with EventBridge + Lambda) | Yes (native managed for RDS / Aurora / Redshift / DocumentDB) |
| Resource-based policy (cross-account) | No | Yes |
| Cross-Region replication | No (manual) | Yes (native) |
| Encryption | KMS (Std direct / Adv envelope) | KMS envelope |
| Best for | Config values, non-rotating secrets, AMI lookups, public params | DB creds, API keys, anything requiring **automatic rotation** or cross-account sharing |

---

## Exam Tips

- **Free for first 10,000 standard parameters** — pick Parameter Store over Secrets Manager when cost matters and rotation is NOT required
- Use `{{resolve:ssm:...}}` and `{{resolve:ssm-secure:...}}` in CloudFormation to avoid hardcoded secrets / AMIs
- Use **public parameters** (`/aws/service/...`) for "always latest AMI" patterns in Launch Templates / Auto Scaling / CloudFormation
- **Advanced tier required** for: > 4 KB values AND parameter policies (Expiration / ExpirationNotification / NoChangeNotification)
- **Hierarchies** enable path-prefix IAM grants — `/prod/*` to ops, `/dev/*` to devs
- **Versions are immutable; labels are movable** — use labels for blue/green and rollback patterns
- **EventBridge** integration enables custom **rotation pipelines** and audit triggers — typical answer when "rotate on a schedule" but mandates Parameter Store
- For cross-account secret sharing **with rotation** → **Secrets Manager** (resource-based policy + replication)
- Use the **Lambda Parameters and Secrets Extension** to cache values and reduce API throttling / cost

---

## Exam Traps

- Parameter Store has **no automatic secret rotation** — if automatic rotation of DB credentials is required, the answer is Secrets Manager
- Parameter Store has **no resource-based policies** — cross-account access must go through IAM roles + AssumeRole; "attach a resource policy to the parameter" is a Secrets Manager feature
- Parameter Store has **no native cross-Region replication** — for multi-Region config, use Secrets Manager replication or build with EventBridge + Lambda / CloudFormation StackSets
- Standard and Advanced SecureString use **different KMS call patterns** — Standard uses KMS directly; Advanced uses envelope encryption, which affects throttling behavior
- Advanced tier is **a one-way upgrade** — you cannot downgrade Advanced to Standard (truncation + policy loss would occur)
- Parameter policies are **Advanced-tier only** — you cannot set Expiration or notification policies on a Standard parameter
- EC2 UserData and `AWS::CloudFormation::Init` **do not support `ssm-secure` dynamic references** — use the instance's IAM role + `aws ssm get-parameter --with-decryption` at runtime instead
- Hierarchy max depth is **15 levels**, and parameter name length is limited to 1011 chars including path
- Parameter Store and AppConfig are **separate Systems Manager features** — AppConfig is for feature flags and validated/staged config rollout (with deployment strategies and rollback on CloudWatch alarm), not secret storage
- Default API throughput is **limited** — for high-TPS workloads enable Higher Throughput (10K TPS) and/or use the Lambda Parameters and Secrets Extension to cache
- SecureString **does not have versioned plaintext history** — unlike Secrets Manager, it cannot maintain staged rotation labels across secret versions
