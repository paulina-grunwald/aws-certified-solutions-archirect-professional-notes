# AWS Firewall Manager

> **Centrally configure and manage firewall rules across multiple accounts and resources in AWS Organizations. One pane of glass for WAF, Shield, Network Firewall, Security Groups, Route 53 Resolver DNS Firewall.**

Maps to: **Domain 1.2 — Prescribe security controls** (centralized security)

---

## Overview

- Centrally manage **firewall rules across all accounts** in AWS Organizations
- Built on top of **AWS WAF, Shield Advanced, Network Firewall, Route 53 Resolver DNS Firewall, Security Groups**
- Automatically apply policies to **new resources** as they're created
- Compliance reporting — flags non-compliant resources
- Requires **AWS Organizations** with all features enabled
- Requires **delegated admin** account for FMS (best practice: dedicated security tooling account)

---

## Policy Types

### WAF Policies
- Apply common WAF rule groups (managed rules, custom rules) to all ALBs / API Gateways / CloudFront / AppSync / Cognito User Pools
- Auto-attach Web ACLs to new resources matching criteria

### Shield Advanced Policies
- Apply Shield Advanced protection to specified resources (CloudFront, ALB, NLB, Route 53, EIPs)
- Centralizes DDoS protection configuration

### Network Firewall Policies
- Centrally deploy and manage AWS Network Firewall rules across VPCs and accounts
- Useful for inspection VPC patterns

### DNS Firewall Policies
- Centrally manage Route 53 Resolver DNS Firewall rule groups across accounts

### Security Group Policies
- Three modes:
  - **Common policy** — distribute a security group to many accounts
  - **Content audit policy** — audit and remediate non-compliant security group rules
  - **Usage audit policy** — find unused security groups / unused rules

### Third-Party Firewall Policies
- Manage Palo Alto Networks Cloud NGFW, Fortinet, Check Point integration via Firewall Manager

---

## Architecture

1. **Designate a Firewall Manager admin account** (a member account, NOT management — best practice for security)
2. Define **Firewall Manager policies** in the admin account
3. Policies automatically apply across the org based on **policy scope** (entire org, OUs, account tags, account IDs)
4. New resources matching scope are automatically protected
5. **Compliance dashboard** shows resources in / out of compliance

---

## Integration with Security Hub

- Firewall Manager findings (non-compliant resources) sent to **Security Hub**
- Combined view of WAF / Shield / Network Firewall / DNS Firewall compliance alongside other security signals

---

## Use Cases

- **Compliance**: enforce that every public ALB / CloudFront has WAF with managed rules
- **Standardization**: apply baseline security group rules across all VPCs in the org
- **DDoS protection**: ensure Shield Advanced on all internet-facing resources
- **Zero trust networking**: enforce Network Firewall rules across all VPCs from a central inspection account
- **DNS hygiene**: block malicious domains org-wide via DNS Firewall

---

## Exam Tips

- Firewall Manager is the answer for **"centrally manage WAF/Shield/SGs across multiple accounts and Regions"**
- Requires **AWS Organizations** with all features enabled
- Use a **dedicated FMS admin account** (delegated administrator) — best practice
- Auto-applies policies to **newly created resources** matching scope
- **Three security group modes**: common (distribute), content audit (remediate), usage audit (find unused)
- Combines well with **Security Hub** for centralized compliance findings
- For DDoS protection scenarios → **Firewall Manager + Shield Advanced** policy
- For "ensure every ALB has WAF" → Firewall Manager **WAF policy**

---

## Exam Traps

- Firewall Manager is the **orchestration layer, not the firewall itself** — Network Firewall is the actual data-plane firewall service; FMS manages its rules across accounts
- Firewall Manager requires **AWS Organizations with all features enabled** — it will not work in standalone accounts
- Firewall Manager admin should be a **delegated admin account, not the management account** — placing it in the management account is an anti-pattern
- FMS policies are **regional** — WAF / Shield / Network Firewall / DNS Firewall must be available in each Region where the policy is scoped
- FMS **centralizes configuration but does not replace** the individual services (WAF, Shield, etc.) — those services still run independently
- FMS costs **$100/month per Region per policy** — plus the underlying service costs (WAF, Shield, etc.)
