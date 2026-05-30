# AWS Amplify

> **End-to-end platform for full-stack web + mobile apps. Combines Amplify Hosting (CI/CD-driven static site + SSR hosting with edge CDN), Amplify Backend (declarative backend with auth, API, storage, functions) and Amplify Studio (visual UI). Generates code for AppSync (GraphQL), Cognito (auth), S3 (storage), Lambda (functions), DynamoDB. Common SAP-C02 use case: modern web/mobile app backend with minimal AWS expertise. Often confused with Elastic Beanstalk (server-based) or App Runner (container-based) — Amplify is for serverless full-stack.**

Maps to: **Domain 4.3 — Modernization**, **Domain 2.5 — High-performance**, **Domain 4.4 — Modernization opportunities**

---

## Three Components

### Amplify Hosting
- **Static site + SSR hosting** with CI/CD from Git
- Branches map to environments (PR previews, main = prod)
- Atomic deploys + instant rollback
- **CloudFront-backed CDN** at the edge
- Custom domains + HTTPS via ACM
- Supports Next.js (SSR/ISR), Nuxt, Astro, Gatsby, plain SPAs

### Amplify Backend
- **Declarative backend** code-first (TypeScript schema)
- Generates infrastructure: AppSync (GraphQL), Cognito, S3, Lambda, DynamoDB
- **Amplify Gen 2** uses TypeScript as IaC; older Gen 1 used CloudFormation directly
- Local dev environments + cloud sandboxes
- Outputs CloudFormation under the hood

### Amplify Studio
- **Visual UI builder** for components (or Figma-to-code)
- Generate React components from designs
- Manage data models + auth from web console

## Use Cases

- **Modern web apps** (React, Vue, Angular, Svelte, Next.js)
- **Mobile apps** (iOS, Android, React Native, Flutter — via Amplify Libraries)
- **Full-stack TypeScript apps** (Gen 2)
- **PR preview environments** automatically per branch
- **Static marketing sites** with CDN

## Hosting Features

- **Branch deployments** — each branch = environment
- **Preview deployments** per PR
- **Atomic deploys** — full bundle uploaded then switched
- **Instant rollback** to previous deploy
- **Password-protected branches** (basic auth)
- **Custom redirects + rewrites** (for SPA routing or SSR)
- **Server-side rendering** for Next.js (SSR + ISR)
- **Custom headers** + cache control
- **Monorepo support**

## Backend Capabilities

- **GraphQL API** — AppSync with schema-driven resolvers
- **REST API** — API Gateway + Lambda
- **Authentication** — Cognito User Pools + social IdPs
- **Storage** — S3 buckets with per-user access
- **Functions** — Lambda (Node.js, Python)
- **Database** — DynamoDB (managed via Amplify schema)
- **AI/ML** — Predictions category integrates Rekognition / Polly / Translate / Comprehend
- **Geo** — Amazon Location Service integration
- **Analytics** — Pinpoint integration

## Amplify Gen 1 vs Gen 2

| | Gen 1 (legacy) | Gen 2 |
|---|---|---|
| IaC | Amplify CLI + CloudFormation | TypeScript schema |
| Local dev | Amplify CLI commands | Cloud sandbox per dev |
| Backend definition | JSON / GraphQL schema | TypeScript types + resolvers |
| Recommended for new apps | No | **Yes** |

Gen 2 is **TypeScript-as-IaC** — more developer-friendly for full-stack TS apps.

## Amplify vs Elastic Beanstalk vs App Runner

| | Amplify | Elastic Beanstalk | App Runner |
|---|---|---|---|
| Workload | Static + SSR + serverless backend | Server app (web tier) | Container web app |
| Source | Git → static build + IaC | Git → EC2 / Docker | Git / ECR → container |
| Backend included | **Yes (Cognito / AppSync / S3 / Lambda)** | No (just app server) | No (just container) |
| Best for | Modern full-stack apps | Lift-and-shift web apps | Container web apps with auto-scale |
| Scaling | Edge (CDN) + serverless | EC2 ASG | Auto request-based |

**Rule of thumb**: new full-stack web/mobile app → **Amplify**. Existing Java/Python web tier → **Beanstalk**. Containerized stateless web service → **App Runner**.

## Amplify vs CloudFront + S3 Static Hosting

- **Plain CloudFront + S3** = manual CI/CD + manual cache invalidation + manual cert / domain
- **Amplify** = git-driven CI/CD + atomic deploys + PR previews + SSR + backend integration in one
- For trivial static sites: CloudFront + S3 is cheaper
- For team workflow + SSR + backend: **Amplify**

## Multi-Region / DR

- **Amplify Hosting** uses CloudFront — globally distributed by default
- **Backend resources** are Region-pinned (Cognito, AppSync, DynamoDB)
- DR strategy: per-region stack with DynamoDB Global Tables + Cognito multi-region (limited)

## Pricing

- **Hosting**: ~$0.01 per build-minute, $0.023 per GB stored, $0.15 per GB served (after free tier)
- **Backend**: pay underlying services (AppSync, Cognito, Lambda, DynamoDB) at normal rates
- **Studio**: free
- Free tier covers small apps

## Common Patterns

### Marketing site with PR previews
- Push to `main` → prod deploy
- Push to feature branch → preview URL per branch
- Marketing reviewers see changes before merge

### Mobile app backend
- Amplify Gen 2 TypeScript schema: User, Post, Comment models
- Generates Cognito + AppSync + DynamoDB automatically
- iOS / Android client uses Amplify Libraries

### SSR Next.js with Cognito
- Amplify Hosting with Next.js SSR
- Backend: Cognito + AppSync data layer
- Each PR gets staging URL with isolated backend sandbox

### Migrate from Vercel / Netlify
- Pull Git repo into Amplify Hosting
- Configure build settings
- Migrate auth from Auth0 → Cognito via Amplify Backend

## Exam Tips

- "End-to-end serverless full-stack platform" → **AWS Amplify**
- "Git-driven CI/CD with PR preview environments for web app" → **Amplify Hosting**
- "Generate Cognito + AppSync + DynamoDB backend from TypeScript schema" → **Amplify Gen 2**
- "Mobile app backend with offline sync" → **Amplify Libraries + AppSync**
- Edge CDN: CloudFront-backed
- Gen 2 uses TypeScript as IaC

## Exam Traps

- **Amplify ≠ Elastic Beanstalk** — Amplify is serverless full-stack; Beanstalk is server-based web tier
- **Amplify ≠ App Runner** — App Runner is container web app; Amplify is full-stack with backend
- **Amplify Hosting alone ≠ Amplify Backend** — you can use just hosting (static + SSR) without the backend
- **Backend is Region-pinned** — hosting is global (CloudFront) but Cognito / AppSync are not
- **Gen 1 and Gen 2 are different** — don't mix; new projects use Gen 2
- **SSR support is for Next.js (and some others)** — not all frameworks
- **Cognito is the auth layer** — not IAM (developers often confuse these)
- **Amplify-generated CloudFormation can be drift-modified manually** — but Amplify will overwrite on next deploy
