# Project Ledger — Go Up Level Vibe Coding

> Living checklist of every task required to build the platform defined in `/docs/`.
> Conventions: `[ ]` pending, `[x]` complete, `[~]` in progress, `[!]` blocked, `[skip]` deferred.
> Each item references the doc and (where applicable) user-story ID it derives from.
> Update this file as work is completed; it is the single source of truth for status.

---

## Status Snapshot

| Phase | Status | Notes |
|-------|--------|-------|
| M0 — Foundations | Not started | First target. Monorepo, CI, local dev, security baseline. |
| M1 — Capture | Not started | URL → manifest. |
| M2 — Analysis | Not started | Manifest → SiteModel. |
| M3 — Generation (single-page) | Not started | SiteModel → Next.js. |
| M4 — SEO + Quality | Not started | Metadata, Lighthouse, llms.txt. |
| M5 — Deployment | Not started | GitHub + Vercel. |
| M6 — Reporting + Delivery | Not started | Reports, webhooks, notifications. |
| M7 — Self-Serve Billing | Not started | Stripe, plans, quotas. |
| M8 — Public API | Not started | REST + SDK. |
| M9 — Multi-Page Sites | Not started | Full nav generation. |
| M10 — Whitelabel + SSO | Not started | V2. |
| M11 — Migration Path | Not started | V2. |
| M12 — Continuous Improvement | Not started | V2. |
| M13 — Autonomous Fleet | Not started | V3. |

---

## 0. Bootstrap & Tracking (M0, immediate)

- [ ] Create this ledger (`LEDGER.md`) and commit.
- [ ] Add `.gitignore` (root) — Node, Python, IDE, OS, secrets, build outputs.
- [ ] Add `CODEOWNERS` stub.
- [ ] Add `CONTRIBUTING.md` linking to `/docs/00..29`.
- [ ] Add `LICENSE` (project default; confirm with owner before publishing).
- [ ] Add `.editorconfig`.
- [ ] Add `.gitattributes` (line endings, linguist overrides).
- [ ] Pin required tool versions in `/.tool-versions` (asdf-compatible) and a `Makefile` `doctor` target that checks them.

References: `docs/22-monorepo-structure.md`, `docs/26-local-development.md`, `docs/20-developer-workflows.md`.

---

## 1. M0 — Foundations

### 1.1 Monorepo Skeleton (`docs/22-monorepo-structure.md`)

- [ ] Create `apps/` directory with planned app folders (empty placeholders): `api/`, `web/`, `capture-worker/`, `analysis-worker/`, `generation-worker/`, `deploy-worker/`, `delivery-worker/`, `billing-worker/`, `scheduler/`, `notifier/`.
- [ ] Create `packages/` directory with planned package folders: `ts/contracts`, `ts/ui`, `ts/sdk`, `ts/tokens`, `ts/tsconfig`, `py/vibe_core`, `py/vibe_models`, `py/vibe_db`, `py/vibe_observability`, `py/vibe_auth`, `py/vibe_billing`, `py/vibe_integrations`, `py/vibe_test`.
- [ ] Create `services/` directory: `browser-pool/`, `llm-router/`, `rate-limiter/`, `feature-flags/`.
- [ ] Create `infra/` directory: `terraform/{modules,envs/{staging,prod}}`, `grafana/{dashboards,alerts}`, `perf/`, `ci/`, `threat-model/`.
- [ ] Create `schemas/` directory: `api/`, `events/`, `llm/`, `site-model/`, `migration/`.
- [ ] Create `tests/` directory: `fixtures/{sites,llm,git}`, `e2e/`, `perf/`, `fuzz/`.
- [ ] Create `tools/` directory: `cli/`, `codegen/`.
- [ ] Add `pnpm-workspace.yaml` listing TS workspaces.
- [ ] Add root `package.json` with `name`, `private: true`, workspace scripts, and `packageManager` pin.
- [ ] Add root `pyproject.toml` (uv workspace) listing Python members.
- [ ] Add `turbo.json` with `build`, `lint`, `typecheck`, `test` pipelines and `dependsOn: ["^build"]`.
- [ ] Add root `Makefile` with targets: `help`, `bootstrap`, `up`, `down`, `test`, `lint`, `typecheck`, `format`, `format-check`, `env`, `doctor`, `db-reset`, `db-shell`, `migrate`, `migration`, `run`, `logs`, `e2e`, `fixtures`, `mocks`, `minio-reset`, `docker-prune`, `clean`.
- [ ] Add `.env.example` with all env vars enumerated in `docs/26-local-development.md` § Environment Configuration.
- [ ] Add `docker-compose.yml` with services: `postgres:16`, `redis:7.4`, `minio/minio`, `temporalio/auto-setup`, `mailhog/mailhog`, `traefik:v3`, `llm-mock`, `git-mock`, `vercel-mock`, `stripe/stripe-mock`.
- [ ] Add `Dockerfile` template for Python apps and a separate template for TS apps.
- [ ] Add `OWNERS` placeholder in each new directory.
- [ ] Add `README.md` updates pointing at `/docs/00-project-overview.md` and the Makefile.

### 1.2 Local Development (`docs/26-local-development.md`)

- [ ] Implement `make doctor` — verifies Node, pnpm, Python, uv, Docker, Docker Compose, ports, secrets.
- [ ] Implement `make bootstrap` — runs doctor, installs pre-commit, `pnpm install`, `uv sync`, `turbo run build`, `docker compose up -d`, migrations, seed.
- [ ] Implement `make up` / `make down` — bring up/tear down the local stack.
- [ ] Implement `make env` — generate `.env.local` from template with safe defaults.
- [ ] Implement `make db-reset`, `make db-shell`, `make migrate`, `make migration name="…"`.
- [ ] Implement `make test`, `make lint`, `make typecheck`, `make format`, `make format-check`.
- [ ] Implement `make run app=<name>` and `make logs svc=<name>`.
- [ ] Implement `make e2e`, `make perf`, `make fixtures`, `make mocks`.
- [ ] Implement `make debug app=<name>` (Python: `debugpy` on 5678; TS: `--inspect=9229`).
- [ ] Add `.vscode/settings.json` and `.vscode/extensions.json` recommendations from `docs/26-local-development.md`.
- [ ] Create `tools/cli/` scaffold (Typer + Rich) for the `vibe` CLI: `vibe --help`, `vibe env doctor`, `vibe db seed`, placeholders for `jobs run`, `jobs list`, `agents replay`, `docs serve`, `token mint`, `webhook trigger`.
- [ ] Confirm `make bootstrap` lands a contributor at the published local URLs: API `:8080`, Web `:3000`, Temporal UI `:8233`, Postgres `:5432`, Redis `:6379`, MinIO console `:9001`, MailHog `:8025`, Traefik `:81`.

### 1.3 Pre-commit, Lint, Format, Type, Test Tooling

- [ ] Add `.pre-commit-config.yaml` with hooks: `ruff format`, `ruff check --fix`, `biome format`, `biome lint --apply`, `mypy` (fast subset), `tsc --noEmit` (fast subset), `gitleaks`, custom `vibe check` for tenant-filter guards.
- [ ] Add `biome.json` (formatter + linter) at repo root with line-length 100, single quotes, trailing commas, import organization.
- [ ] Add `ruff.toml` at repo root (line-length 100, double quotes, target py312).
- [ ] Add `mypy.ini` (`strict = true`, `python_version = 3.12`).
- [ ] Add `tsconfig.base.json` (strict, noUncheckedIndexedAccess, exactOptionalPropertyTypes, noImplicitOverride, noFallthroughCasesInSwitch, target ES2022).
- [ ] Add `vitest.config.ts` for TS packages.
- [ ] Add `pytest.ini` and `conftest.py` (root) with shared fixtures.
- [ ] Add `dependency-cruiser` config enforcing `apps → services → packages → schemas` for TS.
- [ ] Add `import-linter` config enforcing the same for Python.
- [ ] Wire all linters/typecheckers/test runners into `turbo.json` and the Makefile.

### 1.4 CI/CD (`.github/workflows/`)

- [ ] Add `ci.yml` — on PR: pre-commit, lint, typecheck, test, build (affected), dependency-rule check, secret scan, `pip-audit`, `pnpm audit`.
- [ ] Add `ci-nightly.yml` — clean-environment compile + smoke tests.
- [ ] Add `release.yml` — changesets-driven release PR; on tag, publish internal packages and SDK.
- [ ] Add `codeql.yml` — code scanning on push and weekly schedule.
- [ ] Add `docker.yml` — build and push images for affected apps on `main`; tag on release.
- [ ] Add `infra-plan.yml` — `terraform plan` for changed envs on PR.
- [ ] Add `e2e.yml` — scheduled + manual dispatch for end-to-end job runs against a staging env.
- [ ] Add `dependency-review.yml` — runs on PRs that modify manifests.
- [ ] Add `renovate.json` for routine dependency updates and Dependabot-style vulnerability PRs.

### 1.5 Security Baseline (`docs/17-security-model.md`, `docs/25-risk-analysis.md`)

- [ ] Author STRIDE threat model skeleton under `infra/threat-model/` (M0 scope: intake, capture).
- [ ] Add `SECURITY.md` with reporting instructions.
- [ ] Configure secret scanning (gitleaks) in pre-commit and CI.
- [ ] Establish secrets-management approach (env-driven locally; AWS Secrets Manager in staging/prod).
- [ ] Add `LICENSE`-header check to CI (open-source hygiene rule).
- [ ] Add an RLS-ready database schema baseline (even if not used yet, columns/`tenant_id` conventions established) — see `08-database-design.md`.
- [ ] Add audit-log table stubs (`audit_events`).

### 1.6 Observability Baseline (`docs/18-observability.md`)

- [ ] Stand up OpenTelemetry SDK wrappers in `packages/py/vibe_observability`.
- [ ] Stand up Grafana dashboards skeleton in `infra/grafana/dashboards/` for: API, capture, analysis, generation, deploy (each with traces, metrics, logs).
- [ ] Add SLO dashboard skeleton (burn-rate, error budget) per service.
- [ ] Add a per-job trace viewer (links by `job_id`, `tenant_id` span attributes).
- [ ] Add structured logging (JSON) via `structlog` with `job_id`/`tenant_id` context binding.
- [ ] Configure local Tempo (`http://localhost:3200`) and `vibe job trace <id>`.

### 1.7 Database & Schemas (`docs/08-database-design.md`)

- [ ] Create Postgres schema for the canonical tables: `tenants`, `users`, `members`, `jobs`, `job_events`, `artifacts`, `audit_events`, `api_keys`, `webhook_subscriptions`, `webhook_deliveries`, `plans`, `subscriptions`, `usage_records`.
- [ ] Add `tenant_id uuid not null` to every multi-tenant table and a default RLS policy plan.
- [ ] Create Alembic scaffolding under `packages/py/vibe_db` with `alembic.ini`, `env.py`, `script.py.mako`, and an initial migration `0001_init.py`.
- [ ] Add `make migration name=…` that uses Alembic autogenerate against the running dev DB.
- [ ] Add `schemas/site-model/` with the v1 JSON Schema for `SiteModel` (start with pages[], navigation[], seo{} per README example).
- [ ] Add `schemas/api/openapi.yaml` placeholder.
- [ ] Add `schemas/events/` with the canonical domain event shapes.
- [ ] Add `schemas/llm/` with tool-call and structured-output shapes.

### 1.8 API Skeleton (`docs/09-api-specification.md`)

- [ ] Scaffold `apps/api/` with FastAPI, Pydantic v2, structured routing.
- [ ] Add `auth/` middleware (JWT for customers, API key for partners) — stub with allowlist-based dev auth.
- [ ] Add `rate_limit/` (token-bucket in Redis) — stub.
- [ ] Add `validators/` using Pydantic.
- [ ] Add `controllers/`: `jobs.py`, `reports.py`, `billing.py`, `webhooks.py` — minimal endpoints (POST/GET/cancel) returning 501 until wired.
- [ ] Add `tests/` for each controller.
- [ ] Add `/healthz`, `/readyz`, `/version`.
- [ ] Generate `openapi.json` at build time; serve Swagger UI at `/docs`.

### 1.9 Web Dashboard Skeleton (`docs/01..03` + ADR-002)

- [ ] Scaffold `apps/web/` with Next.js 14 App Router, TypeScript strict, Tailwind 3.4.
- [ ] Add routes: `/` (marketing), `/login`, `/dashboard`, `/dashboard/jobs`, `/dashboard/jobs/[id]`, `/dashboard/settings`, `/pricing`, `/legal/privacy`, `/legal/terms`.
- [ ] Add a job history list, a per-job timeline placeholder, and a settings placeholder.
- [ ] Add a status page (`/status`) with manual-update JSON.
- [ ] Add a magic-link login flow against the API (dev mode uses MailHog).
- [ ] Add `packages/ts/ui` and `packages/ts/tokens` with shared component library + design tokens.
- [ ] Add `packages/ts/contracts` for generated API types (from OpenAPI).

### 1.10 Orchestrator Skeleton (`docs/02-system-architecture.md`, ADR-005)

- [ ] Decide Temporal self-host vs. Temporal Cloud for staging (write the decision as `ADR-006`).
- [ ] Add `apps/orchestrator/` Python scaffold with a Temporal worker entrypoint.
- [ ] Define the `JobWorkflow` signature with placeholder activities: `capture`, `analyze`, `generate`, `seo`, `quality_review`, `deploy`, `deliver`.
- [ ] Add per-activity retry/timeout configuration in code.
- [ ] Add a saga-style compensation registry skeleton.
- [ ] Wire versioned workflow definitions.
- [ ] Confirm `Temporal UI` reachable at `http://localhost:8233` after `make up`.

### 1.11 Pricing & Plans (inert)

- [ ] Define plan tiers in `packages/py/vibe_billing/plans.py`: `free_preview`, `starter`, `growth`, `pro`, `enterprise` (no Stripe wiring yet).
- [ ] Add a quota config table (jobs/month, LLM spend cap, capture pages).
- [ ] Document in `docs/01-product-requirements-document.md` and link from `/pricing`.

### 1.12 M0 Exit Criteria

- [ ] A new contributor can `git clone`, `make bootstrap`, and see "hello job" within 30 minutes.
- [ ] All CI gates green on `main`.
- [ ] Security review of the baseline passes (documented sign-off in `docs/adr/CHANGELOG.md`).
- [ ] Architecture review of the baseline passes.

---

## 2. M1 — Capture (`docs/10-capture-engine.md`, `docs/24-backlog.md` EP-CAPTURE)

### 2.1 Engines & Workers

- [ ] Implement `apps/capture-worker/` Python service with a Temporal activity handler.
- [ ] Implement URL Normalizer (lowercase host, strip default ports, remove tracking params, resolve relative segments).
- [ ] Implement Policy Gate: RFC 1918 refusal, denylist check, tenant quota check, `robots.txt` fetch+parse.
- [ ] Implement Browser Session Manager (Playwright Chromium pool, isolated contexts).
- [ ] Implement Page Workflow: HAR recording, screenshot pipeline, DOM snapshot, link discoverer, asset downloader.
- [ ] Implement Crawl Queue with depth/page/time/asset caps.
- [ ] Implement Manifest Builder producing `CaptureManifest` (pages[], assets[], nav, screenshots, HARs).
- [ ] Implement `services/browser-pool/` Playwright pool manager (or embed in worker; document the decision).
- [ ] Persist artifacts to S3 with tenant prefix.

### 2.2 User Stories (EP-CAPTURE)

- [ ] **US-CAP-001** Capture a single URL with SSRF refusal (5 pts).
- [ ] **US-CAP-002** *(M9, listed here for tracking)* Crawl a site within a domain (8 pts).
- [ ] **US-CAP-003** Respect `robots.txt` (3 pts).
- [ ] **US-CAP-004** Detect bot walls; mark manifest partial; one retry with new fingerprint (5 pts).
- [ ] **US-CAP-005** Enforce per-job limits (15 min, 25 MB/page, 100 pages) (3 pts).
- [ ] **US-CAP-006** Per-asset capture metadata (URL, type, size, hash, license hint) (3 pts).
- [ ] **US-CAP-007** *(M9)* Manual URL additions (3 pts).
- [ ] **US-CAP-008** Capture progress visibility in the dashboard (3 pts).

### 2.3 M1 Exit Criteria

- [ ] 95% of a representative set of marketing sites capture to a valid manifest.
- [ ] Bot-walled sites recognized and reported as partial.
- [ ] No SSRF or egress policy violations in adversarial tests.
- [ ] p95 latency for a 30-page capture ≤ 10 minutes.
- [ ] Fixture sites committed under `tests/fixtures/sites/`.

---

## 3. M2 — Analysis (`docs/11-analysis-engine.md`, EP-ANALYSIS)

- [ ] Implement `apps/analysis-worker/`.
- [ ] HTML/CSS parser; content extraction (headings, paragraphs, lists, tables, images, alt flags, language/locale).
- [ ] Page role classifier (`home`, `product`, `service`, `contact`, `blog`, `about`, `legal`, `other`) with confidence.
- [ ] Navigation graph builder (cycles broken by canonical URL, hub pages identified).
- [ ] SEO scanner (title, meta, headings, alt text, internal links, schema.org).
- [ ] A11y scanner (axe-core runner, normalized to WCAG 2.2 AA).
- [ ] Tech stack detector (CMS, framework, analytics, fonts, hosting hints, complexity score).
- [ ] LLM summarizer with cost guard; provider failover via `services/llm-router/`.
- [ ] SiteModel composer producing `schemas/site-model/v1.json`-conforming output.
- [ ] Persist SiteModel to S3 and DB.

### User Stories (EP-ANALYSIS)

- [ ] **US-ANA-001** Extract content (5 pts).
- [ ] **US-ANA-002** Classify page roles (≥ 90% accuracy) (5 pts).
- [ ] **US-ANA-003** Build navigation graph (5 pts).
- [ ] **US-ANA-004** Run SEO audit (5 pts).
- [ ] **US-ANA-005** Run a11y audit (5 pts).
- [ ] **US-ANA-006** Detect tech stack (3 pts).
- [ ] **US-ANA-007** *(M4)* Extract NAP (3 pts).
- [ ] **US-ANA-008** LLM summarization with grounding and cost bounds (5 pts).

### M2 Exit Criteria

- [ ] SiteModel quality reviewed by humans for a sample of 30 sites; ≥ 90% judged usable.
- [ ] LLM cost per analysis within budget.
- [ ] LLM provider failover works in tests.

---

## 4. M3 — Generation (Single-Page) (`docs/12-generation-engine.md`, EP-GEN)

- [ ] Implement `apps/generation-worker/`.
- [ ] Scaffold Next.js 14 (App Router, TypeScript strict, Tailwind 3.4) per ADR-002.
- [ ] Theme system with 5 starter themes using design tokens (no hard-coded colors).
- [ ] Route generator (single-page output for M3).
- [ ] Component generator (Hero, Features, About, Contact, Footer).
- [ ] Content data layer (JSON content files) consumed by pages.
- [ ] Asset pipeline (carry over images/PDFs, rewrite to local paths).
- [ ] `next build` self-test runner with up to 3 self-repair attempts.
- [ ] S3 workspace storage per job.
- [ ] Quarantine path on persistent build failure.

### User Stories (EP-GEN, M3 scope)

- [ ] **US-GEN-001** Generate a single-page site (8 pts).
- [ ] **US-GEN-003** Apply a theme — 5 starter themes (5 pts).
- [ ] **US-GEN-004** Self-repair on build failure (3 attempts) (8 pts).
- [ ] **US-GEN-005** Carry over assets (5 pts).
- [ ] **US-GEN-007** Content data layer (5 pts).

### M3 Exit Criteria

- [ ] Generated single-page app builds successfully for 100% of fixture sites.
- [ ] Lighthouse Performance ≥ 90 on the median fixture.
- [ ] Self-repair rate ≤ 20% on the median fixture.

---

## 5. M4 — SEO + Quality (`docs/13-seo-engine.md`, EP-SEO, EP-QR)

- [ ] Implement `apps/seo-worker/`.
- [ ] Metadata generation (title, description, canonical, OG, Twitter) per page.
- [ ] JSON-LD: Organization, LocalBusiness (when NAP present).
- [ ] `sitemap.xml`, `robots.txt` (AI scrapers allowed by default; opt-out).
- [ ] `public/llms.txt` and optional `llms-full.txt`.
- [ ] OG image generation per page.
- [ ] Alt text synthesis with confidence; reviewable before deploy.
- [ ] Quality reviewer: Lighthouse + axe gates; visual regression; content fidelity diff.
- [ ] Security headers (HSTS, X-Content-Type-Options, X-Frame-Options, Referrer-Policy, modest CSP).

### User Stories (M4 scope)

- [ ] **US-SEO-001** Metadata generation (3 pts).
- [ ] **US-SEO-002** JSON-LD (3 pts).
- [ ] **US-SEO-003** Sitemap (2 pts).
- [ ] **US-SEO-004** robots.txt (2 pts).
- [ ] **US-SEO-005** llms.txt + llms-full.txt (2 pts).
- [ ] **US-SEO-006** Alt text synthesis (5 pts).
- [ ] **US-SEO-007** OG image (3 pts).
- [ ] **US-ANA-007** NAP extraction (3 pts) — feeds SEO.
- [ ] **US-QR-001** Lighthouse gate (5 pts).
- [ ] **US-QR-002** A11y gate (3 pts).
- [ ] **US-QR-003** Visual regression (5 pts).
- [ ] **US-QR-004** Content fidelity (3 pts).
- [ ] **US-QR-005** Security headers (3 pts).
- [ ] **US-ADM-006** Override a quality gate (2 pts).

### M4 Exit Criteria

- [ ] Lighthouse Perf ≥ 90, SEO = 100, BP = 100, A11y ≥ 95 on the median fixture.
- [ ] `llms.txt` validates against the published schema.
- [ ] JSON-LD validates against schema.org.

---

## 6. M5 — Deployment (`docs/14-deployment-engine.md`, `docs/15-github-integration.md`, `docs/16-vercel-integration.md`, EP-DEPLOY)

- [ ] Implement `apps/deploy-worker/`.
- [ ] GitHub App scaffolding (OAuth install, per-installation tokens, disconnect).
- [ ] Vercel integration (API token handling, project provisioning, deploy trigger, build log capture).
- [ ] Repo creation (org or user, default branch `main`, signed commits, provenance commit message).
- [ ] Code push of generated workspace.
- [ ] Custom domain support with DNS verification instructions.
- [ ] Rollback: list past deploys, atomic promote-to-production.
- [ ] Re-deploy from dashboard (idempotent).
- [ ] Failure compensation registry executed on partial failure.

### User Stories (M5)

- [ ] **US-DEP-001** Connect GitHub (5 pts).
- [ ] **US-DEP-002** Create a GitHub repo (3 pts).
- [ ] **US-DEP-003** Push the generated workspace (3 pts).
- [ ] **US-DEP-004** Create a Vercel project (3 pts).
- [ ] **US-DEP-005** Trigger a deploy (3 pts).
- [ ] **US-DEP-006** Custom domain (5 pts).
- [ ] **US-DEP-007** Rollback (3 pts).
- [ ] **US-DEP-008** Re-deploy from dashboard (2 pts).
- [ ] **US-DEP-009** Failure compensation (5 pts).
- [ ] **US-GEN-006** Generate 301 redirects (3 pts).
- [ ] **US-GEN-008** Generate contact form with spam protection + webhook forward (3 pts).
- [ ] **US-QR-006** Final approval gate (3 pts).

### M5 Exit Criteria

- [ ] 95% of jobs deploy to a Vercel preview URL successfully.
- [ ] Compensation registry exercises documented in tests.
- [ ] No cross-tenant access in security tests.

---

## 7. M6 — Reporting + Delivery (`docs/24-backlog.md` EP-DELIVERY)

- [ ] Implement `apps/delivery-worker/`.
- [ ] Per-job report (PDF + JSON) including capture, analysis, generation, deploy outcomes, Lighthouse/axe/SEO scores.
- [ ] Per-tenant summary with date range and status filters, CSV export.
- [ ] Webhook subscriptions (HMAC signing, event filters, disable).
- [ ] Webhook retries with exponential backoff; manual retry; dashboard listing.
- [ ] Email notifications (configurable preferences, per-job opt-out).
- [ ] In-app notifications panel.
- [ ] Customer dashboard timeline (real-time updates, failure explanations).

### User Stories (M6)

- [ ] **US-DEL-001** Per-job report (5 pts).
- [ ] **US-DEL-002** Per-tenant summary (3 pts).
- [ ] **US-DEL-003** Webhook subscriptions (5 pts).
- [ ] **US-DEL-004** Webhook retries (3 pts).
- [ ] **US-DEL-005** Email notifications (3 pts).
- [ ] **US-DEL-006** In-app notifications (2 pts).
- [ ] **US-DASH-002** Per-job timeline (3 pts).
- [ ] **US-DASH-003** Reports download (3 pts).
- [ ] **US-OBS-004** Customer-facing timeline (3 pts).

### M6 Exit Criteria

- [ ] Reports include: original URL, target URL, manifest summary, SiteModel diff highlights, Lighthouse scores, sitemap reference.
- [ ] Webhook delivery success ≥ 99% over 7 days in staging.
- [ ] Customers receive notification within 1 minute of completion.

---

## 8. M7 — Self-Serve Billing (`docs/24-backlog.md` EP-BILLING)

- [ ] Implement `apps/billing-worker/`.
- [ ] Stripe Checkout + Customer Portal integration.
- [ ] Subscription state machine.
- [ ] Plan-based quotas enforced (jobs/month, LLM spend cap, capture pages).
- [ ] Usage metering; overage handling.
- [ ] Invoice generation; VAT/Tax IDs; configurable billing email.
- [ ] Coupon support (per-customer or per-promotion, logged).
- [ ] Idempotent webhook ingestion (replay-safe).
- [ ] Cost transparency dashboard (jobs, LLM spend, capture pages, deploys) with month-end forecast.

### User Stories (M7)

- [ ] **US-BIL-001** Plan selection (3 pts).
- [ ] **US-BIL-002** Plan upgrade/downgrade (3 pts).
- [ ] **US-BIL-003** Quota enforcement (5 pts).
- [ ] **US-BIL-004** Invoices (3 pts).
- [ ] **US-BIL-005** Cancellation (3 pts).
- [ ] **US-BIL-006** Coupon support (3 pts).
- [ ] **US-BIL-007** Cost transparency (3 pts).
- [ ] **US-DASH-005** Usage dashboard (3 pts).
- [ ] **US-INTAKE-006** Trial period (5 pts).
- [ ] **US-ADM-004** Manual refund (3 pts).
- [ ] **US-SEC-005** DPA acceptance (2 pts).

### M7 Exit Criteria

- [ ] Customer can sign up, pick a plan, run a job, and be billed correctly.
- [ ] Plan upgrades and downgrades work without operator intervention.
- [ ] Webhook ingestion is idempotent and survives replay.

---

## 9. M8 — Public API (`docs/09-api-specification.md`, EP-API)

- [ ] Publish versioned OpenAPI.
- [ ] API key management (scoped, optional expiry, plaintext shown once, immediate revocation).
- [ ] `POST /v1/jobs` with Idempotency-Key.
- [ ] `GET /v1/jobs/{id}` (with `?include=events`, ETag/If-None-Match).
- [ ] `GET /v1/jobs` cursor pagination; filters by status, date.
- [ ] `POST /v1/jobs/{id}/cancel` with compensation.
- [ ] `GET /v1/jobs/{id}/report` signed time-bound URL (pdf|json).
- [ ] `POST /v1/webhooks` (list, delete, on-demand test event).
- [ ] `X-RateLimit-*` headers; 429 + `Retry-After`.
- [ ] Public TypeScript SDK (`@vibe/sdk`) published to npm; CI smoke tests.

### User Stories (M8)

- [ ] **US-API-001** Create a job (3 pts).
- [ ] **US-API-002** Get job status (2 pts).
- [ ] **US-API-003** List jobs (3 pts).
- [ ] **US-API-004** Cancel a job (3 pts).
- [ ] **US-API-005** Download report (2 pts).
- [ ] **US-API-006** Webhook subscription (5 pts).
- [ ] **US-API-007** TypeScript SDK (5 pts).
- [ ] **US-API-008** OpenAPI documentation (3 pts).
- [ ] **US-API-009** Rate limits visible (2 pts).
- [ ] **US-AUTH-003** API key issuance (3 pts).

### M8 Exit Criteria

- [ ] API passes the published SLA in staging.
- [ ] SDK smoke tests succeed for all endpoints.
- [ ] Documentation is reviewed by an external developer.

---

## 10. M9 — Multi-Page Sites (`docs/24-backlog.md` EP-CAPTURE/EP-GEN, M9)

- [ ] Multi-page generation with shared layout and global navigation.
- [ ] Component library expansion.
- [ ] Per-job custom theme authoring for paid tiers.
- [ ] Content data layer expansion (MDX/JSON).
- [ ] Per-section regeneration without full rebuild.
- [ ] Blog index and posts; RSS feed.
- [ ] Manual URL additions to a capture.
- [ ] Crawl a site within a domain (same-origin, depth/page cap, `robots.txt` honoring).

### User Stories (M9)

- [ ] **US-CAP-002** Crawl a site within a domain (8 pts).
- [ ] **US-CAP-007** Manual URL additions (3 pts).
- [ ] **US-GEN-002** Multi-page site generation (13 pts).
- [ ] **US-GEN-009** Blog index and posts + RSS (5 pts).
- [ ] **US-GEN-010** Per-section regeneration (8 pts).

### M9 Exit Criteria

- [ ] 95% of multi-page fixtures build successfully.
- [ ] Customer can request a "deep scan" that produces multi-page output.

---

## 11. Cross-Cutting — Auth, Dashboard, Admin, Support, Security, Maintenance

Track now; deliver as part of the milestones that own them. Listed here for visibility.

- [ ] **US-AUTH-001** Magic link login (3 pts, M1).
- [ ] **US-AUTH-002** Role assignment (3 pts, M3).
- [ ] **US-AUTH-004** SSO (SAML/OIDC) (8 pts, M10).
- [ ] **US-AUTH-005** Audit log (3 pts, M3).
- [ ] **US-AUTH-006** Session control (2 pts, M1).
- [ ] **US-AUTH-007** Tenant deletion (3 pts, M3).
- [ ] **US-DASH-001** Job history (3 pts, M3).
- [ ] **US-DASH-004** Settings (3 pts, M3).
- [ ] **US-DASH-006** Brand customization (3 pts, M3).
- [ ] **US-ADM-001** Tenant list (2 pts, M3).
- [ ] **US-ADM-002** Tenant detail (3 pts, M3).
- [ ] **US-ADM-003** Suspend a tenant (2 pts, M3).
- [ ] **US-ADM-005** Replay a job (3 pts, M3).
- [ ] **US-ADM-007** Feature flags (3 pts, M3).
- [ ] **US-SUP-001** Internal support console (5 pts, M3).
- [ ] **US-SUP-002** Impersonation (3 pts, M3).
- [ ] **US-SUP-003** Run a job on behalf (3 pts, M3).
- [ ] **US-SUP-004** Status page authoring (3 pts, M3).
- [ ] **US-SUP-005** Knowledge base (3 pts, M3).
- [ ] **US-SEC-001** SSO enforcement (5 pts, M10).
- [ ] **US-SEC-002** Audit log export (3 pts, M3).
- [ ] **US-SEC-003** GDPR data export (3 pts, M3).
- [ ] **US-SEC-004** GDPR deletion (3 pts, M3).
- [ ] **US-SEC-006** Threat model review (3 pts, M0).
- [ ] **US-MAINT-001** Backups (3 pts, M0).
- [ ] **US-MAINT-002** Retention policies (3 pts, M3).
- [ ] **US-MAINT-003** Dependency updates (2 pts, M0).
- [ ] **US-MAINT-004** Database migrations (3 pts, M0).
- [ ] **US-MAINT-005** Rollback drill (3 pts, M3).
- [ ] **US-MAINT-006** Capacity planning (3 pts, M3).
- [ ] **US-OBS-001** Service dashboards (5 pts, M0).
- [ ] **US-OBS-002** SLO dashboards (3 pts, M0).
- [ ] **US-OBS-003** Per-job trace (3 pts, M0).
- [ ] **US-OBS-005** On-call rotation (2 pts, M0).

---

## 12. V2 — M10 Whitelabel + SSO

- [ ] **US-WL-001** Vibe Connect GitHub App (5 pts).
- [ ] **US-WL-002** Branded output (5 pts).
- [ ] **US-WL-003** Custom Vercel team (5 pts).
- [ ] **US-WL-004** White-label email (5 pts).
- [ ] **US-SEC-001** SSO enforcement (5 pts).
- [ ] **US-AUTH-004** SSO (SAML/OIDC) (8 pts).
- [ ] SOC 2 Type I evidence collection; audit readiness.

---

## 13. V2 — M11 Migration Path

- [ ] 301 redirect map generation from old → new URLs.
- [ ] Asset carry-over (images, PDFs).
- [ ] DNS cutover playbook.
- [ ] Source decommissioning flow.

---

## 14. V2 — M12 Continuous Improvement

- [ ] Nightly agent eval using offline traces.
- [ ] Prompt template versioning with promotion gates.
- [ ] Anomaly detection on per-tenant cost spikes.
- [ ] Auto-generated retrospective reports for ops.

---

## 15. V3 — M13 Autonomous Fleet

- [ ] Continuous improvement loops across the fleet.
- [ ] Per-customer SLAs; dynamic agent tuning.
- [ ] Fleet-wide rollouts with canary.
- [ ] Long-running "concierge" jobs that improve a site over time.

---

## 16. Open Questions / Blockers

- [ ] Confirm the project's LICENSE before any public commit.
- [ ] Confirm the public package namespace (`@vibe/...` is implied; verify with owner).
- [ ] Confirm Temporal self-host vs. Temporal Cloud for staging (ADR-006 pending).
- [ ] Confirm cloud target (AWS is assumed per `docs/00-project-overview.md`; verify).
- [ ] Confirm RLS approach (Postgres RLS vs. application-level tenancy filter).
- [ ] Confirm whether `pglite` or real Postgres is acceptable for the earliest dev cycle.
- [ ] Verify Node 20 LTS vs. 24 LTS (current env has 24.3.0); pin via `.tool-versions`.
- [ ] Verify Python 3.12 (env has 3.14); pin via `.tool-versions` and use `uv` to manage.

---

## 17. How To Use This Ledger

1. **Pick a task from §1 (M0)** — never start M1+ until M0 exit criteria are met.
2. **Mark `[~]` when starting**, `[x]` when the relevant test/check passes, `[!]` if blocked.
3. **Reference a doc and (when applicable) a US-XXX ID** in commit messages.
4. **Update this file in the same PR** as the work it tracks.
5. **Re-baseline `Status Snapshot`** at the top whenever a milestone's exit criteria flip green.

---

_Last updated: project kickoff._
