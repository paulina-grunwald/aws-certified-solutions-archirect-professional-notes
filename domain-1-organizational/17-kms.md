# AWS KMS (Key Management Service)

> **Managed service to create and control encryption keys used by AWS services and your applications. Integrated with 100+ AWS services. Backed by FIPS 140-3 Level 3 validated HSMs.**

Maps to: **Domain 1.2 — Prescribe security controls** (data protection)

---

## Overview

- Centralized **encryption key management** across AWS services
- Backed by **FIPS 140-3 Level 3** validated HSMs (multi-tenant shared HSM service)
- Integrated with 100+ AWS services for SSE (S3, EBS, RDS, DynamoDB, Lambda, SQS, etc.)
- All key operations are **logged in CloudTrail** — full audit trail
- **Per-Region service** — keys are region-scoped
- Use cases: encrypt at rest, encrypt envelopes, sign/verify, generate data keys

---

## Key Types

### Symmetric Keys (default, AES-256)
- One key for both encrypt and decrypt
- Used for most data encryption use cases
- The key never leaves KMS (you call KMS API to encrypt/decrypt)

### Asymmetric Keys (RSA, ECC)
- Public/private key pair
- **Public key** can be downloaded; **private key** stays in KMS
- Use cases: digital signing, encryption with public key by external parties

### HMAC Keys
- For generating HMACs (Hash-based Message Authentication Codes)
- Symmetric secret used for message integrity

### Key Specs
- **SYMMETRIC_DEFAULT** (AES-256, default)
- **RSA_2048**, **RSA_3072**, **RSA_4096**
- **ECC_NIST_P256**, **ECC_NIST_P384**, **ECC_NIST_P521**, **ECC_SECG_P256K1**
- **HMAC_224** through **HMAC_512**

---

## Key Material Origin

- **AWS_KMS** (default) — KMS creates and manages the key material
- **External** — bring your own key (BYOK); import key material from on-prem
- **AWS_CLOUDHSM** — key material in your CloudHSM cluster (custom key store)
- **EXTERNAL_KEY_STORE (XKS)** — key material in your **external HSM outside AWS** (data sovereignty)

---

## AWS-Managed vs Customer-Managed vs AWS-Owned

| Type | Created by | Visible in your account | Rotation | Cost |
|---|---|---|---|---|
| **AWS-owned keys** | AWS | NO | AWS-managed | Free |
| **AWS-managed keys** (`aws/*`) | AWS, on first use of a service | YES (can view, can't manage) | **Mandatory annual rotation** | Free |
| **Customer-managed keys (CMKs)** | You | YES | **Optional rotation** (annual or custom interval) | $1/key/month + API usage |

> 💡 **Customer-managed keys** are the answer when you need: custom key policies, rotation control, cross-account sharing, audit trail of who used the key.

---

## Key Rotation

- **Automatic rotation** (annual, or custom interval)
- Old key material retained for decrypting older ciphertexts
- KMS handles version transparently — applications don't need to know
- **AWS-managed keys** — mandatory annual rotation (uncontrollable)
- **Customer-managed keys** — opt-in; configurable from **90 days to 7 years**

---

## Key Policies

Every KMS key has a **key policy** — a resource-based policy that's the primary access control mechanism.

- **Default key policy** — gives root user full permissions (and the root user can delegate to IAM)
- **Without a key policy that grants access, even root and admin users cannot use the key**
- Combine with IAM policies for layered access control

### Cross-account access
- Key policy grants permissions to another account (or specific principals)
- Recipient account's IAM policy must also grant permission to use the key
- Common pattern for cross-account S3 + KMS scenarios

### Grants
- Temporary alternative to key policy modifications
- Programmatic, fine-grained, revocable
- Useful when a service needs temporary use of a key (e.g., RDS during a copy operation)

---

## Envelope Encryption

For large data (> 4 KB), KMS uses envelope encryption:

1. Application requests a **data key** from KMS via `GenerateDataKey`
2. KMS returns plaintext data key + ciphertext data key
3. Application uses plaintext data key to encrypt the data locally (AES-256 in app)
4. Application stores ciphertext data key alongside the encrypted data
5. To decrypt: send ciphertext data key to KMS, get plaintext data key back, decrypt data locally

**Why**: KMS has a 4 KB direct-encryption limit. Envelope encryption keeps the data plane local while keeping the key plane in KMS.

### Encryption SDK
- AWS Encryption SDK does envelope encryption automatically
- Available in multiple languages
- Handles key caching, versioning, and algorithm choices

---

## Key Stores

### Default key store (AWS-managed HSMs)
- Multi-tenant FIPS 140-3 Level 3 HSMs operated by AWS

### Custom Key Store with AWS CloudHSM
- Your CMKs use **your CloudHSM cluster** for key material
- Combine the convenience of KMS APIs with single-tenant HSM control
- Use case: regulatory requirement for dedicated HSM (e.g., FIPS 140-2 Level 3 specific)

### External Key Store (XKS)
- KMS calls **your external HSM outside AWS** for cryptographic operations
- Total **separation of key material from AWS infrastructure**
- Use case: extreme data sovereignty / regulatory requirements (financial, government)
- You manage availability of the external HSM (KMS can't operate without it)

---

## Cross-Region Considerations

- Keys are **regional** by default
- **Multi-Region keys** — replicated to multiple Regions with the same key ID and key material
- Use cases: disaster recovery with cross-Region encrypted data, global active-active applications
- Each replica behaves as an independent key (each Region has its own policy)
- Replication is **opt-in** per Region

---

## Encryption Context (additional authenticated data)

- Key-value pairs passed to `Encrypt`/`Decrypt` that act as integrity check
- NOT secret; included in CloudTrail logs
- Must match EXACTLY between encrypt and decrypt — otherwise decrypt fails
- Use for: cryptographic binding to a request context (e.g., `aws:Source=arn:aws:s3:::bucket`)

---

## Pricing

- **$1/key/month** for each customer-managed key
- **$0.03 per 10,000 API requests** (encrypt, decrypt, sign, verify, generate data key)
- **Free for AWS-managed keys** (`aws/*`)
- **External Key Store + Custom Key Store** — additional charges
- **Multi-Region keys** charged per replica

---

## Common Service Integrations

- **S3** — SSE-KMS, SSE-S3 (default), DSSE-KMS (dual-layer)
- **EBS** — encrypted volumes (default)
- **RDS / Aurora** — encrypted databases at rest
- **DynamoDB** — server-side encryption (default + custom KMS keys)
- **Lambda** — environment variable encryption, code signing
- **SQS / SNS** — message encryption at rest
- **CloudWatch Logs** — log group encryption
- **Secrets Manager** — secret encryption
- **Systems Manager Parameter Store** — SecureString parameters

---

## Exam Tips

- **AWS-managed keys** = free, mandatory annual rotation, can't customize policy → quick wins
- **Customer-managed keys** = $1/key/month, optional rotation, full policy control → flexibility + audit
- **Custom Key Store with CloudHSM** = use your CloudHSM cluster for KMS keys → dedicated HSM compliance
- **External Key Store (XKS)** = key material in your HSM outside AWS → maximum sovereignty
- **Envelope encryption** when data > 4 KB — use **GenerateDataKey** + local encryption
- **Multi-Region keys** for cross-Region active-active apps (same key ID, same material)
- **S3 Bucket Keys** reduce KMS API calls ~99% — cost optimization for SSE-KMS
- **Encryption Context** is integrity binding, not secrecy — logged in CloudTrail
- **Key policy is the source of truth** — without it, even root has no access
- **Key rotation is automatic and transparent** — old ciphertexts remain decryptable
- **Cross-account KMS access** requires BOTH key policy + caller's IAM policy
- For **deletion**: KMS keys have a **7-30 day waiting period** before permanent deletion (cancel-able)

---

## Exam Traps

- KMS key policy is the **primary access control mechanism** — without a key policy granting access, even the root user cannot use the key; IAM alone is insufficient
- KMS keys are **regional** — for cross-Region encrypted data use Multi-Region keys or re-encrypt at the destination
- SSE-KMS and SSE-S3 are **different encryption options** — SSE-S3 uses AWS-owned keys (no audit, no per-bucket key); SSE-KMS uses your KMS key (audit, control, costs)
- KMS has a **4 KB direct encryption limit** — for larger data you must use envelope encryption via GenerateDataKey
- Asymmetric keys can encrypt **at most ~190 bytes** (RSA 2048) — use symmetric keys or envelope encryption for data payloads
- Custom Key Store requires an **operational CloudHSM cluster** — a cluster outage stops KMS operations for those keys
- External Key Store availability is **your responsibility** — KMS depends on it being reachable; if it goes down, those keys are unusable
- AWS-managed keys are **not deletable** — only customer-managed keys can be scheduled for deletion
- AWS-managed key rotation is **annual and uncontrollable** — for custom rotation intervals you must use customer-managed keys
- Imported key material has an **expiration date** — if not refreshed before expiry, the key becomes unusable
- KMS has **API rate limits** — high-throughput workloads should use S3 Bucket Keys, data key caching, or envelope encryption to reduce API calls
- CloudHSM and KMS are **different services** — CloudHSM is your own dedicated HSM; KMS is AWS-managed key management that can optionally use CloudHSM as a backend
