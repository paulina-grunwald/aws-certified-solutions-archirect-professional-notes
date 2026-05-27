# AWS CloudHSM

> **Dedicated, single-tenant Hardware Security Module (HSM) in AWS. FIPS 140-3 Level 3 validated. You control the keys; AWS doesn't have access.**

Maps to: **Domain 1.2 — Prescribe security controls** (data protection)

---

## Overview

- **Single-tenant**, dedicated HSM (vs KMS which is multi-tenant)
- FIPS 140-3 Level 3 validated hardware
- **You manage your own keys** — AWS has no access to them (not even AWS engineers)
- Industry-standard APIs: PKCS#11, JCE, OpenSSL CNG/KSP, Microsoft CryptoNG
- Use cases: regulatory requirements for dedicated HSM, custom cryptographic operations, SSL/TLS offload, PKI, document signing

---

## When to Use CloudHSM vs KMS

| Need | Service |
|---|---|
| Standard AWS service encryption (S3, EBS, RDS, etc.) | **KMS** |
| AWS service encryption with controlled rotation + key policies | **KMS customer-managed keys** |
| FIPS 140-3 Level 3 **single-tenant** HSM | **CloudHSM** |
| Custom application using PKCS#11 / JCE / OpenSSL APIs | **CloudHSM** |
| SSL/TLS offload from web servers | **CloudHSM** |
| PKI Certificate Authority private keys | **CloudHSM** |
| Document/code signing keys outside AWS | **CloudHSM** |
| You want AWS to NOT have any access | **CloudHSM** |
| KMS but with dedicated HSM backing | **KMS Custom Key Store backed by CloudHSM** |

---

## Architecture

- Deploy a **CloudHSM cluster** in your VPC (across multiple AZs for HA)
- Each cluster has 1+ HSM instances
- Cluster auto-replicates keys and operations across instances
- Client SDK runs on your application servers; talks to HSMs via the cluster
- Access from VPC; can extend to on-prem via VPN/Direct Connect

---

## Cluster Setup Steps

1. Create CloudHSM cluster (specify VPC + subnets)
2. Add at least one HSM instance (start with 2 for HA)
3. Initialize the cluster (sign cluster certificate)
4. Activate the cluster (create initial admin user "PRECO")
5. Install client SDK on application servers
6. Create users with appropriate permissions

---

## User Types

- **Precrypto Officer (PRECO)** — initial setup user; becomes a CO after activation
- **Crypto Officer (CO)** — manages users and policies; cannot use keys for crypto ops
- **Crypto User (CU)** — performs cryptographic operations
- **Appliance User (AU)** — system user for cluster management (AWS-managed)

---

## Integration with KMS

### KMS Custom Key Store
- Use your CloudHSM cluster as the **backing key store for KMS keys**
- Combines KMS convenience + CloudHSM single-tenant HSM control
- Use case: AWS service encryption (S3, EBS, RDS) with dedicated HSM compliance

### CloudHSM as standalone
- Direct PKCS#11 / JCE / OpenSSL access from your applications
- AWS services that need KMS still need KMS (not direct CloudHSM)

---

## High Availability and Backup

- **Multi-AZ cluster**: minimum 2 HSMs in different AZs for HA
- Keys are **replicated** across HSMs in the cluster automatically
- **Automated daily backups** to AWS-managed S3 (encrypted, you can't download key material)
- Restore creates a new cluster from backup
- Manual backup via cluster snapshot

---

## Networking

- Cluster lives in **your VPC**
- HSMs get ENIs in your subnets
- Access via private IPs from VPC
- Cross-VPC access: VPC peering, Transit Gateway, PrivateLink
- On-premises access: VPN / Direct Connect to VPC

---

## CloudHSM via AWS RAM

- Share CloudHSM clusters across accounts using **AWS Resource Access Manager (RAM)**
- Useful for centralized HSM management in shared services account

---

## Pricing

- **Per-HSM-hour** charges (significantly more than KMS)
- Charges accrue as soon as HSM is in cluster
- No per-API charges (unlike KMS)
- For high-volume crypto workloads, CloudHSM may be **cheaper than KMS** at scale

---

## Compliance Use Cases

- **PCI DSS** — payment card key storage
- **HIPAA** — healthcare data encryption keys
- **FedRAMP High** — federal data with dedicated HSM
- **eIDAS** (EU) — qualified electronic signatures
- **GDPR** — encryption key control
- **Financial regulators** requiring "key in your possession"

---

## Exam Tips

- CloudHSM is the answer for: **"FIPS 140-3 Level 3"**, **"single-tenant HSM"**, **"AWS cannot access keys"**, **"PKCS#11 / JCE / OpenSSL APIs"**
- For **AWS service encryption** with HSM compliance: use **KMS Custom Key Store backed by CloudHSM**
- CloudHSM is in **your VPC** — apps access via VPC networking
- **Multi-AZ cluster** for HA — minimum 2 HSMs
- Share via **AWS RAM** for cross-account access
- **No per-API charges** — pay per HSM hour; can be cost-effective at high volume
- For **SSL/TLS offload** on web servers (Nginx, Apache, IIS) — CloudHSM via OpenSSL/CNG

---

## Exam Traps

- CloudHSM is a **dedicated single-tenant HSM you manage** — KMS is the managed key service that AWS services integrate with directly; they are different services
- AWS services use **KMS, not CloudHSM directly** for server-side encryption — use KMS Custom Key Store to back KMS keys with your CloudHSM cluster
- Cluster credentials are **irrecoverable by AWS** — if you lose your cluster credentials, your keys are lost permanently
- CloudHSM requires **significant operational effort** — you manage users, client SDK installation, and certificate management yourself
- Custom Key Store availability **depends on the CloudHSM cluster** — a cluster outage means KMS operations fail for any keys backed by that cluster
- Backups are **encrypted with AWS-managed keys** — you can restore a cluster from backup but cannot extract raw key material
- CloudHSM is **not multi-Region** — for cross-Region you need separate clusters and your own key replication strategy
- CloudHSM Classic is **deprecated** — current version is CloudHSM v2; old exam material may reference Classic
