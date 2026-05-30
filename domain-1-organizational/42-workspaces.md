# Amazon WorkSpaces (+ WorkSpaces Web + AppStream 2.0)

> **End-User Computing (EUC) family. WorkSpaces = persistent virtual desktops (Windows / Linux / Ubuntu). WorkSpaces Web = browser-isolation service for SaaS access. AppStream 2.0 = streamed application sessions (no full desktop). WorkSpaces Thin Client = dedicated hardware for accessing WorkSpaces. Common SAP-C02 scenarios: replace on-prem VDI (Citrix, VMware Horizon), give contractors secure access without shipping laptops, isolate browsing in regulated industries.**

Maps to: **Domain 4.2 — Migration**, **Domain 4.3 — Modernization**, **Domain 1.2 — Security**

---

## Amazon WorkSpaces (Virtual Desktops)

### Overview

- **Managed persistent desktop-as-a-service**
- Bundles: Windows 10 / 11, Amazon Linux 2, Ubuntu 20.04 / 22.04
- Persistent user data on root + user volumes (EBS-backed; encrypted)
- Streamed via **PCoIP** or **WSP (WorkSpaces Streaming Protocol)** — WSP is the modern default

### Compute Bundles

- **Value / Standard / Performance / Power / PowerPro / Graphics / GraphicsPro / Graphics.g4dn**
- GPU bundles for CAD, 3D, ML workstations

### Billing Modes

| Mode | When | Cost |
|---|---|---|
| **Monthly** | Always-on, full-time users | Fixed per desktop/month |
| **Hourly** | Part-time users | Small monthly fixed + per-hour active charge |

> Hourly auto-stops after configured idle period; resumes on user connect.

### Directory Integration

- Required: **AWS Managed Microsoft AD**, **AD Connector**, or **Simple AD**
- Or trust relationship to on-prem AD
- Users sign in with their AD credentials
- **MFA** via RADIUS server (Duo, RSA, etc.)

### Networking

- Each WorkSpace gets an **ENI in a customer VPC subnet**
- Sees customer-VPC private resources (RDS, file shares, internal apps)
- Internet access via NAT GW or VPC route
- **WorkSpaces management interface** is a separate AWS-managed ENI in another subnet (transparent to users)

### Security

- **EBS volumes encrypted with KMS** (CMK supported)
- **WSP / PCoIP** session encryption
- **IP Access Control Groups** — limit which client IPs can connect
- **MFA via RADIUS**
- **CloudWatch metrics** + **CloudTrail** logs for sessions
- **HIPAA, PCI, SOC, ISO** eligible

### Common Patterns

- **Contractor / remote workforce** — give vendors secure desktops without shipping laptops
- **Regulated workloads** — keep data in AWS; user only sees pixels
- **Citrix / VMware Horizon migration** — lift VDI to managed service
- **Developer environments** — full IDE, large memory, low local hardware needs

---

## WorkSpaces Web (formerly Amazon WorkSpaces Web)

### Overview

- **Browser-isolation service** — users access SaaS web apps via a managed browser running in AWS
- No desktop, no installer; user just gets a browser tab
- Sessions are ephemeral — closed when user disconnects
- **Use case**: BYOD contractors accessing internal web apps; minimize data leakage

### Capabilities

- Federate with **IAM Identity Center** or SAML IdP
- Policy controls: copy/paste, download, upload, print, file transfer
- Audit logs to CloudWatch / S3
- IP allowlist for client device
- **No client install** — open the URL, sign in, browser session starts

### WorkSpaces Web vs Verified Access

| Need | Pick |
|---|---|
| Stream a browser session running in AWS (no local browser data) | **WorkSpaces Web** |
| Authenticate + authorize each request to existing internal web app | **Verified Access** |

> Verified Access protects an app; WorkSpaces Web isolates the browser itself.

---

## AppStream 2.0

### Overview

- **Streamed application sessions** — user sees one app (e.g., AutoCAD), not a full desktop
- Per-session fleets (auto-scale based on demand)
- **Always-On**, **On-Demand**, **Elastic** fleet types
- Streaming via HTML5 in browser; no client install
- Sessions are stateless; user data via shared S3, EFS, or Home Folders

### Use Cases

- Software publishers distributing apps to customers without install
- High-spec apps (CAD, video editing) on low-spec laptops
- Trial / demo environments

### AppStream vs WorkSpaces

| Need | Pick |
|---|---|
| Persistent desktop for daily user work | **WorkSpaces** |
| Single app streamed for hours / occasional use | **AppStream 2.0** |
| Browser session only | **WorkSpaces Web** |

---

## WorkSpaces Thin Client

- Dedicated hardware ($195) that boots directly into WorkSpaces / WorkSpaces Web
- Manages via central console
- Drop-in replacement for legacy thin clients (Citrix, Wyse) at lower TCO
- Good for call centers, retail, healthcare front desks

---

## Cost-Saving + DR Patterns

- **Auto-stop** for hourly WorkSpaces — saves cost on idle desktops
- **Multi-Region WorkSpaces** — primary + warm-standby Region with cross-Region AD replication
- **WorkSpaces Pools** — non-persistent ephemeral desktops for workforce that doesn't need persistent state (cheaper)
- **BYOL Windows** — bring your own Windows license for ~17% savings
- **Custom images / bundles** — bake apps into a custom AMI; reduces user setup time

---

## Exam Tips

- **WorkSpaces = full persistent virtual desktop** (Windows / Linux), VDI-as-a-service
- **WorkSpaces Web = managed browser session** for SaaS / web apps
- **AppStream 2.0 = streamed single application**, not a desktop
- **Hourly vs Monthly** billing — pick hourly for part-time users
- **Directory required** — Managed AD, AD Connector, or trust to on-prem AD
- **WSP** is the modern streaming protocol (vs older PCoIP)
- **WorkSpaces Pools** for ephemeral non-persistent desktops
- **GPU bundles** for CAD / 3D / ML workstations
- **Thin Client** for low-cost dedicated hardware
- Default exam answer for "managed VDI to replace on-prem Citrix/Horizon" → **WorkSpaces**
- Default exam answer for "secure browser access for contractors with no install" → **WorkSpaces Web**
- Default exam answer for "stream one app to many users" → **AppStream 2.0**

## Exam Traps

- **WorkSpaces requires a Directory Service** (Managed AD, AD Connector, or Simple AD) — not optional
- **WorkSpaces Web ≠ WorkSpaces** — Web is browser-only; WorkSpaces is full desktop
- **Hourly auto-stop** has a minimum 1-hour billing — sub-hour sessions still bill the full hour
- **WorkSpaces ENIs sit in customer VPC subnets** — make sure the subnet has IPs available + route to NAT for internet
- **PCoIP is being deprecated** in favor of WSP — pick WSP for new deployments
- **WorkSpaces Pools** are non-persistent — user data is lost at session end unless redirected to S3 / FSx
- **AppStream sessions are ephemeral** by default — persistent user data needs Home Folders (S3) or shared FSx
- **Multi-Region WorkSpaces** require manual setup (no native cross-Region replication) — replicate AD + use cross-Region image copy
- **GPU bundles are expensive** — scope to users who need them; standard users on Standard/Performance
- **BYOL Windows requires meeting Microsoft licensing rules** (dedicated tenancy / minimum user count)
- **Don't confuse AppStream with Elastic Beanstalk Streaming** — AppStream is EUC; ES is for web apps
