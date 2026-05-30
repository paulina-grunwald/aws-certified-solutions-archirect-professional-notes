# Other Niche SAP-C02 Services (Consolidated Reference)

> **One-page reference for low-frequency in-scope services that appear as distractors but rarely as the correct answer in SAP-C02: Amazon Managed Blockchain, Amazon Elastic Transcoder, AWS IoT Things Graph. Memorize each one's "pick when" line — almost always there's a better-matching service.**

Maps to: **Domain 2 — multiple subdomains**, **Domain 4.3 — Modernization (distractor coverage)**

---

## Amazon Managed Blockchain

### Overview
- **Managed blockchain networks**: Hyperledger Fabric + Ethereum
- Membership-based (Hyperledger) — invite peers via AWS account
- AWS handles node provisioning, certificate authority, ordering service, scaling
- Use case: multi-party transaction ledger, supply chain provenance, financial settlement

### When to Pick
- "Multi-party shared ledger with cryptographic trust" → **Managed Blockchain**
- "Hyperledger Fabric network without operating it" → **Managed Blockchain**
- "Build on Ethereum mainnet / private chains" → **Managed Blockchain** (Ethereum support)

### When NOT to Pick
- Single-party audit trail → **QLDB** (Quantum Ledger Database — append-only immutable log) — but QLDB is being phased out by 2025 (no new customers); use DynamoDB with versioning
- Cross-account audit log → **CloudTrail + S3 Object Lock**
- Simple transactional database → **RDS / DynamoDB**

### Limitations
- Operating a Hyperledger Fabric network requires blockchain expertise
- Performance: lower throughput than traditional databases
- Cost: per-node hourly + data transfer
- **Often a distractor** in SAP-C02 unless the question specifically mentions blockchain or shared multi-party ledger

---

## Amazon Elastic Transcoder

### Overview
- **Media transcoding** service: convert video files between formats (MP4 ↔ HLS / MOV / WebM)
- Pre-defined output presets (mobile, web, HD)
- Pipelines + jobs model
- S3 input + output

### When to Pick
- "Convert uploaded video to multiple resolutions for streaming" → **Elastic Transcoder** (or **MediaConvert**)

### When NOT to Pick
- **AWS now prefers MediaConvert** for new workloads — more features, lower cost
- **Elastic Transcoder is legacy** — still in scope for SAP-C02 but rarely the best answer
- Live streaming → **MediaLive / MediaPackage**
- Origin / packaging → **MediaPackage**
- Storage of media assets → **MediaStore** (or S3)

### Elastic Transcoder vs MediaConvert

| | Elastic Transcoder | MediaConvert |
|---|---|---|
| Status | Legacy (still supported) | **Current recommended** |
| Pricing | Per-minute output | Per-minute output (more granular) |
| Features | Basic transcoding | Broadcast features (HDR, captions, ad markers) |
| When | Existing workloads | New workloads |

**For SAP-C02**: If both Elastic Transcoder and MediaConvert appear as options for transcoding, MediaConvert is usually correct. Elastic Transcoder is a distractor.

---

## AWS IoT Things Graph

### Overview
- **Visual designer + runtime** for IoT workflows across devices + cloud
- Connect heterogeneous devices using pre-built or custom models
- Drag-and-drop workflow builder

### Status
- **Deprecated** — limited new functionality
- Still listed in SAP-C02 in-scope services
- Use **AWS IoT Greengrass** (edge compute) or **AWS IoT Core** (cloud routing) + **AWS IoT Events** (rules) instead

### When to Pick (Rare)
- Almost never on SAP-C02 — distractor option
- If forced: "visual designer for IoT device workflow"

### When NOT to Pick
- IoT edge processing → **AWS IoT Greengrass**
- IoT rules engine → **AWS IoT Core Rules**
- IoT event detection → **AWS IoT Events** (also deprecated May 2026)
- Industrial IoT → **AWS IoT SiteWise**
- Digital twin → **AWS IoT TwinMaker**

---

## Quick "Pick When" Reference

| Hint in question | Service |
|---|---|
| Hyperledger Fabric / Ethereum / multi-party ledger | **Managed Blockchain** |
| Single-party append-only audit log | **QLDB** (legacy; consider DynamoDB + versioning) |
| Video file format conversion | **MediaConvert** (preferred) or **Elastic Transcoder** (legacy) |
| Live video streaming | **MediaLive + MediaPackage** |
| Visual IoT workflow designer | **IoT Things Graph** (rare — usually distractor) |

---

## Exam Tips

- Managed Blockchain = **Hyperledger Fabric + Ethereum**, managed
- Elastic Transcoder = **legacy**; MediaConvert is the modern answer for video transcoding
- IoT Things Graph = **deprecated** — rarely the right answer
- All three are **in-scope** for SAP-C02 but appear mostly as distractors

## Exam Traps

- **Don't pick Managed Blockchain for ordinary audit trail** — use CloudTrail / DynamoDB versioning / QLDB instead
- **Don't pick Elastic Transcoder when MediaConvert is also offered** — MediaConvert is preferred for new workloads
- **Don't pick IoT Things Graph for IoT rules** — use IoT Core Rules or IoT Events
- **QLDB is closed to new customers (2024)** — questions may still test it; alternative is DynamoDB with item versioning
- **Most of these services are distractors** — the question usually points to a more specific service
