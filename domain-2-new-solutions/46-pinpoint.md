# Amazon Pinpoint

> **Multi-channel customer engagement service. Send marketing + transactional messages via email, SMS, voice, push notifications, in-app messages, and custom channels. Built-in user segmentation, A/B testing, journey workflows (Pinpoint Journeys), and analytics (delivery, open, click). Common SAP-C02 distractor against SES (email-only), SNS (pub-sub), and Connect (contact center). Pick Pinpoint when the scenario mentions targeted user campaigns + analytics + multi-channel.**

Maps to: **Domain 4.3 — Modernization**, **Domain 4.4 — Modernization opportunities**

---

## Overview

- **Customer engagement platform** for marketing + transactional messaging
- **Multi-channel**: email, SMS, voice, push (APNs / FCM), in-app, custom
- **Segmentation**: target users by attributes / behavior
- **Journeys**: multi-step customer flows (welcome series, re-engagement, abandoned cart)
- **A/B testing**: variants of message / journey
- **Analytics**: delivery, open, click, conversion rates
- **Campaigns**: ad-hoc message sends to a segment

## Channels in Detail

### Email
- Backed by **SES** (Pinpoint sends via SES under the hood)
- Templates with substitution variables
- Open + click tracking

### SMS
- Sender ID / short code / long code (per country regulations)
- Two-way SMS (inbound replies)
- **AWS End User Messaging SMS** (rebranded subset)

### Push Notifications
- iOS via **APNs**
- Android via **FCM**
- Amazon devices via **ADM**
- Web push via **GCM**

### Voice
- Outbound text-to-speech voice calls
- Use case: 2FA codes, automated reminders

### In-App
- Banners / modals in your mobile app via Pinpoint SDK
- Triggered by user behavior in app

### Custom Channel
- Send to anything (your own webhook → Lambda)

## Pinpoint Journeys

- **Multi-step automated flows**: send email → wait 3 days → if not opened, send SMS → if clicked, mark as engaged
- Decision splits, wait times, holdout groups
- Visual editor in Pinpoint console
- Used for: welcome series, drip campaigns, lifecycle marketing

## Segmentation

- **Dynamic segments** based on user attributes (location, lifetime value, app behavior)
- **Imported segments** from S3
- **Behavioral**: users who opened email X, clicked Y, visited URL Z
- Use SQL-like filter expressions

## Analytics

- Delivery rate, bounce rate, open rate, click-through rate
- Funnel analytics: campaign → email open → click → conversion
- **Endpoint** = a destination per user per channel (email address, phone, device token)
- **User** = identity that may have multiple endpoints
- Export to S3 for deeper analysis (Athena / QuickSight)

## Pinpoint vs SES vs SNS vs Connect

| | Pinpoint | SES | SNS | Connect |
|---|---|---|---|---|
| Channel | Email + SMS + push + voice + in-app | Email only | Pub-sub + SMS + email + push | Voice + chat (contact center) |
| Segmentation | **Yes** (rich) | No | No | N/A |
| Journeys | **Yes** | No | No | Contact flows |
| Analytics | **Rich** (open, click, funnel) | Delivery only | Delivery only | Call metrics |
| Best for | Marketing campaigns + user engagement | Transactional email at scale | Pub-sub messaging, system notifications | Customer service / call center |

## Send Pattern Decision

- **Transactional email at scale** (no segmentation, no journey) → **SES** (cheaper)
- **One-off SMS / push to a topic** → **SNS**
- **Marketing campaigns, segmented + analytics + multi-channel** → **Pinpoint**
- **Customer service voice / chat** → **Connect**

## Recent Rebranding

AWS is splitting Pinpoint:
- **AWS End User Messaging SMS** (SMS / voice subset)
- **AWS End User Messaging Push** (push subset)
- **Pinpoint** core for campaigns + journeys + email
- Functionality unchanged; pricing & APIs rebranded per channel

## Integration

- **Lambda** event triggers Pinpoint campaign
- **Kinesis** + **Personalize** for ML-driven recommendations
- **S3** import / export segments
- **AppFlow** for CRM data sync
- **EventBridge** events on engagement
- **AWS Mobile SDK** for in-app endpoint registration

## Security

- IAM-based access
- TLS for delivery
- Sender authentication via SES (SPF / DKIM / DMARC) for email
- 10DLC / toll-free registration for US SMS

## Pricing

- **Per channel** + per message
- Email: ~$0.0001 per message (uses SES rates)
- SMS: varies by country ($0.00645 US originating, much higher internationally)
- Push: ~$1 per million
- Endpoints + monthly active users count toward tier
- Free tier: 5,000 targeted users, 1M push, 1M emails (via SES)

## Common Patterns

### Welcome series journey
- New user signup → Pinpoint event
- Day 0: welcome email
- Day 3: tip email
- Day 7: if not engaged → SMS check-in
- Visual journey editor in console

### Personalized recommendations
- User behavior → Kinesis → SageMaker / Personalize → segment update
- Targeted campaign sends product recommendation
- Track click → conversion attribution

### Multi-region transactional + marketing
- SES for transactional (receipts, password reset) — cheaper
- Pinpoint for marketing campaigns — segmentation + analytics
- Both use same verified domain

### 2FA via voice
- Login attempt → Lambda → Pinpoint voice channel
- TTS reads OTP to user phone
- Backup for users without SMS

## Exam Tips

- "Multi-channel customer engagement (email + SMS + push)" → **Amazon Pinpoint**
- "Marketing campaign with segmentation + analytics" → **Pinpoint**
- "Multi-step customer journey (welcome series, drip campaign)" → **Pinpoint Journeys**
- "Transactional email at scale, no segmentation" → **SES** (cheaper)
- "Send to a topic with subscribers" → **SNS** (pub-sub)
- "Voice call center" → **Amazon Connect**
- Channels: email, SMS, voice, push, in-app, custom

## Exam Traps

- **Pinpoint ≠ SES** — Pinpoint adds segmentation + journeys + analytics on top of SES email
- **Pinpoint ≠ SNS** — Pinpoint is engagement platform; SNS is generic pub-sub
- **Pinpoint ≠ Connect** — Connect is call center; Pinpoint is outbound marketing
- **Email uses SES under the hood** — SES rates apply (still verify sender domain)
- **SMS pricing varies massively by country** — US is cheap, international expensive
- **Endpoint vs User** — endpoint is a destination; user can have many endpoints
- **A/B testing requires sufficient sample** — small segments give noisy results
- **Pinpoint being split into separate services** — End User Messaging SMS / Push as separate products
- **Push needs SDK integration** — server-side alone can't deliver native push
