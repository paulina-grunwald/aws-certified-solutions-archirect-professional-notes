# Speechify Podcast Prompts — SAP-C02

30 podcast prompts covering every SAP-C02 in-scope service in the repo. Each prompt is designed for a **~15-minute** podcast that emphasizes **Exam Tips** and **Exam Traps** — the parts that actually win or lose the exam.

## How to use

Each `.md` file is a structured prompt you feed into Speechify (or any AI podcast generator). The prompt tells the generator:

- What topic + which services to cover
- How deep on each service (some get deep dives, some get a one-liner)
- Exactly which Exam Tips + Exam Traps must be verbalized
- The decision matrix to read aloud
- Tone + structure for a 15-min listen

The actual content source is the repo notes in `domain-1-organizational/`, `domain-2-new-solutions/`, `domain-3-continuous-improvement/`, `domain-4-migration/`. Each prompt references the specific repo files to draw from.

## Listening order

The order is built so each podcast assumes you've heard the earlier ones. If you're cramming, skip to the weak areas — every podcast also stands alone.

### Foundations (1–4)
1. [Global Infrastructure & Edge](01-global-infrastructure-edge.md)
2. [IAM & STS Fundamentals](02-iam-sts-fundamentals.md)
3. [Federated Identity](03-federated-identity.md)
4. [Multi-Account Strategy](04-multi-account-strategy.md)

### Networking (5–9)
5. [VPC Core](05-vpc-core.md)
6. [VPC Connectivity Patterns](06-vpc-connectivity-patterns.md)
7. [Hybrid Networking](07-hybrid-networking.md)
8. [DNS & Edge Delivery](08-dns-edge-delivery.md)
9. [Network Security](09-network-security.md)

### Security & Governance (10–12)
10. [Encryption & Secrets](10-encryption-secrets.md)
11. [Security Detection & Audit](11-security-detection-audit.md)
12. [Operations, Governance & Compliance](12-operations-governance.md)

### Compute (13–16)
13. [Compute & Purchasing](13-compute-purchasing.md)
14. [Auto Scaling & Load Balancing](14-autoscaling-load-balancing.md)
15. [Serverless & Managed Apps](15-serverless-managed-apps.md)
16. [Containers](16-containers.md)

### Storage & Data (17–21)
17. [Block & File Storage](17-block-file-storage.md)
18. [S3 Deep Dive](18-s3-deep-dive.md)
19. [Relational Databases](19-relational-databases.md)
20. [NoSQL & Specialty Databases](20-nosql-specialty-databases.md)
21. [Analytics & Data Lakes](21-analytics-data-lakes.md)

### Integration (22–23)
22. [Messaging & Streaming](22-messaging-streaming.md)
23. [Orchestration & API Layer](23-orchestration-api-layer.md)

### Operations (24–27)
24. [Observability](24-observability.md)
25. [DR, Backup & Resilience](25-dr-backup-resilience.md)
26. [IaC & CI/CD](26-iac-cicd.md)
27. [Migration Toolkit](27-migration-toolkit.md)

### Cost & Specialty (28–30)
28. [Cost Management](28-cost-management.md)
29. [AI/ML Services](29-ai-ml-services.md)
30. [IoT, End-User Computing & Niche](30-iot-euc-niche.md)

## Suggested study calendar

| Week | Podcasts | Focus |
|---|---|---|
| 1 | 1–4 | Identity + multi-account |
| 2 | 5–9 | All networking |
| 3 | 10–12 | Security + governance |
| 4 | 13–16 | Compute |
| 5 | 17–21 | Storage + data |
| 6 | 22–27 | Integration + ops + migration |
| 7 | 28–30 + re-listen weak areas | Cost + specialty + cram |

## Prompt structure

Each prompt has:

- **Length target** — words to speak in 15 min
- **Repo references** — which notes the content draws from
- **Coverage depth** — deep vs brief per service
- **Structured outline** — section-by-section timing
- **Must-mention Exam Tips** — verbatim items to call out
- **Must-mention Exam Traps** — verbatim items to call out
- **Decision matrix** — verbalized table
- **Tone & style** — coaching tone, explicit "EXAM TIP:" / "EXAM TRAP:" cues
- **Rapid-fire recap** — 60-second closer drilling the traps

## Style conventions for the generator

- Use phrase "**EXAM TIP:**" and "**EXAM TRAP:**" to cue listener attention before each
- Decision matrices read as "If you see X in the question, pick Y"
- No emojis (auditory waste)
- Speak service names fully on first mention, then acronyms (e.g., "Elastic Compute Cloud — EC2 — ...")
- End every podcast with a 60-second rapid-fire trap drill
- Keep dates / "since YYYY" phrasing out — questions can be timeless
