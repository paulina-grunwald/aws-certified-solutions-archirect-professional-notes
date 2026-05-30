# AWS CloudFormation

> **Infrastructure as Code (IaC) service. Provision AWS resources declaratively with JSON / YAML templates. Resources managed as stacks. StackSets for multi-account / multi-Region. Change Sets for preview. Drift detection. Custom Resources for non-AWS provisioning. Stack Refactoring to reorganize resources without import/export.**

Maps to: **Domain 2.1 — Deployment strategies (IaC)**, **Domain 1.4 — Multi-account environments**, **Domain 4.2 — Migration**

---

## Overview

- Declarative IaC for AWS — JSON or YAML templates
- Resources managed as **stacks** (logical units)
- **Free** service — pay only for the underlying resources
- Automatic **rollback on failure**
- Built-in **drift detection**
- Integrates with **AWS Organizations, Service Catalog, Control Tower, CodePipeline**

### Key Concepts

- **Template** — JSON/YAML blueprint
- **Stack** — single unit of related resources managed together
- **Change Set** — preview of proposed changes
- **Stack Policy** — IAM-like policy protecting stack resources from unintended updates
- **StackSets** — deploy stacks across accounts + Regions

---

## Template Anatomy

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: String
Metadata:    # Additional info about the template
Parameters:  # Input values at stack creation/update time
Mappings:    # Key-value lookup tables (static)
Conditions:  # Conditional resource creation
Transform:   # Macros (e.g., AWS::Serverless-2016-10-31)
Resources:   # REQUIRED — AWS resources to provision
Outputs:     # Values to export or return
```

**Parameters** support: String, Number, List, CommaDelimitedList, AWS-Specific (e.g., `AWS::EC2::KeyPair::KeyName`). Max **200 parameters** per template.

**Intrinsic Functions**: `!Ref`, `!GetAtt`, `!Sub`, `!Join`, `!Select`, `!Split`, `!If`, `!Condition`, `Fn::ImportValue`, `Fn::FindInMap`, `Fn::GetAZs`, `Fn::Cidr`, `Fn::ToJsonString`

---

## StackSets (Multi-Account / Multi-Region)

Extend stacks across **multiple accounts and Regions** in a single operation.

- **Administrator Account** — where StackSets are created
- **Target Accounts** — where stack instances deploy
- **Stack Instances** — reference to a stack in target account/Region

### Deployment Permissions

- **Self-managed** — manually create `AWSCloudFormationStackSetAdministrationRole` (admin) + `AWSCloudFormationStackSetExecutionRole` (targets)
- **Service-managed** — uses **AWS Organizations**; auto-deploy to new accounts via OUs

### Deployment Configuration

- `MaxConcurrentCount` / `MaxConcurrentPercentage` — parallel account deployments
- `FailureToleranceCount` / `FailureTolerancePercentage` — failures allowed before stop
- **Region ordering** — deploy to Regions in sequence
- **Concurrency mode** — `STRICT_FAILURE_TOLERANCE` (default) or `SOFT_FAILURE_TOLERANCE`
- **Account Gate (Lambda)** — pre-deployment approval/reject check

### Organizations Integration

- **Delegated administrator** support — designate a member account
- **Automatic deployment to new accounts** joining an OU
- Target OUs directly (no need to list accounts)

---

## Drift Detection

- Identifies resources whose actual configuration differs from CloudFormation-managed configuration
- Status: `IN_SYNC`, `MODIFIED`, `DELETED`, `NOT_CHECKED`
- **Limitations**:
  - Not all resource types/properties support drift detection
  - Only detects on resources managed by CloudFormation
  - Does not detect drift on resources in nested stacks (check each separately)
- **Drift-Aware Change Sets** — show drift status alongside proposed changes
- **Import drifted resources** to bring back under CloudFormation management

---

## Change Sets

- **Preview proposed changes** before execution
- Types: **Create**, **Update**, **Import**
- **Template Validation During Change Set Creation**:
  - Validates invalid property syntax
  - Checks for resource name conflicts
  - Validates S3 bucket emptiness on delete operations
  - Catches errors before provisioning begins
- Only one change set can be executed at a time per stack

---

## Nested Stacks

- Stacks within stacks via `AWS::CloudFormation::Stack` resource
- **Use cases**: reuse patterns (VPC, SGs), overcome 500-resource limit, isolate lifecycle
- Updates to nested stacks must be initiated from the parent (root) stack
- Each nested stack has its own change set preview
- Deleting parent deletes all nested stacks

### Nested Stacks vs Cross-Stack References

| Pattern | Use case |
|---|---|
| **Nested stacks** | Component reuse, tightly coupled lifecycle |
| **Cross-stack references** | Share outputs between independent stacks (Export/ImportValue) |

---

## Cross-Stack References

- Stack A exports a value in `Outputs` with `Export: Name`
- Stack B imports with `Fn::ImportValue`
- **Rules**:
  - Export names unique within Region per account
  - Cannot delete a stack with exports referenced elsewhere
  - Cannot modify/remove an exported value if imported
  - **Same Region only** — no cross-Region

---

## DeletionPolicy & UpdateReplacePolicy

| Policy | Behavior |
|---|---|
| **Delete** (default for most) | Resource deleted with stack |
| **Retain** | Resource kept; no longer managed by CloudFormation |
| **Snapshot** | Final snapshot taken before deletion (EBS, RDS, Redshift, Neptune, ElastiCache, DocumentDB) |

**UpdateReplacePolicy** applies when a resource is replaced during update.

> **Default for `AWS::RDS::DBCluster` and `AWS::RDS::DBInstance` is `Snapshot` (not Delete)**.

---

## Custom Resources

- Extend CloudFormation with custom provisioning logic via Lambda or SNS
- Types: `AWS::CloudFormation::CustomResource`, `Custom::MyResourceName`
- **Lifecycle**: Create / Update / Delete request → custom logic → must send response to **pre-signed S3 URL** (SUCCESS / FAILED)
- **If no response, CloudFormation waits until timeout (up to 1 hour)**

### Use Cases

- Provision non-AWS resources (third-party APIs)
- Custom validation / lookup during stack ops
- Fetch AMI IDs dynamically
- Empty S3 buckets before deletion
- Manage resources in other accounts

---

## cfn-init, cfn-signal, cfn-hup

Helper scripts for EC2 bootstrapping + config.

- **cfn-init** — reads `AWS::CloudFormation::Init` metadata; configures packages, files, services, commands, users, groups; idempotent
- **cfn-signal** — signals CloudFormation that a resource (EC2) is ready; works with `CreationPolicy` or `WaitCondition`
- **cfn-hup** — daemon that detects metadata changes and triggers re-config; supports in-place updates

### CreationPolicy vs WaitCondition

- **CreationPolicy** — attached directly to a resource (ASG, EC2); simpler
- **WaitCondition** — separate resource; can wait for signals from outside the stack

---

## Resource Import

1. Add resource to template with `DeletionPolicy: Retain`
2. Create an **import change set**
3. Provide the resource identifier (physical ID)
4. Execute the change set

- Resource cannot already be managed by another stack
- Not all resource types support import
- Resource configuration in template must match actual

### Stack Refactoring

- Reorganize resources across stacks without manual import/export
- Simplifies splitting / merging stacks

---

## Stack Policies & Protection

- **Stack Policy** — JSON document controlling which resources can be updated
  - Default: all resources updatable
  - Once applied, protects ALL by default — must explicitly allow updates
  - Can temporarily override during update with `--stack-policy-during-update-body`
  - **Cannot be removed once applied** — only modified
- **Termination Protection** — prevents accidental stack deletion; must be explicitly disabled
- **Stack-level permissions** — IAM controls who can call CloudFormation; **service role** can have higher resource-creation permissions (separation of concerns)

---

## CloudFormation Registry & Modules

- **Registry** — central repository for resource types, modules, hooks
  - Public extensions (AWS + third-party)
  - Private extensions (your org)
  - **Hooks** — run validation logic before/after resource provisioning
- **Modules** — reusable, encapsulated CloudFormation building blocks
- **CloudFormation Guard (cfn-guard)** — policy-as-code validation for templates

---

## CloudFormation vs CDK vs Terraform

| Feature | CloudFormation | CDK | Terraform |
|---|---|---|---|
| Language | YAML / JSON | TypeScript, Python, Java, C#, Go | HCL |
| State | Managed by AWS | Synthesizes to CFN | State file (local or remote) |
| Provider | AWS only | AWS only (via CFN) | Multi-cloud |
| Drift Detection | Built-in | Via CFN | `terraform plan` |
| Modularity | Nested stacks, modules | Constructs (L1/L2/L3) | Modules |
| Rollback | Automatic | Via CFN | Manual |
| Cost | Free | Free | Free (OSS) / paid (Cloud) |

**Choose CloudFormation** for AWS-only with tight AWS integration (StackSets, Service Catalog).
**Choose CDK** for familiar programming languages with CloudFormation backend.
**Choose Terraform** for multi-cloud / hybrid.

---

## Recent Updates

- **Stack Refactoring** — reorganize resources across stacks without import/export
- **Template Validation in Change Sets** — early error detection before provisioning
- **CloudFormation Language Server** — IDE integration (VS Code, Kiro) with context-aware auto-complete
- **Drift-Aware Change Sets** — show drift status alongside proposed changes
- **Infrastructure as Code MCP Server** — AI-assisted IaC development tooling
- **Improved StackSets** — better concurrency controls, delegated admin enhancements
- **CloudFormation Guard** — policy-as-code validation for templates

---

## Exam Tips

- **StackSets + Organizations + service-managed permissions** = auto-deploy to new accounts in OUs (no manual IAM role setup)
- **Cross-stack references** = same Region only; cannot delete exporting stack while imported
- **DeletionPolicy: Snapshot** is default for RDS (not Delete) — know which services support Snapshot
- **Custom resources MUST send a response** to the pre-signed S3 URL or stack hangs until timeout
- **Change sets do NOT make changes** — they preview; you must explicitly execute
- **cfn-signal + CreationPolicy** = wait for EC2 bootstrap before CREATE_COMPLETE
- **Nested stacks** for component reuse; **cross-stack references** for sharing between independent stacks
- **Stack Policy once applied cannot be removed** — only made more permissive
- **Resource Import** requires `DeletionPolicy: Retain` and config matching actual
- **StackSets MaxConcurrentCount** controls parallelism; **FailureToleranceCount** controls when to stop
- "Multi-cloud IaC" → **Terraform**; "AWS-only with programming languages" → **CDK**
- **Service role** separates user permissions from resource-creation permissions (least privilege)
- **Stack Refactoring** = reorganize resources across stacks without manual import/export

---

## Exam Traps

- **Cross-stack references do NOT work cross-Region** — for multi-Region use StackSets
- **Drift detection does NOT automatically remediate drift** — it only reports
- **Nested stack updates MUST be initiated from the root stack** — updating a nested stack directly causes issues
- **DeletionPolicy: Retain orphans the resource** — kept but no longer managed by CloudFormation
- **StackSets with self-managed permissions** still require manual IAM role creation in every target account
- **WaitCondition is NOT the same as CreationPolicy** — WaitCondition is a separate resource, CreationPolicy is an attribute
- **`Fn::ImportValue` cannot use `Ref` or `GetAtt`** for the import name — static string or `Fn::Sub` only
- **CloudFormation does NOT support all AWS resource types** — check registry; some need custom resources
- **Rollback on failure is default** — if disabled and stack fails, enters `UPDATE_ROLLBACK_FAILED` requiring manual intervention
- **Stack Policy protects against updates, NOT deletion** — use **Termination Protection** for deletion prevention
