# Amazon ElastiCache (& MemoryDB)

> **Fully managed, in-memory caching. Engines: Valkey (default), Redis OSS 7.4 (legacy), Memcached. Sub-ms reads. Deploy as provisioned (cluster mode disabled/enabled) or Serverless (auto-scale). For durable Redis/Valkey workloads use the sibling service MemoryDB. Vector search supported (Valkey 8 / Redis 7.4) for GenAI / RAG.**

Maps to: **Domain 2.5 — Caching / performance**, **Domain 3.2 — Improve performance**

---

## Engines

| Engine | Notes |
|---|---|
| **Valkey** | Open-source Redis fork — Linux Foundation-backed. **AWS default** after Redis Inc. relicensed Redis 7.4+ to non-OSI. **~20% cheaper** than Redis OSS on ElastiCache. Wire-compatible with Redis clients. Versions 7.2 → 8.x |
| **Redis OSS** | Last OSI version = 7.4. Redis 8+ (Source Available) NOT on ElastiCache. AWS recommends Valkey for new workloads |
| **Memcached** | Simple, **multi-threaded**, key-value only; no replication, no Multi-AZ, no persistence; horizontal scale-out via Auto Discovery |

> Valkey and Redis OSS are **wire-compatible** — same client libraries, same commands. Switching engines does not require app code changes.

---

## Deployment Models

### Provisioned

- Choose node type + cluster topology
- **Cluster Mode Disabled** — single shard, primary + up to 5 read replicas; vertical scale only
- **Cluster Mode Enabled** — partition across **up to 500 shards** (250 TiB of data); horizontal + vertical scale; hash-slot sharding
- **Multi-AZ** with auto-failover (Redis / Valkey only — Memcached has no Multi-AZ)
- **Reserved nodes** for 1- or 3-year commitments

### Serverless

- Engine variants: **Valkey Serverless**, **Redis OSS Serverless**, **Memcached Serverless**
- **Auto-scales** in seconds based on usage
- Pay per **ECPU** (ElastiCache Processing Unit) + GB-hour stored
- **Sub-millisecond reads, single-digit-ms writes**
- **99.99% SLA**, multi-AZ by default
- No node sizing required — eliminates capacity planning

---

## Key Features

### Redis / Valkey (advanced)

- **Data structures**: strings, lists, sets, sorted sets, hashes, streams, bitmaps, HyperLogLog, geospatial
- **Master + replicas** with auto-failover (Multi-AZ)
- **Persistence** — RDB snapshots + AOF logs
- **Backup / restore** to S3
- **Vector search** support (Valkey 8 / Redis 7.4) — GenAI memory and RAG
- **Pub/Sub** + **Streams** for messaging patterns
- **Up to 500 shards** in cluster mode → 250 TiB of data
- **Global Datastore** — cross-Region read replication with < 1 s lag (read scaling + DR)
- **Data Tiering** (`r6gd` nodes) — stores cold keys on NVMe local SSD, hot keys in RAM; ~60% cheaper for large datasets
- **Encryption at rest** (KMS) + **in transit** (TLS)
- **IAM authentication** — auth tokens via IAM, eliminates static passwords
- **Redis AUTH** classic password mechanism still supported (`auth_token` + `transit-encryption-enabled`)
- **Role-Based Access Control (RBAC)** for fine-grained user permissions

### Memcached (simple)

- **Multi-threaded** — vertical scale with bigger node sizes
- **No replication, no Multi-AZ, no persistence**
- **Horizontal scale-out** by adding nodes — **Auto Discovery** handles client-side sharding
- Key/value with values up to 1 MB

---

## Caching Strategies

### Lazy Loading (cache-aside)

- App checks cache → miss → fetch from DB → write to cache
- **Pros**: only cached data is actually requested; resilient to misses
- **Cons**: misses are slow (3 trips); stale data possible

### Write-Through

- Every DB write also writes to cache
- **Pros**: cache is never stale
- **Cons**: every write incurs cache write (latency + cost); cached data may never be read
- **DAX uses write-through** (DynamoDB Accelerator) — same pattern, different service

### TTL

- Per-key expiration (Redis `EXPIRE`, Memcached TTL)
- Forces refresh from DB after expiry
- Combine TTL + lazy loading + write-through for fresh-enough data without manual invalidation

---

## MemoryDB (sibling service)

- **Durable, persistent Redis / Valkey** — Multi-AZ with strong consistency on writes
- Use case: primary DB needing sub-ms reads + ms writes (chat, leaderboards, geospatial, ML feature stores)
- **MemoryDB durably persists writes to a Multi-AZ transaction log** (microsecond reads, single-digit-ms writes)
- ElastiCache = cache (volatile); **MemoryDB = primary DB (durable)**
- **MemoryDB Multi-Region** for active-active across Regions
- Supports **vector search** for GenAI

---

## Use Cases

- **Session store** — stateless app; user session in ElastiCache shared across instances
- **Database query caching** — offload reads from RDS / DynamoDB / Postgres / Aurora
- **Leaderboards / counters** — Redis sorted sets
- **Real-time analytics** — Redis Streams
- **Rate limiting** — atomic counters
- **Pub/Sub** — Redis Pub/Sub or Streams
- **ML feature stores** — sub-ms feature retrieval
- **Vector search / RAG** — Valkey 8 / Redis 7.4 with vector data types

---

## Security

- **VPC-only** access (no public endpoint)
- **Encryption at rest** with KMS (AWS-owned / AWS-managed / customer-managed)
- **Encryption in transit** with TLS (Redis / Valkey; Memcached also supports TLS)
- **IAM authentication** for Redis / Valkey
- **RBAC** with user groups for Redis / Valkey
- **Subnet groups** + security groups for network isolation

---

## Cross-Region (Redis / Valkey)

- **Global Datastore** — primary cluster + up to 2 secondary read-only clusters in other Regions
- Replication lag typically < 1 second
- Promote a secondary on DR — writes only happen in the primary
- For **active-active multi-Region writes**, use **MemoryDB Multi-Region** or pick a different service (Aurora DSQL, DynamoDB Global Tables with MRSC)

---

## Exam Tips

- "Sub-millisecond cache for a database" → **ElastiCache (Valkey / Redis)**
- "Session store for stateless app" → **ElastiCache Redis / Valkey**
- "Need data structures (sorted set, hash, geospatial)" → **Redis / Valkey**
- "Simple multi-threaded key-value cache, scale out horizontally" → **Memcached**
- "Durable in-memory primary DB" → **MemoryDB** (not ElastiCache)
- "No capacity planning" → **ElastiCache Serverless**
- "~60% cheaper for large in-memory dataset, willing to tier cold keys to SSD" → **Data Tiering** on `r6gd` nodes
- "Cross-Region read replication, < 1 s lag" → **Global Datastore**
- "Vector search for GenAI" → Valkey 8 / Redis 7.4 on ElastiCache or MemoryDB
- "Cache for DynamoDB with no code changes" → **DAX** (not ElastiCache)
- "Eliminate static Redis passwords" → **IAM auth + RBAC**
- **DAX uses write-through only** — Redis / Memcached support multiple strategies
- **Valkey is AWS's default engine** — Redis OSS supported only at v7.4 (last OSI version)
- "Multi-Region active-active in-memory writes" → **MemoryDB Multi-Region**

---

## Exam Traps

- **ElastiCache is VPC-only** — never publicly accessible
- **Memcached has no Multi-AZ, no replication, no persistence** — never use for stateful caching
- **Cluster Mode Enabled requires sharding-aware client** — apps must use Redis Cluster-compatible client
- **Newer Redis (8+, Source Available) is NOT on ElastiCache** — AWS supports Redis OSS 7.4 max; for newer features, use Valkey
- **Promoting a Global Datastore secondary is one-way** (same trap as RDS read replicas)
- **Cache invalidation is your responsibility** — TTL + write-through + lazy-loading combo
- **ElastiCache is volatile** — for durability, use **MemoryDB**
- **Using ElastiCache requires application code changes** — unlike DAX, which is drop-in for DynamoDB
- **Encryption at rest must be enabled at cluster creation** — cannot be turned on later
- **In-transit encryption (TLS) must be enabled at creation** for Redis AUTH to work
- **Auto Discovery (Memcached)** requires a compatible client library
- **Serverless ElastiCache still requires VPC connectivity** — "no infrastructure" only means no node sizing
- **DAX ≠ ElastiCache** — DAX is purpose-built for DynamoDB (no code changes); ElastiCache is general-purpose with app-level cache logic
