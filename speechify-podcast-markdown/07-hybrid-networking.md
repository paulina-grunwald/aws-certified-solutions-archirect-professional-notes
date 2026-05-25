# Podcast 07 — Hybrid Networking

**Length target**: 15 min
**Repo references**: `domain-1-organizational/12-direct-connect.md`, `30-site-to-site-vpn.md`, `31-client-vpn.md`, `41-verified-access.md`

## Topic & Scope

Connecting on-prem to AWS. Direct Connect vs Site-to-Site VPN vs Client VPN vs Verified Access — and the recurring HA patterns SAP-C02 tests.

## Service Coverage Depth

**Deep**:
- Direct Connect — physical fiber, hosted vs dedicated, virtual interfaces (private/public/transit)
- Site-to-Site VPN — IPsec tunnels, two tunnels per connection by default
- DX + VPN as HA failover pattern
- AWS Verified Access — zero-trust, no VPN needed for app access

**Brief**:
- Client VPN — managed OpenVPN endpoint for individual users
- Direct Connect Gateway (mention, defer to TGW context)

## Structured Outline

1. **Open (30s)** — "Three ways to connect on-prem to AWS, one zero-trust alternative. The exam tests when to pick each."
2. **Site-to-Site VPN (3 min)** — IPsec, two tunnels for HA, BGP optional, encrypted over public internet, fast to set up, GB-scale OK
3. **Direct Connect (4 min)** — physical fiber to AWS, dedicated 1/10/100 Gbps vs hosted shared, private VIF (to VPC), public VIF (to AWS services), transit VIF (to TGW)
4. **HA Pattern: DX + VPN (2 min)** — primary DX, backup VPN with BGP failover
5. **Client VPN (2 min)** — managed OpenVPN, individual users (employees, contractors), MFA via SAML
6. **Verified Access (2.5 min)** — zero-trust browser-based access to apps without VPN, JIT IdP + device trust verification
7. **Rapid-Fire Trap Drill (60s)**

## Must-Mention Exam Tips

- Site-to-Site VPN: encrypted over public internet, two tunnels per connection (use both for HA)
- Site-to-Site VPN goes up in minutes, supports BGP for dynamic routing
- Direct Connect: physical fiber, takes weeks to provision (cross-connect order)
- DX dedicated: 1/10/100 Gbps, single tenant fiber
- DX hosted: shared via APN partner, 50 Mbps – 10 Gbps
- DX VIFs: private (to VPC), public (to AWS public services), transit (to TGW)
- DX is NOT encrypted by default — add IPsec on top for encryption
- Direct Connect Gateway: connect one DX to multiple VPCs / Regions
- HA pattern: DX as primary, Site-to-Site VPN as backup with BGP, automatic failover
- For more bandwidth: LAG (link aggregation) on dedicated DX, or multiple connections
- Client VPN: per-user OpenVPN, SAML auth for MFA
- Verified Access: zero-trust app access — no VPN — checks IdP + device trust per request

## Must-Mention Exam Traps

- EXAM TRAP: Direct Connect is NOT encrypted by default — add IPsec VPN over DX for encryption (MACsec on 10/100 Gbps DX is an option)
- EXAM TRAP: Direct Connect provisioning takes WEEKS, not minutes — VPN is the only fast option
- EXAM TRAP: Site-to-Site VPN provides TWO tunnels — use both in HA, not one
- EXAM TRAP: VPN goes over public internet — latency varies; not deterministic
- EXAM TRAP: Direct Connect alone is not highly available — single fiber = SPoF; use multiple DX locations or DX + VPN
- EXAM TRAP: A private VIF connects to ONE VPC — for multiple VPCs use Direct Connect Gateway
- EXAM TRAP: Public VIF gives access to all AWS public services (S3 endpoints, etc.) but NOT your VPC
- EXAM TRAP: Client VPN supports SAML federation — pairs with Identity Center
- EXAM TRAP: Verified Access is for HTTPS web apps only — not for general network access
- EXAM TRAP: Verified Access requires an IdP integration (Identity Center, Okta, OIDC) — not standalone

## Key Decision Matrix

| Scenario | Pick |
|---|---|
| Fast on-prem-to-AWS connectivity, encrypted | Site-to-Site VPN |
| Consistent low latency + high bandwidth on-prem | Direct Connect |
| HA hybrid connectivity | DX primary + VPN backup with BGP |
| Encrypted Direct Connect | DX + IPsec (or MACsec on 10/100 Gbps) |
| Individual employee remote access | Client VPN |
| Browser-based zero-trust app access without VPN | Verified Access |
| Connect one DX to many VPCs | Direct Connect Gateway |
| Connect DX to TGW for multi-VPC | Transit VIF |

## Tone & Style

- Frame as "encrypted-but-public" (VPN), "dedicated-but-unencrypted" (DX), "zero-trust-no-VPN" (Verified Access)
- Emphasize the weeks-to-set-up DX latency vs minutes for VPN
- Repeat "DX is NOT encrypted by default" three times

## Rapid-Fire Closer

"DX encrypted by default?" — "NO." "Site-to-Site VPN tunnels?" — "TWO." "DX provisioning time?" — "WEEKS." "DX + VPN pattern?" — "HA, primary + backup." "Verified Access replaces?" — "VPN for web apps."
