# Amazon CodeGuru

> **ML-powered code review + application performance profiling. Two components: CodeGuru Reviewer = automated code review for Java + Python (pull request bot, finds bugs + security issues + best-practice violations). CodeGuru Profiler = production runtime profiler that identifies CPU + memory hotspots in live apps. CodeGuru Security adds dedicated security scanning. Common SAP-C02 use: improve code quality + perf as a continuous improvement initiative.**

Maps to: **Domain 3.2 — Performance**, **Domain 3.4 — Improve deployments**, **Domain 3.3 — Improve security**

---

## Three Components

### CodeGuru Reviewer
- **Automated code review** powered by ML
- Languages: **Java, Python**
- Integrates with **GitHub, GitHub Enterprise, GitLab, Bitbucket, CodeCommit**
- Posts findings as pull request comments
- Categories: security, concurrency, AWS best practices, resource leaks, code maintainability, sensitive info leaks
- Trained on Amazon's internal codebase + open source

### CodeGuru Profiler
- **Runtime profiling** for live production apps
- Languages: **Java (JVM), Python, .NET (preview)**
- Continuous sampling: CPU usage, heap usage, latency
- Identifies **hot methods, frame graphs, anomalies**
- Recommendations: "this method consumes X% CPU; here's why"
- Low overhead (~< 1%)

### CodeGuru Security
- Dedicated **security scanning** for code
- Detects OWASP / CWE issues, hardcoded secrets, IaC misconfigurations
- Integrates with CI/CD (CodePipeline, GitHub Actions, Jenkins)
- Similar value to Snyk / Veracode but AWS-native

## CodeGuru Reviewer Details

### Supported Sources
- AWS CodeCommit (legacy — closed to new customers 2024)
- GitHub, GitHub Enterprise
- GitLab
- Bitbucket
- Amazon S3 zipped source

### Finding Categories
- **Security** — SQL injection, hardcoded secrets, AWS credential leaks
- **AWS best practices** — anti-patterns with AWS SDK usage (e.g., creating SDK client inside Lambda handler)
- **Concurrency** — race conditions, deadlocks
- **Resource leaks** — unclosed streams, connections
- **Sensitive information** — PII patterns
- **Maintainability** — overly complex methods, code duplication

### Workflow
- Connect repo
- On each PR: CodeGuru reviews changes + posts comments
- Developer addresses findings or marks false-positive (feeds back to ML)
- No human reviewer required for first-pass

## CodeGuru Profiler Details

### Deployment
- **Java**: agent JAR added to JVM startup
- **Python**: `import codeguru_profiler_agent` + start
- Profiles continuously sent to CodeGuru
- Console shows **frame graphs** + recommendations

### Frame Graphs
- Visualize call stack frequency
- Wider frame = more CPU time
- Click → drill into specific method
- Identify "I had no idea X consumed 30% CPU"

### Recommendations
- Specific suggestions: "Use `Arrays.asList()` instead of `new ArrayList<>()` for read-only lists"
- ML-derived from large codebase patterns

### Comparing Versions
- Compare profile between releases
- Detect perf regressions per deploy

## CodeGuru Security Details

- Scans on every commit / PR
- Output: CWE-tagged findings + remediation suggestions
- Track findings across repos in a dashboard
- Integrates with Security Hub + Amazon Inspector

## Comparison with Other AWS Tools

| Tool | Purpose |
|---|---|
| **CodeGuru Reviewer** | Code review (style, bugs, security in PRs) |
| **CodeGuru Profiler** | Runtime perf (CPU / memory hotspots) |
| **CodeGuru Security** | Security-focused code scan |
| **X-Ray** | Distributed tracing (latency across services) |
| **Inspector** | Vulnerability scan of EC2 / containers / Lambda runtime |
| **DevOps Guru** | ML-based operational anomaly detection from CloudWatch + X-Ray |
| **Lambda Power Tuning** | Lambda memory/cost optimizer (not AWS managed; community) |

**Common confusion**: Inspector scans **deployed artifacts** for known CVEs; CodeGuru Security scans **source code** for vulnerabilities. They complement.

## Integration Patterns

### CI/CD with PR review
- CodeGuru Reviewer attached to GitHub repo
- Every PR gets ML review + comments
- Engineers address before merge
- Combined with CodeBuild for build/test

### Continuous prod profiling
- CodeGuru Profiler agent in JVM-based microservices
- Console reviewed weekly for perf wins
- Quarterly perf-improvement OKR

### Security gate in pipeline
- CodeGuru Security in CodePipeline
- Fail pipeline if HIGH severity finding
- Force fix before deploy

## Pricing

### CodeGuru Reviewer
- $10 per 100 lines reviewed per month (full scan)
- $0.50 per 100 lines for incremental (PR)
- Free tier: 100,000 lines/month for first 90 days

### CodeGuru Profiler
- $0.005 per profiling group hour
- Free tier: first profiler hour per month per group

### CodeGuru Security
- $5 per 100,000 lines per scan

## Common Patterns

### Java microservice quality program
- Reviewer on PRs
- Profiler in prod
- Quarterly perf + code-quality OKRs measured
- Visible improvements over time

### Lambda function optimization
- Reviewer flags AWS SDK best practices (instantiate client outside handler)
- Profiler shows cold start vs warm time
- Combine: code-time + runtime optimization

### Pre-launch security gate
- CodeGuru Security in pre-prod pipeline
- Block deploy on HIGH findings
- Integrates with Security Hub for org-wide visibility

## Exam Tips

- "ML-powered automated code review on PRs" → **CodeGuru Reviewer**
- "Runtime performance profiling in production" → **CodeGuru Profiler**
- "Detect hardcoded secrets / SQL injection in source code" → **CodeGuru Security**
- Languages: Reviewer = **Java + Python**; Profiler = **Java + Python + .NET (preview)**
- Integrates with **GitHub / GitLab / Bitbucket / CodeCommit**
- Low-overhead profiling (~< 1%)

## Exam Traps

- **CodeGuru Reviewer ≠ CodeGuru Profiler** — Reviewer is static analysis on PRs; Profiler is runtime perf
- **Reviewer only supports Java + Python** — not all languages
- **Profiler overhead is low but non-zero** — disable in latency-critical Lambda hot paths if needed
- **CodeGuru Security ≠ Inspector** — Security scans source code; Inspector scans deployed artifacts for CVEs
- **CodeGuru does not test functionality** — pair with unit/integration tests; it finds patterns/perf, not correctness
- **Reviewer takes 10–20 minutes per PR** — can slow tight CI loops
- **Profiler needs explicit instrumentation** — not zero-config
- **CodeCommit is closed to new customers (2024)** — but CodeGuru works with GitHub, GitLab, Bitbucket
