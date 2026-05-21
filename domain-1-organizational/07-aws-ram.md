# AWS RAM (Resource Access Manager)

> **Share AWS resources you own with other AWS accounts. Avoid duplication. Free service — you only pay for the underlying shared resources.**

Maps to: **Domain 1.4 — Design a multi-account AWS environment**

---

## Overview

- Share AWS resources with other AWS accounts, OUs, or entire AWS Organization
- Avoid resource duplication
- **Free service** — no additional cost; you only pay for the underlying shared resources
- Uses **resource shares** — a container defining the resources to share, the principals to share with, and the permissions granted
- Principals: AWS accounts, OUs, entire Organization, IAM roles, or IAM users (for supported types)

---

## Key Concepts

- **AWS Managed Permissions** — predefined permissions for each shareable resource type; applied by default
- **Customer Managed Permissions** — define exactly which actions principals can perform on shared resources (fine-grained access control)
- **Resource shares within Organizations** — when sharing is enabled in Organizations, sharing with accounts in the same org happens automatically without invitation acceptance
- **Resource shares with external accounts** — receiving account must accept an invitation before accessing shared resources
- **RetainSharingOnAccountLeaveOrganization** — parameter that maintains resource sharing continuity when accounts move between Organizations (mergers, acquisitions, restructuring)
- **Managed Prefix List** — set of CIDR blocks; simplifies security groups and route tables
- **Route 53 Outbound Resolver** — centrally managed resolver rules for hybrid DNS across many accounts/VPCs

---

## Integration with AWS Organizations

- Must **enable sharing with AWS Organizations** in the RAM console (or CLI/API) before sharing within the org
- Can share with entire org, specific OUs, or specific accounts
- When enabled, resource shares to org members don't require invitation acceptance
- SCPs can enforce consistent RAM sharing policies across the org
- CloudTrail logs all RAM API calls for audit and compliance

---

## Shareable Resources (Key Exam Services)

- **VPC Subnets** — multiple accounts launch resources in the same subnet (must be same Org; can't share security groups or default VPC)
- **AWS Transit Gateway** — share across accounts for hub-and-spoke topologies
- **Route 53** (Resolver Rules, DNS Firewall Rule Groups) — centralized hybrid DNS
- **License Manager Configurations**
- **Aurora DB Clusters** — cross-account read access
- **ACM Private Certificate Authority** — consistent cert issuance across accounts
- **CodeBuild Project**
- **EC2** — Dedicated Hosts, Capacity Reservation, Prefix Lists
- **AWS Glue** — Catalog, Database, Table (for cross-account ETL/analytics)
- **AWS Network Firewall Policies**
- **AWS Resource Groups**
- **Systems Manager Incident Manager** (Contacts, Response Plans)
- **AWS Outposts** (Outpost, Site) + **S3 on Outposts**
- **AWS App Mesh**
- **Amazon VPC Lattice** (Service, Service Network)
- **AWS Cloud WAN** (Core Network)
- **Amazon CloudFront** (Distribution tenancies)
- **AWS CloudHSM** (Clusters)
- **Amazon DataZone** (Domains)
- **Amazon Bedrock** (Custom models, Model invocation profiles)
- **AWS Billing and Cost Management** dashboards (Aug 2025)
- **IPAM Pools**
- **Image Builder** (Components, Images, Recipes)

---

## VPC Subnet Sharing — Detail

- Participants can manage **their own** resources in the shared subnet
- Participants **can't view, modify, or delete** resources belonging to other participants or the owner
- Enables centralized network management with decentralized workloads
- Owner pays for the VPC infrastructure; participants pay for their own resources
- Common pattern: shared services account owns the VPC; workload accounts launch into shared subnets

---

## Exam Tips

- RAM is the go-to answer when a question asks about sharing resources across accounts **without duplication**
- VPC subnet sharing via RAM is a common scenario: multiple accounts launch resources in the same subnet, centralizing network management while keeping workload ownership decentralized
- **RAM + Organizations** = no invitation needed; RAM with external accounts = invitation required
- **Transit Gateway sharing via RAM** is the standard multi-account networking pattern
- RAM is **free** — if a question asks about cost-effective resource sharing, RAM is usually the answer
- **Customer managed permissions** allow fine-grained access control on shared resources
- **IPAM pool sharing** via RAM is the centralized IP address management pattern across accounts

---

## Exam Traps

- RAM manages **resource sharing at scale with governance** — resource policies (S3 bucket policies, KMS key policies) grant cross-account access to individual resources; they are different mechanisms
- Lambda functions and SNS topics are **not shareable via RAM** — use resource-based policies for cross-account access to those services
- IAM roles and security groups are **not shareable via RAM** — each account manages its own security groups, even in shared subnets
- VPC subnet sharing requires accounts to be in the **same Organization** — not all resource types have this restriction, but subnets specifically do
- RAM shares **actual live resources** — AWS Service Catalog distributes CloudFormation templates/products; they serve different purposes
- RAM and License Manager are **different services** — License Manager tracks and enforces license rules; RAM can share License Manager configurations, but their purposes are distinct
