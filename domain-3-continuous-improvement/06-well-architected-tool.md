# AWS Well-Architected Tool (WA Tool)

> **Free, self-service review tool that walks you through the AWS Well-Architected Framework questionnaire across the six pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability. Identifies high-risk issues (HRIs), tracks improvement plans, and supports lenses for specialized workloads (Serverless, IoT, SaaS, ML, Foundation Model, FinServ, etc.). Integrates with Trusted Advisor + Service Catalog AppRegistry. Used to demonstrate governance + continuous improvement.**

Maps to: **Domain 3 — All improvement task statements**, **Domain 1.4 — Multi-account governance**

---

## The Well-Architected Framework — Six Pillars

| Pillar | Focus |
|---|---|
| **Operational Excellence** | Run + monitor systems, continually improve processes |
| **Security** | Protect data, systems, assets — apply least privilege, defense in depth |
| **Reliability** | Recover from failures, scale automatically, test recovery |
| **Performance Efficiency** | Use compute efficiently, evolve as needs change |
| **Cost Optimization** | Avoid unneeded costs, match supply to demand |
| **Sustainability** | Minimize environmental impact |

Memorize the six. SAP-C02 questions test which pillar a concern maps to.

## Workloads

- A **workload** = the set of resources delivering business value
- One review = one workload
- Define: workload name, owner, environment (prod/non-prod), regions, AWS accounts

## Conducting a Review

- Answer questions per pillar (yes / no / not applicable)
- Each question maps to AWS best practices
- WA Tool calculates **High-Risk Issues (HRIs)** and **Medium-Risk Issues (MRIs)** based on unaddressed best practices
- **Improvement plan** auto-generated — prioritized list of items to fix
- Track progress over time across multiple milestones

## Milestones

- Snapshot of the review at a point in time
- Compare milestones to see resilience/security improvements
- Required for tracking "we improved from Y HRIs to X HRIs over Q3"

## Lenses

Specialized question sets for specific workload types:

- **Serverless Lens** — Lambda, API Gateway, DynamoDB patterns
- **SaaS Lens** — multi-tenant SaaS apps
- **IoT Lens** — device + cloud + analytics
- **Machine Learning Lens** — ML lifecycle (data, training, deployment)
- **Foundation Model Lens** — generative AI workloads (Bedrock, SageMaker)
- **Financial Services Lens**
- **Data Analytics Lens**
- **Streaming Media Lens**
- **DevOps Lens**
- **Container Build Lens**
- **Government / Public Sector Lens**

You can also write **custom lenses** for your org-specific best practices.

## Trusted Advisor Integration

- WA Tool surfaces relevant Trusted Advisor checks per pillar
- Helps quickly identify items that already have automated detection

## Service Catalog AppRegistry Integration

- Link an **AppRegistry application** to a workload
- WA review tracks against the application bound to real AWS resources
- Tag-based linking

## Sharing Reviews

- Share a workload review with another **AWS account** or **IAM principal**
- Read-only or contributor access
- Used when consultants or partners run the review

## Profiles

- A **profile** = business context (industry, role, threats)
- WA Tool tailors which questions/best practices are prioritized
- E.g., a regulated financial-services profile elevates security/compliance questions

## Connector to Jira

- Sync WA improvement items to **Jira** as tickets
- Track remediation work in your engineering tool of choice

## Pricing

- **Free** — the tool itself
- Costs come from the recommended changes (architecture, services)

## Common Patterns

### Pre-launch review

- Conduct a WA review before promoting a workload to prod
- Address all HRIs before go-live
- Establish baseline milestone

### Quarterly continuous improvement

- Run the review each quarter
- Track HRI count over time
- Tie reduction to engineering OKRs

### Lens-specific deep dive

- Serverless workload → apply Serverless Lens (more relevant questions than general WAF)
- ML workload → apply ML Lens

### Multi-account, multi-workload

- Run separate WA reviews per workload
- Use **Profiles** + **AppRegistry** for org-wide consistency
- Share with central architecture team

## Relationship to Other Services

| Service | Relationship |
|---|---|
| **Trusted Advisor** | WA Tool surfaces relevant TA checks; TA is automated detection, WA is structured questionnaire |
| **Resilience Hub** | Resilience-pillar-specific automation; WA Tool covers all six pillars |
| **AWS Config** | Compliance automation; WA Tool is strategic review |
| **Security Hub** | Security findings automation; WA Tool Security pillar is strategic |
| **Service Catalog AppRegistry** | Link workload to real resources |

## Exam Tips

- "Structured review of architecture against AWS best practices" → **Well-Architected Tool**
- Six pillars — memorize all six and what each covers
- **Sustainability** pillar — environmental impact
- **Lenses** for specialized workloads (Serverless, ML, FM, SaaS, IoT, FinServ, etc.)
- **Milestones** track improvement over time
- **HRI / MRI** = high / medium-risk issues
- Free service; only the remediation costs money
- **Profile** = business context that shifts question priorities
- **Custom lenses** for org-specific best practices

## Exam Traps

- **WA Tool is a questionnaire, not automation** — it doesn't scan resources; it asks structured questions and tracks your answers
- **Trusted Advisor ≠ WA Tool** — TA is automated checks; WA is strategic review (they complement)
- **Reliability pillar ≠ Resilience Hub** — Resilience Hub is automated assessment with RTO/RPO; WA Reliability pillar is qualitative review
- **Cost pillar ≠ Cost Explorer / Cost Optimization Hub** — WA Cost is strategic; the others are tactical
- **"Operational Excellence" pillar covers monitoring + ops processes** — easy to confuse with Reliability
- **Lenses are additive** — apply a Lens on top of the base framework, not instead
- **Sustainability pillar is real and tested** — don't forget it exists
