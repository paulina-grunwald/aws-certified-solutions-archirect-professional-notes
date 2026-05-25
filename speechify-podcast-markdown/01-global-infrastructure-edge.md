# Podcast 01 — AWS Global Infrastructure & Edge

**Length target**: 15 min (~2,200 spoken words)
**Repo references**: `domain-1-organizational/01-aws-global-infrastructure.md`, `13-aws-local-zones.md`, `14-aws-outposts.md`, `domain-2-new-solutions/43-wavelength.md`

## Topic & Scope

The geographic + edge story: Regions, AZs, Local Zones, Outposts, Wavelength. SAP-C02 tests when to pick each — these scenarios are almost always disguised "which location" puzzles.

## Service Coverage Depth

**Deep**:
- Regions + AZs (failure domains, latency, data sovereignty)
- Local Zones vs Outposts vs Wavelength (the decision matrix is the whole point)

**Brief — "what it does + when to pick"**:
- Edge locations / CloudFront PoPs (covered fully in podcast 8)
- AWS Verified Access edge (covered in podcast 7)

## Structured Outline

1. **Open (30s)** — "Where you put workloads decides latency, cost, and compliance. Today: the four AWS placement options."
2. **Regions & AZs (3 min)** — pair of independent power/network/cooling DCs per AZ, 3+ AZs per Region, Region selection criteria (data residency, latency, service availability, cost)
3. **Local Zones (3 min)** — single-AZ extension in a metro area, sub-10ms to city users, subset of services, parent-Region attachment
4. **Outposts (3 min)** — AWS hardware in YOUR data center, full local services subset, low-latency to on-prem systems, fully managed
5. **Wavelength (3 min)** — embedded in 5G carrier networks, single-digit-ms to mobile devices, carrier gateway not IGW
6. **Decision Matrix (1.5 min)** — verbalize which to pick for each scenario
7. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- "Highly available within Region" → multi-AZ
- "Recover from Region failure" → multi-Region
- "Sub-10ms latency to a US city for web users" → Local Zone
- "On-prem latency or data residency on customer premises" → Outposts
- "Sub-10ms latency to mobile devices on 5G network" → Wavelength
- AZs are physically separated by meaningful distance but synchronously close
- A Region selection criteria: data residency, latency to user, service availability, cost

## Must-Mention Exam Traps

- EXAM TRAP: Local Zone ≠ Outposts ≠ Wavelength — each solves a different latency problem
- EXAM TRAP: Wavelength only helps devices on the carrier's 5G network — Wi-Fi users don't benefit
- EXAM TRAP: Outposts requires physical hardware delivery + power/network — not instant
- EXAM TRAP: Not every AWS service is available in every Region — check before designing
- EXAM TRAP: Multi-AZ ≠ Multi-Region — questions often confuse these
- EXAM TRAP: Local Zones have a parent Region — control plane lives there
- EXAM TRAP: AZ identifiers (us-east-1a) are randomized per account — don't hardcode across accounts

## Key Decision Matrix to Verbalize

| Scenario clue | Pick |
|---|---|
| Sub-10ms to mobile 5G devices | Wavelength |
| Sub-10ms to a metro area (web/gaming) | Local Zone |
| On-prem co-location or data residency on customer premises | Outposts |
| HA across power/cooling/network | Multi-AZ |
| HA across Region disasters | Multi-Region |

## Tone & Style

- Open with the "where to put it" framing
- Use coastal-city examples for Local Zones (LA, Boston)
- Say "EXAM TRAP" / "EXAM TIP" explicitly before each one
- End with rapid-fire: "Mobile 5G?" — "Wavelength." "Customer DC?" — "Outposts." Etc.

## Rapid-Fire Closer (60s)

Run through 8 scenarios with one-word answers. Repeat answer once for memorization. Example: "On-prem manufacturing floor needs sub-millisecond compute? — Outposts. Outposts. Next."
