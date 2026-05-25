# Podcast 18 — S3 Deep Dive

**Length target**: 15 min
**Repo references**: `domain-2-new-solutions/07-s3.md`

## Topic & Scope

S3 alone deserves a full podcast. Storage classes, lifecycle, replication, security models, performance, Glacier tiers. Almost every SAP-C02 question touches S3 somewhere.

## Service Coverage Depth

**Deep**:
- Storage classes (Standard, Intelligent-Tiering, Standard-IA, One Zone-IA, Glacier tiers)
- Lifecycle policies
- Replication (SRR, CRR, RTC)
- Encryption (SSE-S3, SSE-KMS, DSSE-KMS, SSE-C, CSE)
- Access controls (bucket policy, ACL, Access Points, block public access)
- Versioning, MFA Delete, Object Lock, Object Tagging

**Brief**:
- S3 Object Lambda
- S3 Multi-Region Access Points
- Event notifications + Inventory

## Structured Outline

1. **Open (30s)** — "S3 is everywhere. Storage classes, replication, encryption, lifecycle — let's nail every dimension."
2. **Storage Classes (3 min)** — Standard, Intelligent-Tiering (auto move based on access), Standard-IA, One Zone-IA, Glacier Instant Retrieval, Glacier Flexible Retrieval, Glacier Deep Archive
3. **Lifecycle Policies (1.5 min)** — transition rules between classes, expiration, versioning interaction
4. **Replication (2.5 min)** — SRR (same Region) vs CRR (cross-Region), Replication Time Control (15-min SLA), prefixes / tags / KMS support, ownership override
5. **Encryption (2.5 min)** — SSE-S3 (default), SSE-KMS (audit trail, key policy control), DSSE-KMS (double encryption), SSE-C (you provide key), CSE (client-side), Bucket Keys reduce KMS cost
6. **Access Control (2 min)** — block public access (account + bucket), bucket policy, ACLs (legacy), Access Points (per-app endpoints), Multi-Region Access Points
7. **Versioning + Object Lock + MFA Delete (1.5 min)** — versioning enables Object Lock (compliance + governance modes), MFA Delete for extra safety
8. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- Storage classes (cost vs retrieval): Standard > Standard-IA > One Zone-IA > Glacier IR > Glacier Flex > Glacier Deep Archive
- Intelligent-Tiering: small monitoring fee, auto-moves objects between Frequent / Infrequent / Archive Instant / Archive Access / Deep Archive based on access
- Standard-IA: 30-day minimum, 128 KB minimum chargeable size
- One Zone-IA: single-AZ cheaper, OK for re-creatable data
- Glacier IR: milliseconds retrieval, low-access archive
- Glacier Flexible: minutes to hours, batch retrieval (Bulk, Standard, Expedited)
- Glacier Deep Archive: 12 hours retrieval, cheapest
- Lifecycle policy: transitions + expirations; supports current and noncurrent versions
- Replication: SRR (same Region) or CRR (cross-Region); async; requires versioning on source + destination
- Replication Time Control (RTC): 15-min replication SLA + metrics — paid
- Replication can change storage class + ownership + KMS key on destination
- Encryption: SSE-S3 (default), SSE-KMS (audit + KMS key policy), DSSE-KMS (compliance), SSE-C (customer key), CSE (encrypt before upload)
- Bucket Keys reduce KMS API calls by 99% — recommended with SSE-KMS
- Access control priority: explicit deny > bucket policy / IAM > ACL
- Block Public Access: account-level setting; turn ON to prevent accidental public exposure
- Access Points: simplify access patterns per-app with own endpoint + policy
- Multi-Region Access Points: route requests to nearest replicated bucket
- S3 Object Lambda: transform objects on retrieval via Lambda (redact, resize, format-convert)
- Versioning + Object Lock: WORM (Write Once Read Many), Compliance vs Governance mode
- MFA Delete requires root user + MFA token

## Must-Mention Exam Traps

- EXAM TRAP: One Zone-IA is SINGLE-AZ — data loss if AZ fails (use for re-creatable data only)
- EXAM TRAP: Standard-IA has 30-day minimum + 128 KB minimum-billed size
- EXAM TRAP: Replication requires versioning on BOTH source and destination buckets
- EXAM TRAP: Replication is ASYNC — not immediate consistency for replicas
- EXAM TRAP: Bucket policy explicit deny ALWAYS wins — even over root account
- EXAM TRAP: ACLs are LEGACY — turn off if not needed (Object Ownership = "Bucket Owner Enforced")
- EXAM TRAP: SSE-KMS bucket key reduces per-request KMS API charges — enable for cost savings
- EXAM TRAP: SSE-C means YOU send the key on every request — AWS doesn't store it
- EXAM TRAP: Object Lock Compliance mode is NOT modifiable — not even by root; Governance mode allows admin override
- EXAM TRAP: MFA Delete is on the BUCKET — and only root can enable / disable / use it
- EXAM TRAP: Lifecycle transitions cost money + minimums apply (transitions to Glacier have minimum retention before next transition)
- EXAM TRAP: Pre-signed URLs grant time-limited access — works on private buckets without IAM
- EXAM TRAP: S3 strong consistency for read-after-write applies for ALL operations (since 2020+)
- EXAM TRAP: CloudFront + S3 needs OAC (replaces OAI) for restricting bucket to CloudFront-only
- EXAM TRAP: Cross-account replication needs replication role + destination bucket policy + (optional) ownership override

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| General-purpose hot data | S3 Standard |
| Unknown access pattern | S3 Intelligent-Tiering |
| Infrequent, multi-AZ resilience | Standard-IA |
| Cheap, re-creatable, single-AZ OK | One Zone-IA |
| Compliance archival, 12h OK | Glacier Deep Archive |
| Archive with millisecond access | Glacier Instant Retrieval |
| Cross-Region DR / latency | Cross-Region Replication |
| 15-min replication SLA | Replication Time Control |
| WORM compliance | Object Lock Compliance mode |
| Encrypt with KMS but reduce cost | SSE-KMS + Bucket Keys |
| Transform image on download | S3 Object Lambda |
| Per-app access policy | Access Points |
| Global low-latency reads | Multi-Region Access Points |
| Prevent accidental public exposure | Block Public Access (account + bucket) |

## Tone & Style

- Storage class progression "Standard → IA → Glacier" verbalized as a temperature gradient (hot → warm → cold)
- Encryption types listed with mnemonics
- Repeat "block public access" as the cardinal rule

## Rapid-Fire Closer

"Unknown access pattern?" — "Intelligent-Tiering." "Single-AZ S3?" — "One Zone-IA." "12-hour archival?" — "Glacier Deep Archive." "Replicate cross-Region?" — "Versioning on both." "WORM compliance?" — "Object Lock Compliance mode."
