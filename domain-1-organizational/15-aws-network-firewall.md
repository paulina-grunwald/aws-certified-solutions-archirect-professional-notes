# AWS Network Firewall

> **Managed stateful firewall for VPCs. Suricata-compatible IPS/IDS, domain name filtering, TLS SNI inspection. Layer 3-7 inspection deployed in your VPC.**

Maps to: **Domain 1.1 — Architect network connectivity strategies** (traffic inspection)

---

## Overview

- Managed firewall and intrusion prevention service for **VPC traffic**
- **Stateful** — tracks connections across packets
- Supports **Suricata-compatible** IPS / IDS rules
- Operates at **layers 3-7**: IP, port, protocol, domain filtering, TLS SNI inspection
- Deployed in **its own subnet** within a VPC (firewall subnet)
- Highly available and auto-scaling — AWS manages capacity

---

## Architecture Patterns

### Centralized inspection VPC
- Single inspection VPC with Network Firewall
- All inter-VPC and egress traffic routed through it via Transit Gateway
- Spoke VPCs send traffic to inspection VPC via TGW route tables
- Most common SAP-C02 pattern

### Distributed (per-VPC) deployment
- Network Firewall in each VPC's firewall subnet
- More cost (one firewall per VPC) but lower latency
- Use when traffic must NOT leave the VPC for inspection

---

## Components

### Firewall
- The Network Firewall resource itself; deployed in a VPC
- Charged per hour + per GB processed

### Firewall Policy
- Top-level container — references rule groups + default actions
- Stateful default action: `pass`, `drop`, `alert`
- Stateless default action: `pass`, `drop`, `forward to stateful`

### Rule Groups
- **Stateless rule groups** — packet-level filters (5-tuple); no connection tracking
- **Stateful rule groups** — connection-tracking; Suricata syntax for full IPS rules

---

## Rule Types

### Stateless Rules
- 5-tuple match (source/dest IP, port, protocol)
- Pass / drop / forward to stateful

### Stateful Rules
- **Domain List** — allow / deny by domain name (HTTPS via TLS SNI, HTTP via Host header)
- **5-tuple** — same as stateless but with connection tracking
- **Suricata-compatible** — full Suricata rule syntax for IPS/IDS signatures

### TLS Inspection (since 2023)
- Decrypt TLS to inspect payload
- Requires ACM certificate
- Can be used with managed signatures for deep inspection

---

## Logging

### Alert logs
- Generated when a rule with `alert` action matches
- Sent to CloudWatch Logs, S3, or Kinesis Data Firehose

### Flow logs
- Detailed flow records of all traffic through the firewall
- Useful for compliance and forensics

### Flow Capture and Flow Flush (2025)
- **Capture** active flow metadata for monitoring
- **Flush** active flows to terminate them during security incidents

---

## Integration with Firewall Manager

- **AWS Firewall Manager** can centrally deploy Network Firewall policies across multiple accounts
- Useful for org-wide consistent firewall policies

---

## Use Cases

- **Domain filtering** — block malicious or unauthorized domains
- **IPS / IDS** — detect known attack signatures via Suricata rules
- **Centralized inspection** — single inspection VPC with TGW routing
- **Compliance** — PCI / HIPAA requiring inline IPS / IDS
- **Egress filtering** — control what your workloads can reach on the internet

---

## Exam Tips

- Network Firewall is the answer for: **"domain filtering", "IPS/IDS at L7", "centralized inspection VPC", "Suricata rules", "TLS inspection"**
- Deploy in **inspection VPC + TGW** for centralized scope; deploy per-VPC for low-latency / data-residency
- **Stateful for connection tracking + Suricata; stateless for simple 5-tuple filters**
- Integrates with **Firewall Manager** for org-wide deployment
- **TLS inspection** (2023) for inspecting encrypted traffic
- Combines with **CloudWatch Logs / S3 / Firehose** for alert and flow logging
- **Flow Capture + Flow Flush** (2025) for incident response — terminate suspicious flows

---

## Exam Traps

- Network Firewall operates at **layers 3-7 including domain and TLS** — Security Groups and NACLs are L3/L4 only and cannot do deep inspection
- Network Firewall is for **general VPC traffic inspection** — AWS WAF is specifically L7 HTTP/HTTPS for web apps (ALB, CloudFront, API GW); they are different services
- Network Firewall is the **data-plane firewall** — Firewall Manager is the orchestration layer that can manage Network Firewall rules across accounts
- Network Firewall requires a **dedicated firewall subnet** in the VPC — account for the /26 minimum subnet size when planning
- TLS inspection requires **your own certificates in ACM** — you cannot inspect arbitrary TLS traffic without managing the certificate trust chain
- Pricing is **per-AZ plus per-GB processed** — costs scale with the number of AZs deployed and traffic volume
- Network Firewall operates **only inside VPCs** — it cannot inspect edge traffic; use WAF or CloudFront-integrated rules for edge protection
