# Podcast 08 — DNS & Edge Delivery

**Length target**: 15 min
**Repo references**: `domain-1-organizational/11-route-53.md`, `28-cloudfront.md`, `29-global-accelerator.md`

## Topic & Scope

Route 53 routing policies, CloudFront, and Global Accelerator. The "low latency to global users" pattern is the #1 SAP-C02 question type and these are the answers.

## Service Coverage Depth

**Deep**:
- Route 53 — all routing policies (Simple, Weighted, Latency, Geo, Geoproximity, Failover, Multi-value)
- CloudFront — origins, behaviors, OAC, signed URLs/cookies, edge functions, caching
- Global Accelerator — anycast IPs, traffic dial, endpoint groups

**Brief**:
- Route 53 Resolver (rules, inbound/outbound endpoints — for hybrid DNS)
- Route 53 Application Recovery Controller (mentioned in podcast 25)

## Structured Outline

1. **Open (30s)** — "Three services solve 'low-latency for global users'. They overlap, they differ — let's nail the differences."
2. **Route 53 (5 min)** — public/private hosted zones, all 7 routing policies with use cases, health checks, alias records, DNSSEC, Resolver
3. **CloudFront (5 min)** — global PoPs, origin (S3 / ALB / EC2 / on-prem), behaviors / cache key, OAC for S3, signed URLs vs cookies, Lambda@Edge vs CloudFront Functions, geo-restriction
4. **Global Accelerator (3 min)** — two static anycast IPs, TCP/UDP, traffic dial + endpoint weights, endpoint groups per Region
5. **CloudFront vs Global Accelerator (1.5 min)** — the most-tested distinction
6. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- Route 53 policies: Simple, Weighted (A/B), Latency (route to lowest-latency Region), Geolocation (route by user country/continent), Geoproximity (route by geographic distance, bias-tunable), Failover (active-passive), Multi-value (client-side LB with health checks)
- Alias record: free, points to AWS resources (ELB, CloudFront, API Gateway, S3 website), can be root domain
- CNAME cannot be on the apex/root domain — alias can
- Health checks: HTTPS, TCP, calculated (combine other checks), CloudWatch alarm
- CloudFront: 400+ edge PoPs, caches static + dynamic content, supports any TCP/UDP origin
- OAC (Origin Access Control) replaces OAI for restricting S3 access to CloudFront only
- Signed URLs: per-file; signed cookies: per-session, multiple files
- Lambda@Edge: viewer/origin request/response, full Node/Python runtime, longer execution
- CloudFront Functions: viewer-only, JavaScript, sub-millisecond, simpler header/URL manipulation
- Global Accelerator: 2 static anycast IPs, TCP + UDP, routes to healthiest/closest endpoint, traffic dial controls Region split
- Global Accelerator works at network layer (any TCP/UDP); CloudFront at HTTP/HTTPS

## Must-Mention Exam Traps

- EXAM TRAP: CNAME can't be at zone apex — use Alias for the root domain
- EXAM TRAP: Latency-based routing routes to the lowest-LATENCY Region, NOT the closest geographic Region — they often differ
- EXAM TRAP: Geolocation vs Geoproximity — Geolocation is "user from country X go to record Y"; Geoproximity is distance-based with bias
- EXAM TRAP: CloudFront is HTTP/HTTPS only — for TCP/UDP latency-sensitive apps use Global Accelerator
- EXAM TRAP: Global Accelerator gives static IPs — CloudFront does NOT (it has many edge IPs)
- EXAM TRAP: Signed URLs are per-FILE — for many files use Signed Cookies
- EXAM TRAP: OAC replaces OAI — OAI is legacy; OAC supports SSE-KMS + S3 access points
- EXAM TRAP: CloudFront caches based on the CACHE KEY (URL + headers + query string + cookies) — misconfigured cache key = cache misses
- EXAM TRAP: Lambda@Edge can run at 4 points; CloudFront Functions only at viewer request/response
- EXAM TRAP: Route 53 health checks are global from multiple regions — they DON'T originate from your VPC
- EXAM TRAP: A failover routing record needs a HEALTHY primary AND a secondary — health check on primary

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| HTTP/HTTPS content acceleration globally | CloudFront |
| TCP/UDP non-HTTP traffic global acceleration | Global Accelerator |
| Static anycast IP for white-listing | Global Accelerator |
| Route users to closest Region by latency | Route 53 Latency policy |
| Route users by country / continent | Route 53 Geolocation |
| Active-passive DR | Route 53 Failover policy + health check |
| Restrict S3 origin to CloudFront only | OAC |
| Per-file restricted download URL | Signed URL |
| Per-session restricted access | Signed Cookies |
| Sub-millisecond header rewrite at edge | CloudFront Functions |
| Complex edge logic with full runtime | Lambda@Edge |

## Tone & Style

- Start with "low-latency for global users" framing
- Lean hard on CloudFront vs Global Accelerator distinction
- For Route 53 policies, give each a one-sentence example

## Rapid-Fire Closer

"CNAME at apex?" — "NO, use Alias." "Latency vs Geoproximity?" — "Latency = network speed, Geo = location." "HTTPS global cache?" — "CloudFront." "TCP/UDP static IPs?" — "Global Accelerator." "OAI replaced by?" — "OAC."
