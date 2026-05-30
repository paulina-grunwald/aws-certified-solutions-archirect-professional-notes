# AWS Lake Formation

> **Centralized permissions and governance layer for AWS data lakes built on S3 + Glue Data Catalog. Replaces the IAM + S3 bucket policy + Glue policy mess with one fine-grained permission model.**

Maps to: **Domain 2.5 — Design high-performance architectures** (data lake governance)

---

## 1. Core Building Blocks

- **Glue Data Catalog** — the metastore (databases, tables, schemas, partitions). LF doesn't replace it; it governs access to it.
- **Registered S3 locations** — you register an S3 path with LF. Once registered, LF (not IAM) is the gatekeeper for that data.
- **Data lake admin** — IAM principal authorized to grant LF permissions. Keep this list small.
- **LF permissions** — granted on databases, tables, columns, rows, and cells. Far more granular than IAM / S3.
- **LF-tags** — labels you attach to catalog resources to drive tag-based access control (LF-TBAC).
- **Resource link** — a Glue catalog object in the consumer account that points to a shared resource in the producer account. Required for cross-account queries from Athena, Redshift Spectrum, EMR, Glue ETL. Without a resource link, these services cannot directly access cross-account catalog data.
- **Data filters** — column projection + row filter expressions for fine-grained security at query time.

---

## 2. Permission Grant Models

### Named resource
- Grant directly on a specific database or table
- Simple, explicit, doesn't scale
- Use when: one-off, single resource, single consumer

### LF-TBAC (tag-based access control) — preferred
- Attach LF-tags (e.g., `Classification=Public`, `Team=Analytics`) to catalog resources
- Grant permissions on tags, not on individual resources
- New resources matching the tag are automatically shared
- Use when: multiple tables, multiple consumers, growing data lake. **Default exam answer.**

### Hybrid Access Mode — incremental migration
- Selectively enable LF permissions for **specific databases / tables**, while keeping IAM + S3 bucket policies for others
- Allows migration from IAM-based to LF-based access **one use case at a time** — doesn't break existing workloads
- Opt-in flag at the table or database level in the Glue Data Catalog
- Use when: rolling out LF on an existing data lake with active IAM-based consumers; can't do big-bang migration

> 💡 **Exam pattern**: "adopt LF without breaking existing IAM-based consumers" → **Hybrid access mode**.

---

## 3. Cross-Account Sharing

Lake Formation uses **AWS RAM** under the hood to share Glue catalog resources across accounts (or to an entire AWS Organization / OU).

### Required steps for cross-account query access
1. **Producer**: register S3 location with LF
2. **Producer**: grant LF permissions to consumer account (via LF-TBAC or named resources)
3. **Consumer**: accept the RAM share (auto-accepted within an Org)
4. **Consumer**: create a **resource link** in its own Glue catalog pointing to the shared resource
5. **Consumer**: grant LF permissions on the resource link to the IAM role used by the query engine (Redshift cluster role, Athena workgroup role, etc.)

### Resource link vs data location permissions — the classic exam trap
- **Data location permissions** = LF permission on a registered S3 location. Lets a principal *create* tables that point to that S3 path. About S3 path access, not table query access.
- **Resource link** = the local catalog shortcut that makes a shared table queryable from the consumer account.
- **Cross-account Athena / Redshift Spectrum / EMR queries require a resource link.** Period.

---

## 4. Fine-Grained Security

Lake Formation enforces these at query time across Athena, Redshift Spectrum, EMR, Glue ETL, and QuickSight:

- **Column-level security** — hide PII columns from analysts
- **Row-level security** — show only rows matching a filter (e.g., `region='EU'`)
- **Cell-level security** — combination of row + column filters in a single data filter

These replace ad-hoc IAM policies and S3 bucket policies — they all share the same LF-defined ruleset.

### Trusted Identity Propagation (TIP) with IAM Identity Center
- Propagate the **actual user identity** from IAM Identity Center into LF queries — fine-grained per-user permissions enforced at the data layer
- LF data filters and row / column security applied **per-user** (not per shared role)
- Audit logs in CloudTrail and at the query engine show the **actual user**, not the role

**Supported query engines**:
- **Amazon QuickSight** (TIP for Athena Direct Query)
- **Amazon Athena**
- **Amazon Redshift** (including Redshift Data API)
- **Amazon EMR / EMR Serverless** (7.8.0+ via Apache Livy)

**Setup**: enable IAM Identity Center as the identity source → configure Lake Formation to accept Identity Center identities → grant LF permissions to **users or groups** (instead of IAM roles).

> 💡 **Exam pattern**: "audit logs must show the actual analyst who ran the query, not a shared role" → **TIP + Lake Formation + IAM Identity Center**.

---

## 5. Redshift Integration

Three integration patterns. Don't conflate them.

### 5a. Redshift Spectrum querying LF tables (the main one)
- Spectrum queries S3 data via an **external schema** mapped to a Glue database
- LF permissions enforced at query time, including column / row filters
- Cross-account: consumer needs a resource link + Redshift IAM role needs LF grants
- Redshift cluster IAM role needs `lakeformation:GetDataAccess` plus the LF data permissions

### 5b. Redshift data sharing (different beast — no S3)
- Share live, transactional data between Redshift clusters / Serverless workgroups
- **Requires RA3 nodes** (or Serverless) on producer and consumer
- Cross-account, cross-region supported
- Producer creates a **datashare**, adds objects, grants to consumer namespace / account
- Lake Formation can govern Redshift datashares — apply column / row security to shared Redshift data

### 5c. Redshift federated queries (related, often confused)
- Redshift queries Aurora PostgreSQL / MySQL or RDS PostgreSQL / MySQL **directly** (not S3)
- Uses Secrets Manager for credentials
- Nothing to do with Lake Formation
- Trap answer when scenario says "query operational DB from Redshift without ETL"

### Disambiguator

| Scenario | Pick |
|---|---|
| "Query S3 data from Redshift" | Spectrum |
| "Share live Redshift data, no copying" | Data sharing (RA3) |
| "Query operational RDS / Aurora from Redshift" | Federated query |
| "Cross-account analytics on a data lake with column-level security" | Spectrum + LF |

---

## 6. Canonical Cross-Account Data Lake Pattern

1. **Central account** owns the data lake (S3 + Glue catalog + LF + LF-tags)
2. Central tags resources by classification, team, or domain
3. Central grants via **LF-TBAC** to consumer accounts (or to the entire AWS Organization / OU)
4. **Consumer accounts** create resource links and query via Athena, Spectrum, EMR
5. Column / row filters applied per consumer for least-privilege access

---

## 7. Exam Tips

- Once an S3 location is **registered with LF**, IAM is no longer the access gatekeeper for that data. Answers that say "use IAM / S3 bucket policies" are usually wrong when LF is in play
- **Resource link** is the magic word for cross-account queryability via Athena, Redshift Spectrum, EMR, Glue ETL
- **LF-TBAC over named resources** for any non-trivial data lake (multiple tables, multiple accounts, growing scope)
- **Column / row / cell-level security** = LF data filters. Same ruleset across all integrated query engines
- **RA3 (or Serverless) is required for Redshift data sharing.** DC2 = wrong answer
- **Spectrum + LF supports column-level filtering** at query time — analysts can be restricted to non-PII columns on the same table
- LF can grant to an entire **AWS Organization** or OU, simplifying fan-out vs per-account grants
- LF integrates with **AWS Glue, Athena, Redshift Spectrum, EMR, QuickSight** — same permission model across all of them
- **Hybrid access mode** is the answer when scenario says "migrate to LF without breaking existing IAM-based consumers" or "incrementally adopt LF"
- **Trusted Identity Propagation (TIP)** with IAM Identity Center is the answer when scenario requires per-user fine-grained access and audit trails showing the actual user (not a shared role)
- TIP works across QuickSight, Athena, Redshift, EMR (7.8.0+). For analytics workflows that need user-level audit trails, this is the modern pattern

---

## 8. Exam Traps

- Data location permissions and resource links solve **different problems** — data location = S3 path access for table creation/registration; resource link = local catalog object for cross-account queries. Don't swap them
- Glue resource policies work for raw Glue but **bypass LF's fine-grained controls** — they are wrong when the scenario uses LF
- Spectrum reads **S3 data**; data sharing shares **live Redshift data** between clusters — watch where the data lives in the scenario to pick the right pattern
- Federated queries, Spectrum, and data sharing are **three distinct patterns** — federated = Redshift querying RDS / Aurora directly
- Once an S3 location is **registered with LF**, LF governs access — bucket policies are typically the wrong answer in that context
- Even with LF, the Redshift cluster IAM role needs **`lakeformation:GetDataAccess`** — both the LF layer and the IAM role are required, not either/or
- Spectrum can query S3 **cross-Region** (with a cost penalty), but Redshift data sharing cross-Region requires **RA3 + explicit setup** — don't assume it works automatically
- IAM-only permission management **scales poorly** for data lakes with many tables and consumers — LF is the right answer for that pattern
- LF-tags are attached to **catalog resources, not S3 objects** — LF-TBAC governs Glue catalog access, not raw S3 ACLs
- Hybrid access mode is **per-resource, not a global switch** — you opt in table-by-table. Misunderstanding this leads to expecting wholesale behavior change
- TIP propagates **identity, not roles** — if the query engine still uses a shared IAM role end-to-end, LF cannot enforce per-user permissions. TIP requires Identity Center as the identity source and a supported engine version (e.g., EMR 7.8.0+)

---

## 9. Likely Question Patterns

- Cross-account data lake → Spectrum / Athena query → which combination? (LF-TBAC + resource link)
- Hide PII column from analysts on a shared table → (LF column-level security / data filter)
- Share live Redshift data with another business unit, no S3 copy → (Redshift data sharing on RA3)
- Centrally manage permissions across many tables and accounts → (Lake Formation + LF-tags + Org-wide grants)
- Query Aurora from Redshift without ETL → (Redshift federated query — NOT Spectrum, NOT LF)
- Migrate from IAM-based data lake to LF without breaking consumers → (Hybrid access mode)
- Need per-user audit trail in analytics queries → (TIP + IAM Identity Center + Lake Formation)

---

## 10. Quick Reference

| Concept | One-liner |
|---|---|
| Lake Formation | Permissions layer on Glue catalog + S3 |
| LF-TBAC | Tag-based grants, scales |
| Named resources | Direct grants on specific tables |
| Resource link | Consumer-side catalog pointer for cross-account queries |
| Data location permission | S3 path-level grant for table creation |
| Data filter | Column / row / cell-level security expression |
| Redshift Spectrum | Query S3 from Redshift (LF-aware) |
| Redshift data sharing | Share live Redshift data, RA3-only |
| Redshift federated query | Query RDS / Aurora directly (no LF) |
| Hybrid access mode | Selectively enable LF per table / db without breaking IAM consumers |
| Trusted Identity Propagation | Propagate user identity from IAM Identity Center into LF queries |
