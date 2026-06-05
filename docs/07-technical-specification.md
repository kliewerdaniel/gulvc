# 07 — Technical Specification

> The concrete technology choices that turn the architecture into a buildable system. Includes versions, rationale, alternatives, and migration policy.

---

## Purpose

This document specifies the technologies, frameworks, and infrastructure choices that implement the architecture defined in `02-system-architecture.md`. Each choice is justified, alternatives are recorded, and a deprecation policy is established.

A change to this document requires either:

- A minor revision (patch or compatible upgrade), or
- An ADR (when adopting or replacing a technology with significant migration cost)

---

## Scope

In scope:

- Language and runtime choices
- Frameworks and primary libraries
- Storage, queue, cache, and event technologies
- Cloud and infrastructure
- Observability stack
- Developer tooling
- Build, test, and deploy toolchain

Out of scope:

- Implementation details inside engines (covered in documents 10–14)
- API contracts (`09-api-specification.md`)
- Database schema (`08-database-design.md`)

---

## Choice Catalog (At a Glance)

| Concern | Choice | Version (pinned) |
|---------|--------|------------------|
| Backend language | Python | 3.12.x |
| Frontend / generation runtime | Node.js | 20 LTS (20.18+) |
| Frontend / generation language | TypeScript | 5.5.x |
| Backend HTTP framework | FastAPI | 0.115.x |
| Workflow runtime | Temporal | Server 1.25.x, Python SDK 1.9.x |
| Browser automation | Playwright (Python) | 1.49.x |
| HTML parsing | BeautifulSoup4 + lxml | 4.12.x / 5.3.x |
| LLM clients | OpenAI Python, Anthropic Python | latest stable |
| Web framework (frontend) | Next.js | 14.2.x (App Router) |
| Generation framework (output) | Next.js | 14.2.x (App Router) |
| CSS | Tailwind CSS | 3.4.x |
| Component primitives | Radix UI (frontend), generated only (output) | latest |
| ORM / DB toolkit (Python) | SQLAlchemy 2.x + Alembic | 2.0.x / 1.13.x |
| Validation | Pydantic | 2.9.x |
| Database | PostgreSQL | 16.x |
| Cache / rate limit | Redis | 7.4.x |
| Queue | Temporal (workflow) + SQS (DLQ) | — |
| Object storage | AWS S3 | — |
| Secret store | AWS Secrets Manager | — |
| Container runtime | Docker | 25.x |
| Container orchestration | AWS ECS Fargate | — |
| Infrastructure-as-Code | Terraform | 1.9.x |
| CI | GitHub Actions | — |
| Package manager (JS) | pnpm | 9.x |
| Package manager (Python) | uv | 0.4.x |
| Monorepo tooling | Turborepo + Nx (selective) | latest |
| Test runner (Python) | pytest | 8.x |
| Test runner (JS) | Vitest | 2.x |
| E2E test | Playwright | 1.49.x |
| Lint (Python) | Ruff | latest |
| Format (Python) | Ruff format | latest |
| Type check (Python) | mypy --strict | 1.11.x |
| Lint (JS) | ESLint | 9.x |
| Format (JS) | Prettier | 3.x |
| Observability — traces | OpenTelemetry | latest |
| Observability — logs | OpenTelemetry / Loki | latest |
| Observability — metrics | Prometheus / OpenTelemetry | latest |
| Observability — dashboard | Grafana | latest |
| Error tracking | Sentry | latest |
| Email | Resend | — |
| Payments | Stripe | — |
| Auth (customer) | Custom JWT + magic link (email) | — |
| Auth (operator) | SSO via WorkOS or Auth0 | — |

---

## Languages & Runtimes

### Python 3.12

**Used for:** API, orchestrator, capture worker, analysis worker, SEO worker, deployment worker, internal tooling.

**Rationale:**

- Best-in-class libraries for browser automation (Playwright), HTML parsing (BeautifulSoup, lxml), LLM clients, and data manipulation.
- Mature async I/O (`asyncio`) suitable for I/O-bound agent work.
- Strong typing via `mypy --strict` with Pydantic for runtime validation.
- Hiring: large talent pool with relevant ML/agent experience.

**Alternatives considered:**

- **Go**: better concurrency story but weaker LLM/agent ecosystem and harder hiring for AI roles.
- **TypeScript everywhere**: appealing for stack uniformity, but Python's data and ML libraries are decisive.

**Constraints:**

- Single-version policy: only one Python minor version is supported at a time in CI.
- All code must pass `mypy --strict` and `ruff check`.
- All public functions and classes must have type hints.

### Node.js 20 LTS

**Used for:** Frontend (Next.js), generated-app build and validation (inside the Generation Worker), local tooling.

**Rationale:**

- Required by Next.js.
- LTS line for predictable security support.
- Native fetch and modern ECMAScript features.

**Constraints:**

- Pin via `.nvmrc` and `engines` in `package.json`.
- All Node code is TypeScript (`strict: true`).

### TypeScript 5.5

**Used for:** Frontend, admin dashboard, generated apps, shared schema packages.

**Rationale:**

- Industry standard for typed web development.
- First-class Next.js support.
- Enables shared types between frontend and backend via codegen (OpenAPI → TS).

**Constraints:**

- `tsconfig.json` shared via `@vibe/tsconfig` package; per-app overrides are minimal.

---

## Backend Frameworks

### FastAPI 0.115

**Used for:** Public API, admin API.

**Rationale:**

- Native Pydantic integration aligns with the typed-contract principle.
- Async first; matches I/O-bound workload.
- Generates OpenAPI 3.1 automatically.
- Excellent developer ergonomics.

**Alternatives considered:**

- **Litestar**: comparable, slightly smaller community.
- **Django REST Framework**: heavier, sync-first, less aligned with async/agent workloads.

### Temporal (Server 1.25, Python SDK 1.9)

**Used for:** Job orchestration. See ADR-005.

**Rationale:**

- Durable, versioned workflows.
- Built-in retries, timeouts, signals, queries, sagas.
- First-class Python SDK.
- Avoids reinventing distributed state machines.

**Deployment options:**

- **MVP**: Temporal Cloud (managed) to minimize ops burden.
- **V2+**: Self-hosted Temporal cluster on EKS for cost control at scale, optional.

**Constraints:**

- All workflows declare a version constant. Breaking changes increment the version and use patching for in-flight workflows.

### Playwright (Python) 1.49

**Used for:** Capture worker. See ADR-003.

**Rationale:**

- Cross-browser (Chromium, Firefox, WebKit).
- Built-in HAR recording, screenshot, and tracing.
- Asynchronous Python API matches FastAPI / asyncio stack.

**Alternatives considered:**

- **Puppeteer**: Chrome-only, JS-only.
- **Selenium**: older API, weaker HAR support.

---

## Frontend Frameworks

### Next.js 14.2 (App Router)

**Used for:**

1. The Vibe marketing site and customer dashboard (deployed on Vercel).
2. The admin console (deployed on Vercel under a separate project).
3. The **output** of the generation engine.

**Rationale:** See ADR-002.

- React Server Components match SEO and performance goals.
- Vercel deployment is industry-leading for Next.js.
- Image, font, and metadata APIs are first-class.
- App Router supports the static and dynamic rendering mix the generated sites require.

**Constraints:**

- App Router only (not Pages Router) for new code.
- React 18.3+.
- All components are server components by default; client components are explicit.

### Tailwind CSS 3.4

**Used for:** All UI styling, including generated apps.

**Rationale:**

- Predictable utility classes are easy to generate programmatically.
- Eliminates the need for a custom CSS generator inside the generation engine.
- Small runtime footprint.

**Alternatives considered:**

- **CSS Modules**: harder to generate consistently from agents.
- **Styled Components**: runtime cost, SSR friction.

### Radix UI (frontend only)

**Used for:** Customer dashboard and admin console primitives.

**Rationale:** Accessible, unstyled primitives. Pairs with Tailwind. Avoids reinventing dialog, popover, menu, etc.

**Note:** Generated apps do not depend on Radix; they receive plain semantic HTML so customers can edit freely.

---

## Storage Layer

### PostgreSQL 16

**Used for:** All transactional state: users, tenants, jobs, billing records, audit logs, agent runs.

**Rationale:** See ADR-004 (and the database design doc).

- ACID transactions.
- Mature JSONB for semi-structured payloads (`SiteModel` summaries, agent traces).
- Strong tooling: SQLAlchemy 2.x, Alembic migrations, pgBouncer connection pooling.
- Multi-AZ on AWS RDS.

**Constraints:**

- Single primary write node; read replicas as needed.
- All schema changes via Alembic migrations, reviewed in PR.
- No raw SQL in application code outside the data layer; use ORM or explicit Core constructs.
- No `JSONB` for fields that are queried frequently; use normalized columns.

### Redis 7.4

**Used for:**

- Rate limiting (per-tenant, per-route token buckets).
- Short-lived caching (LLM response cache, free-preview cache).
- Distributed locks (e.g., scheduler leadership).
- WebSocket pub/sub (dashboard real-time updates).

**Rationale:**

- Universally adopted.
- Managed via ElastiCache.

**Constraints:**

- No long-lived state in Redis; PostgreSQL is the source of truth.
- All keys are namespaced with `vibe:{env}:{purpose}:`.

### AWS S3

**Used for:** Artifacts (HAR, screenshots, generated workspaces, reports), backups.

**Rationale:** Native durability, lifecycle policies, versioning, cost.

**Constraints:**

- Buckets are encrypted with KMS keys per environment.
- Tenant data is prefixed by `tenant_id` for IAM-based isolation.
- Lifecycle policies enforce retention NFRs.

---

## Queue & Workflow Layer

### Temporal (primary)

- Owns durable execution of all jobs.
- Signals, queries, and saga compensations.

### AWS SQS (fallback / DLQ)

- Dead-letter queue for webhooks and notification retries.
- Used for non-workflow background tasks (e.g., daily roll-ups).

---

## Event Layer

### Internal Domain Events

The platform emits domain events on key transitions (`JobQueued`, `JobCaptured`, `JobDelivered`, etc.). MVP delivers events via:

- A Postgres outbox table.
- A poller that publishes to Redis pub/sub (for real-time UI) and to webhook subscribers.

V3 introduces a Kafka or Kinesis-backed event bus when analytics consumers proliferate.

---

## Cloud & Infrastructure

### AWS (primary cloud)

**Region:** `us-east-1` (MVP). Secondary capture region `eu-west-1` (V3).

**Services used:**

- ECS Fargate (compute)
- RDS for PostgreSQL (Multi-AZ)
- ElastiCache for Redis
- S3 (artifacts)
- Secrets Manager
- KMS (key management)
- CloudWatch (some logs and metrics; not primary)
- Route 53 (DNS)
- ALB (load balancing)
- VPC, IAM, CloudTrail

**Rationale:** Maturity, talent availability, ecosystem.

**Alternatives considered:**

- **GCP**: comparable, smaller talent pool internally.
- **Vercel-only**: insufficient for browser pools and long-running workflows.

### Vercel (frontend hosting)

**Used for:** Web frontend, admin console, and the **generated customer sites**. See ADR-004.

**Rationale:** Best Next.js host. Aligns generation target with delivery platform.

---

## Infrastructure-as-Code

### Terraform 1.9

**Used for:** All AWS resources, Cloudflare records (if used), Vercel projects (via provider).

**Structure:**

- `infra/modules/` — reusable modules
- `infra/envs/{dev,staging,prod}/` — environment-specific compositions
- State stored in S3 with DynamoDB locking

**Constraints:**

- No clickops in production AWS.
- All Terraform changes go through PR + plan review.
- Atlantis or Terraform Cloud for plan/apply automation.

---

## Developer Tooling

### Package Managers

- **pnpm 9** for JavaScript / TypeScript. Monorepo workspaces.
- **uv 0.4** for Python. Fast resolver, lockfile, workspace-aware.

### Monorepo

- **Turborepo** for task pipelines and remote caching.
- **Nx** considered for advanced module boundaries; introduced selectively if needed.

### Lint, Format, Type-check

- **Ruff** (lint + format) for Python.
- **mypy --strict** for Python.
- **ESLint 9** flat config for JS/TS.
- **Prettier 3** for JS/TS formatting.
- **tsc --strict** for TypeScript.

All enforced in CI; pre-commit hooks via Lefthook or pre-commit.

### Testing

- **pytest 8** + **pytest-asyncio** + **pytest-cov** for Python.
- **Vitest 2** for JS/TS unit tests.
- **Playwright 1.49** for E2E (also used by the capture worker; shared dependency policy in ADR-003).
- **Schemathesis** for API property-based testing against the OpenAPI spec.

---

## Observability Stack

| Concern | Tool |
|---------|------|
| Distributed tracing | OpenTelemetry SDK → OTLP → Grafana Tempo |
| Metrics | OpenTelemetry SDK / Prometheus exporter → Mimir |
| Logs | OpenTelemetry / structured JSON → Loki |
| Dashboards | Grafana |
| Alerting | Grafana Alerting / PagerDuty |
| Error tracking | Sentry |
| Synthetic monitoring | Checkly or Better Uptime |
| Real-user metrics | Vercel Analytics (frontend) + custom (delivered sites, opt-in) |

**Rationale:** OTel-first means we can self-host or migrate to Grafana Cloud / Datadog / Honeycomb without rewriting instrumentation.

See `18-observability.md`.

---

## Third-Party Services

| Service | Purpose | Failover |
|---------|---------|----------|
| Stripe | Payments | None (single source). Mitigation: graceful degradation, manual processing. |
| Resend | Transactional email | Postmark as secondary. |
| OpenAI | LLM (primary) | Anthropic as secondary. |
| Anthropic | LLM (secondary or task-specialized) | OpenAI as fallback. |
| GitHub | Code hosting | None for code; mitigation is queueing and retry. |
| Vercel | Deployment | None; mitigation is queueing and retry. |
| Cloudflare (optional) | DNS, CDN, WAF | Native AWS equivalents available. |
| WorkOS (V2+) | SSO for operators | Self-service password fallback. |

---

## Versioning & Upgrade Policy

- **Patch upgrades** (security fixes): applied automatically via Renovate within 7 days.
- **Minor upgrades**: applied within 30 days after CI passes.
- **Major upgrades**: require an issue, a migration plan, and an ADR if user-visible.
- **End-of-life software**: must be migrated off before EOL. Tracked in the dependency register.

---

## Local Development Requirements

See `26-local-development.md` for the full setup. Briefly:

- macOS, Linux, or WSL2.
- Docker Desktop or OrbStack.
- Python 3.12 via `mise` or `pyenv`.
- Node 20 via `mise` or `nvm`.
- `uv`, `pnpm`, `turbo` installed globally.
- `make bootstrap` provisions everything.

---

## Why These Choices Together

The stack composition reinforces itself:

- **Python + FastAPI + Pydantic + Temporal + Playwright** give us a typed, async, durable agent platform.
- **Next.js + Tailwind + TypeScript** give us a consistent output target that also powers our own UI.
- **PostgreSQL + Redis + S3** are boring, durable, well-understood.
- **OpenTelemetry-first** gives us vendor portability.
- **Terraform + GitHub Actions** give us reproducible infrastructure and CI/CD.

Each choice is replaceable in isolation if requirements shift.

---

## Assumptions

- The team is comfortable operating a small AWS footprint without a dedicated SRE.
- Temporal Cloud's pricing is acceptable for MVP volumes.
- LLM providers remain commercially accessible.
- Vercel's pricing model remains compatible with our COGS targets.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Two-language stack (Python + TypeScript) | Best-in-class for backend agents and frontend respectively. |
| Temporal over custom state machines | Reliability and developer velocity. |
| Pin minor versions, automate patch upgrades | Predictability with security freshness. |
| OpenTelemetry as the only instrumentation API | Vendor portability. |
| pnpm + uv (not npm or pip) | Speed, determinism, lockfile quality. |
| Tailwind in both UI and output | Eliminates a CSS generator dependency. |

---

## Open Questions

- Should the marketing site share a workspace with the customer dashboard or be its own app?
- Should we self-host Temporal at V2 or stay on Temporal Cloud?
- Should we adopt Cloudflare for WAF and CDN in front of AWS, or use AWS-native equivalents?
- Should we standardize on `uv` for Python or remain compatible with `pip-tools`?

---

## Future Enhancements

- Multi-region active-active for the API and capture workers.
- Edge-rendered customer dashboard pieces (e.g., status page) on Vercel Edge.
- A SDK in TypeScript and Python for partner integrations (V3).
- A graph database (Neo4j) for cross-job pattern mining (V3+).

---

## Cross-References

- Architecture → `02-system-architecture.md`
- ADRs → `ADR/`
- Monorepo layout → `22-monorepo-structure.md`
- Local dev → `26-local-development.md`
- Coding standards → `21-coding-standards.md`
