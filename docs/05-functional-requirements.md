# 05 — Functional Requirements

> The exhaustive list of capabilities the platform must provide, expressed in a verifiable form.

---

## Purpose

This document enumerates the functional requirements of the platform. A functional requirement specifies what the system shall do. Each requirement is identified, testable, traceable, and prioritized.

Functional requirements derived from this document feed:

- Engineering backlog (`24-backlog.md`)
- Test plan (`19-testing-strategy.md`)
- Acceptance gates per milestone (`23-milestone-roadmap.md`)

---

## Scope

In scope:

- All functional requirements for MVP, V2, and V3.
- Requirements grouped by subsystem.
- Priority and milestone tagging.

Out of scope:

- Performance, security, and other non-functional requirements (`06-nonfunctional-requirements.md`).
- Implementation details (`07-technical-specification.md`).

---

## Requirement Format

Each requirement is expressed as:

> **FR-<area>-<n>** — <imperative statement>
>
> Priority: `MUST` | `SHOULD` | `MAY`
> Milestone: `M1`–`M6`
> Source: `PRD §...` or `US-...`
> Verification: `unit` | `integration` | `e2e` | `manual` | `observability`

`MUST` requirements are blocking for the indicated milestone. `SHOULD` requirements are blocking for the milestone unless explicitly deferred. `MAY` requirements are aspirational within the milestone.

---

## Areas

| Code | Area |
|------|------|
| `INT` | Intake & validation |
| `CAP` | Capture |
| `ANA` | Analysis |
| `GEN` | Generation |
| `SEO` | SEO |
| `QR` | Quality review |
| `DEP` | Deployment |
| `RPT` | Reporting & delivery |
| `BIL` | Billing & subscriptions |
| `AUTH` | Authentication & authorization |
| `DASH` | Dashboard |
| `ADM` | Admin / operator console |
| `API` | Public API |
| `INF` | Infrastructure & ops |
| `OBS` | Observability |
| `MNT` | Maintenance (continuous improvement) |
| `WL` | White-label |

---

## FR-INT — Intake & Validation

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-INT-001 | The system shall accept a URL via a public form. | MUST | M1 | US-INTAKE-001 | e2e |
| FR-INT-002 | The system shall validate that the URL is syntactically well-formed (RFC 3986). | MUST | M1 | US-INTAKE-001 | unit |
| FR-INT-003 | The system shall reject URLs whose hostname resolves to a private IP. | MUST | M1 | Security (§17) | integration |
| FR-INT-004 | The system shall enforce a configurable domain denylist at intake. | MUST | M1 | US-INTAKE-004 | integration |
| FR-INT-005 | The system shall attempt canonical URL resolution (`http`, `https`, `www.`). | SHOULD | M1 | US-INTAKE-003 | integration |
| FR-INT-006 | The system shall provide a free preview within 30 seconds of submission. | SHOULD | M1 | US-INTAKE-001 | e2e |
| FR-INT-007 | The system shall accept bulk CSV intake of up to 500 rows. | MUST | M5 | US-INTAKE-006 | integration |
| FR-INT-008 | The system shall persist every intake attempt (accepted or rejected) for audit. | MUST | M1 | Security (§17) | observability |
| FR-INT-009 | The system shall produce a customer-facing error message distinguishing DNS, network, and content failures. | SHOULD | M1 | US-INTAKE-005 | e2e |

---

## FR-CAP — Capture

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-CAP-001 | The system shall fetch the root URL using a headless browser. | MUST | M1 | US-CAPTURE-001 | integration |
| FR-CAP-002 | The system shall record one HAR file per captured page. | MUST | M1 | US-CAPTURE-002 | integration |
| FR-CAP-003 | The system shall capture a full-page screenshot for desktop (1440×900) and mobile (390×844) viewports per page. | MUST | M1 | US-CAPTURE-003 | integration |
| FR-CAP-004 | The system shall crawl internal links to a configurable depth (default 2). | MUST | M1 | US-CAPTURE-001 | integration |
| FR-CAP-005 | The system shall respect `robots.txt`. | MUST | M1 | US-CAPTURE-005 | integration |
| FR-CAP-006 | The system shall identify a custom `User-Agent` string. | MUST | M1 | Security (§17) | integration |
| FR-CAP-007 | The system shall enforce a maximum-pages cap per tier. | MUST | M1 | Cost (§28) | integration |
| FR-CAP-008 | The system shall enforce a maximum-bytes-per-page cap. | MUST | M1 | Cost, Security | integration |
| FR-CAP-009 | The system shall wait for network idle or up to 10 seconds before snapshot. | SHOULD | M1 | US-CAPTURE-004 | integration |
| FR-CAP-010 | The system shall detect common bot-wall signatures and abort with a specific status. | SHOULD | M1 | US-CAPTURE-006 | integration |
| FR-CAP-011 | The system shall capture multi-language sites via `hreflang` discovery. | SHOULD | M5 | US-CAPTURE-007 | integration |
| FR-CAP-012 | The system shall store all captured artifacts in object storage under the tenant prefix. | MUST | M1 | Security (§17) | integration |
| FR-CAP-013 | The system shall record per-page capture latency. | MUST | M1 | OBS | observability |
| FR-CAP-014 | The system shall support authenticated capture given user-provided credentials. | MAY | M6 | US-CAPTURE-008 | integration |
| FR-CAP-015 | The system shall produce a capture manifest conforming to the `CaptureManifest` v1 schema. | MUST | M1 | §03 | unit |

---

## FR-ANA — Analysis

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-ANA-001 | The system shall produce a `SiteModel` conforming to v1 schema from a capture manifest. | MUST | M2 | §03 | unit |
| FR-ANA-002 | The system shall build a page inventory with a role classification per page. | MUST | M2 | US-ANALYSIS-001 | integration |
| FR-ANA-003 | The system shall build a navigation graph distinguishing primary, footer, and in-content links. | MUST | M2 | US-ANALYSIS-002 | unit |
| FR-ANA-004 | The system shall extract structured content preserving heading hierarchy. | MUST | M2 | US-ANALYSIS-003 | unit |
| FR-ANA-005 | The system shall produce an SEO audit per page with severity-scored findings. | MUST | M2 | US-ANALYSIS-004 | integration |
| FR-ANA-006 | The system shall produce an accessibility audit per page using axe-core rules. | MUST | M2 | US-ANALYSIS-005 | integration |
| FR-ANA-007 | The system shall classify the site by industry with a confidence score. | SHOULD | M2 | US-ANALYSIS-006 | integration |
| FR-ANA-008 | The system shall extract contact methods (phone, email, address, hours) where present. | SHOULD | M2 | US-ANALYSIS-007 | unit |
| FR-ANA-009 | The system shall detect the site's primary technology stack. | SHOULD | M2 | §11 | unit |
| FR-ANA-010 | The system shall fall back to a generic industry template when classification confidence is below 0.6. | MUST | M2 | US-ANALYSIS-006 | integration |
| FR-ANA-011 | The system shall record token usage and LLM cost per analysis. | MUST | M2 | §28 | observability |
| FR-ANA-012 | The system shall produce a content map cross-referencing pages and reusable blocks. | SHOULD | M2 | §11 | unit |

---

## FR-GEN — Generation

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-GEN-001 | The system shall produce a Next.js application that builds successfully with `pnpm build`. | MUST | M3 | US-GEN-001 | integration |
| FR-GEN-002 | The system shall produce TypeScript code with no type errors. | MUST | M3 | US-GEN-001 | integration |
| FR-GEN-003 | The system shall produce code that passes the project's lint configuration. | MUST | M3 | §21 | integration |
| FR-GEN-004 | The system shall generate one Next.js route per source page by default. | MUST | M3 | US-GEN-002 | integration |
| FR-GEN-005 | The system shall use semantic HTML5 landmarks for the generated layout. | MUST | M3 | US-GEN-003 | unit |
| FR-GEN-006 | The system shall use Tailwind CSS for styling. | MUST | M3 | §07 | unit |
| FR-GEN-007 | The system shall derive a color palette from the source site or apply a sensible default. | SHOULD | M3 | US-GEN-004 | unit |
| FR-GEN-008 | The system shall use the Next.js `<Image>` component for all `<img>` references. | MUST | M3 | US-GEN-006 | unit |
| FR-GEN-009 | The system shall serve images in WebP or AVIF where supported. | SHOULD | M3 | US-GEN-006 | integration |
| FR-GEN-010 | The system shall preserve forms with submission to a configurable endpoint. | SHOULD | M3 | US-GEN-007 | integration |
| FR-GEN-011 | The system shall self-repair build errors up to 3 times per build before failing. | MUST | M3 | US-GEN-005 | integration |
| FR-GEN-012 | The system shall produce a generated workspace archived to object storage. | MUST | M3 | §03 | integration |
| FR-GEN-013 | The system shall include a generated `README.md` describing how to run the site locally. | MUST | M3 | §03 | unit |
| FR-GEN-014 | The system shall include a generated `LICENSE` (MIT unless overridden). | MUST | M3 | §03 | unit |
| FR-GEN-015 | The system shall pin all dependencies in `package.json` with exact versions. | MUST | M3 | §21 | unit |
| FR-GEN-016 | The system shall commit a `pnpm-lock.yaml` lockfile. | MUST | M3 | §21 | unit |
| FR-GEN-017 | The system shall include a default `vercel.json` configured for Next.js. | SHOULD | M3 | §16 | unit |

---

## FR-SEO — SEO

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-SEO-001 | The system shall set a unique `<title>` on every page. | MUST | M4 | US-SEO-001 | integration |
| FR-SEO-002 | The system shall set a unique meta description on every page. | MUST | M4 | US-SEO-001 | integration |
| FR-SEO-003 | The system shall emit `application/ld+json` Schema.org structured data appropriate to the page type. | MUST | M4 | US-SEO-002 | integration |
| FR-SEO-004 | The system shall generate a `sitemap.xml` at the site root. | MUST | M4 | US-SEO-003 | integration |
| FR-SEO-005 | The system shall split sitemaps when URL count exceeds 50,000. | SHOULD | M4 | US-SEO-003 | integration |
| FR-SEO-006 | The system shall generate a `robots.txt` referencing the sitemap. | MUST | M4 | US-SEO-006 | integration |
| FR-SEO-007 | The system shall emit Open Graph metadata on every page. | MUST | M4 | US-SEO-005 | integration |
| FR-SEO-008 | The system shall emit Twitter card metadata on every page. | SHOULD | M4 | US-SEO-005 | integration |
| FR-SEO-009 | The system shall emit `llms.txt` per the current convention. | MUST | M4 | US-SEO-004 | integration |
| FR-SEO-010 | The system shall validate all JSON-LD against the Schema.org spec. | MUST | M4 | §13 | unit |
| FR-SEO-011 | The system shall include canonical link tags on every page. | MUST | M4 | §13 | unit |
| FR-SEO-012 | The system shall generate alt text for every image. | MUST | M4 | A11y | integration |

---

## FR-QR — Quality Review

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-QR-001 | The system shall run Lighthouse on every generated site before deployment. | MUST | M3 | §03 | integration |
| FR-QR-002 | The system shall run axe-core on every generated site before deployment. | MUST | M3 | §03 | integration |
| FR-QR-003 | The system shall fail the gate when median Lighthouse Performance < 85. | MUST | M3 | §01 | integration |
| FR-QR-004 | The system shall fail the gate when Accessibility < 90. | MUST | M3 | §01 | integration |
| FR-QR-005 | The system shall fail the gate when SEO < 95. | MUST | M3 | §01 | integration |
| FR-QR-006 | The system shall verify no broken internal links. | MUST | M3 | §03 | integration |
| FR-QR-007 | The system shall validate all metadata (JSON-LD, OG, Twitter) parses cleanly. | MUST | M3 | §13 | unit |
| FR-QR-008 | The system shall route failing gates back to the Generation Agent for self-repair, up to 2 iterations. | MUST | M3 | §03 | integration |
| FR-QR-009 | Operators may override a failing gate with a logged reason. | SHOULD | M3 | US-ADMIN-004 | integration |

---

## FR-DEP — Deployment

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-DEP-001 | The system shall create a GitHub repository for every successful job. | MUST | M5 | US-DEPLOY-001 | integration |
| FR-DEP-002 | The system shall commit the generated workspace as the initial commit. | MUST | M5 | US-DEPLOY-002 | integration |
| FR-DEP-003 | The system shall create a Vercel project linked to the repository. | MUST | M5 | US-DEPLOY-003 | integration |
| FR-DEP-004 | The system shall trigger a production deployment after repo push. | MUST | M5 | US-DEPLOY-004 | integration |
| FR-DEP-005 | The system shall capture build logs from Vercel. | MUST | M5 | §16 | integration |
| FR-DEP-006 | The system shall verify the deployment is reachable before reporting success. | MUST | M5 | §16 | integration |
| FR-DEP-007 | The system shall provide a default repo visibility of `private`. | MUST | M5 | §17 | unit |
| FR-DEP-008 | The system shall support pushing into a customer-installed GitHub App org. | SHOULD | M5 | US-DEPLOY-006 | integration |
| FR-DEP-009 | The system shall support deploying to a customer-linked Vercel team. | SHOULD | M5 | US-WL-004 | integration |
| FR-DEP-010 | The system shall support attaching a custom domain. | SHOULD | M5 | US-DEPLOY-005 | integration |
| FR-DEP-011 | The system shall compensate (delete repo, delete Vercel project) on irrecoverable failure. | MUST | M5 | §02 | integration |
| FR-DEP-012 | The system shall use idempotency keys for all GitHub and Vercel side effects. | MUST | M5 | §03 | integration |

---

## FR-RPT — Reporting & Delivery

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-RPT-001 | The system shall produce a Markdown report per job. | MUST | M5 | US-REPORT-002 | unit |
| FR-RPT-002 | The system shall produce a PDF version of the report. | SHOULD | M5 | US-REPORT-002 | integration |
| FR-RPT-003 | The system shall send a delivery email to the customer within 5 minutes of job completion. | MUST | M5 | US-REPORT-001 | integration |
| FR-RPT-004 | The system shall include pre and post Lighthouse scores in the report. | MUST | M5 | US-REPORT-003 | integration |
| FR-RPT-005 | The system shall include a content diff summary in the report. | SHOULD | M5 | US-REPORT-004 | unit |
| FR-RPT-006 | The system shall dispatch a webhook on job completion if a callback URL was provided. | MUST | M6 | US-API-002 | integration |
| FR-RPT-007 | The system shall sign webhooks with HMAC-SHA256. | MUST | M6 | US-API-002 | unit |

---

## FR-BIL — Billing

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-BIL-001 | The system shall integrate Stripe Checkout for payment. | MUST | M5 | US-BILLING-001 | integration |
| FR-BIL-002 | The system shall create a Stripe customer per tenant. | MUST | M5 | §08 | integration |
| FR-BIL-003 | The system shall handle Stripe webhooks for `checkout.session.completed`, `invoice.paid`, `customer.subscription.updated`, `customer.subscription.deleted`. | MUST | M5 | §08 | integration |
| FR-BIL-004 | The system shall enforce per-tenant job quota synchronously at intake. | MUST | M5 | US-BILLING-004 | integration |
| FR-BIL-005 | The system shall issue a full refund when a job fails terminally. | MUST | M5 | US-BILLING-005 | integration |
| FR-BIL-006 | The system shall support subscriptions with monthly cadence. | SHOULD | M5 | US-BILLING-002 | integration |
| FR-BIL-007 | The system shall calculate prorations on tier upgrades. | SHOULD | M5 | US-BILLING-006 | integration |
| FR-BIL-008 | The system shall produce tax-compliant invoices via Stripe Tax. | SHOULD | M5 | US-BILLING-003 | integration |

---

## FR-AUTH — Authentication & Authorization

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-AUTH-001 | The system shall authenticate customers via passwordless email magic link. | MUST | M5 | §17 | integration |
| FR-AUTH-002 | The system shall issue short-lived (15 min) access JWTs and long-lived refresh tokens. | MUST | M5 | §17 | unit |
| FR-AUTH-003 | The system shall authenticate API callers via tenant-scoped API keys. | MUST | M6 | §17 | integration |
| FR-AUTH-004 | The system shall enforce tenant isolation on every resource read. | MUST | M5 | §17 | integration |
| FR-AUTH-005 | The system shall support roles `admin`, `operator`, `viewer` within a tenant. | SHOULD | M5 | US-DASH-005 | integration |
| FR-AUTH-006 | The system shall require MFA for `admin` operator console access. | MUST | M5 | §17 | manual |
| FR-AUTH-007 | The system shall log every authentication event. | MUST | M5 | §17 | observability |

---

## FR-DASH — Dashboard

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-DASH-001 | The dashboard shall list all jobs belonging to the authenticated tenant. | MUST | M5 | US-DASH-001 | e2e |
| FR-DASH-002 | The dashboard shall display real-time job status. | SHOULD | M5 | US-DASH-002 | e2e |
| FR-DASH-003 | The dashboard shall allow re-running a job. | SHOULD | M5 | US-DASH-003 | e2e |
| FR-DASH-004 | The dashboard shall expose Stripe customer portal for billing. | SHOULD | M5 | US-DASH-004 | integration |
| FR-DASH-005 | The dashboard shall expose the report bundle as a downloadable archive. | MUST | M5 | US-REPORT-002 | e2e |
| FR-DASH-006 | The dashboard shall expose team-member invitations. | SHOULD | M5 | US-DASH-005 | integration |
| FR-DASH-007 | The dashboard shall expose API key management (V3). | MUST | M6 | US-API-001 | e2e |

---

## FR-ADM — Admin

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-ADM-001 | The admin console shall list every job across all tenants. | MUST | M5 | US-ADMIN-001 | integration |
| FR-ADM-002 | The admin console shall allow per-phase replay of a job. | MUST | M5 | US-ADMIN-002 | integration |
| FR-ADM-003 | The admin console shall allow tenant suspension and unsuspension. | MUST | M5 | US-ADMIN-003 | integration |
| FR-ADM-004 | The admin console shall allow quality-gate override with a logged reason. | SHOULD | M5 | US-ADMIN-004 | integration |
| FR-ADM-005 | The admin console shall allow manual refunds. | SHOULD | M5 | US-ADMIN-005 | integration |
| FR-ADM-006 | The admin console shall display agent traces per job. | MUST | M5 | §18 | e2e |

---

## FR-API — Public API

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-API-001 | The system shall expose `POST /v1/jobs` for job creation. | MUST | M6 | US-API-001 | integration |
| FR-API-002 | The system shall expose `GET /v1/jobs` for listing jobs. | MUST | M6 | US-API-003 | integration |
| FR-API-003 | The system shall expose `GET /v1/jobs/{id}` for job retrieval. | MUST | M6 | §09 | integration |
| FR-API-004 | The system shall expose `POST /v1/jobs/{id}/cancel`. | MUST | M6 | US-API-004 | integration |
| FR-API-005 | The system shall expose `GET /v1/reports/{id}`. | MUST | M6 | §09 | integration |
| FR-API-006 | The system shall version all routes under `/v1/`. | MUST | M6 | US-API-005 | integration |
| FR-API-007 | The system shall return JSON responses with `application/json; charset=utf-8`. | MUST | M6 | §09 | unit |
| FR-API-008 | The system shall return RFC 7807 `application/problem+json` for errors. | MUST | M6 | §09 | unit |
| FR-API-009 | The system shall rate-limit API calls per tenant and per route. | MUST | M6 | §17 | integration |
| FR-API-010 | The system shall publish an OpenAPI 3.1 specification. | MUST | M6 | §09 | unit |

---

## FR-INF — Infrastructure

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-INF-001 | All services shall be deployable via Infrastructure-as-Code (Terraform). | MUST | M1 | §27 | integration |
| FR-INF-002 | All services shall run in containers. | MUST | M1 | §02 | integration |
| FR-INF-003 | The system shall support a single-region production deployment. | MUST | M1 | §02 | manual |
| FR-INF-004 | The system shall support multi-region capture (V3). | MAY | M6 | §02 | manual |
| FR-INF-005 | The system shall use ephemeral compute for workers. | MUST | M1 | §02 | manual |
| FR-INF-006 | The database shall be Multi-AZ. | MUST | M1 | §02 | manual |
| FR-INF-007 | Object storage shall be versioned. | MUST | M1 | §02 | manual |
| FR-INF-008 | Secrets shall be stored in a managed secret store. | MUST | M1 | §17 | manual |

---

## FR-OBS — Observability

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-OBS-001 | All services shall emit OpenTelemetry traces. | MUST | M1 | §18 | integration |
| FR-OBS-002 | All services shall emit structured JSON logs. | MUST | M1 | §18 | integration |
| FR-OBS-003 | All services shall emit metrics for golden signals (latency, traffic, errors, saturation). | MUST | M1 | §18 | integration |
| FR-OBS-004 | Each agent run shall produce an audit row including cost. | MUST | M1 | §03 | integration |
| FR-OBS-005 | The system shall maintain a status page reflecting real-time health. | SHOULD | M5 | §18 | manual |
| FR-OBS-006 | The system shall alert on SLO burn rates. | MUST | M5 | §18 | manual |

---

## FR-MNT — Maintenance

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-MNT-001 | The system shall schedule monthly recaptures for active subscribers. | MUST | M6 | US-MAINTAIN-001 | integration |
| FR-MNT-002 | The system shall open a PR per detected improvement. | SHOULD | M6 | US-MAINTAIN-002 | integration |
| FR-MNT-003 | The system shall support auto-merge for safe categories when opted in. | MAY | M6 | US-MAINTAIN-003 | integration |
| FR-MNT-004 | The system shall support subscription pause/resume. | SHOULD | M6 | US-MAINTAIN-004 | integration |

---

## FR-WL — White-Label

| ID | Requirement | Priority | Milestone | Source | Verification |
|----|-------------|----------|-----------|--------|--------------|
| FR-WL-001 | The system shall apply tenant-specific branding to reports. | SHOULD | M5 | US-WL-001 | integration |
| FR-WL-002 | The system shall omit Vibe attribution from generated code on Pro tier. | SHOULD | M5 | US-WL-002 | integration |
| FR-WL-003 | The system shall route deployments to a tenant-linked GitHub App installation. | MUST | M5 | US-WL-003 | integration |
| FR-WL-004 | The system shall route deployments to a tenant-linked Vercel team. | MUST | M5 | US-WL-004 | integration |

---

## Traceability Summary

| Milestone | MUST Count | SHOULD Count | MAY Count |
|-----------|------------|--------------|-----------|
| M1 | 27 | 4 | 0 |
| M2 | 8 | 4 | 0 |
| M3 | 17 | 3 | 0 |
| M4 | 9 | 3 | 0 |
| M5 | 26 | 13 | 0 |
| M6 | 10 | 4 | 3 |

(Counts are indicative; the backlog `24-backlog.md` decomposes each requirement into trackable stories.)

---

## Assumptions

- All requirements assume the platform is operated on AWS with the technology choices specified in `07-technical-specification.md`.
- All `MUST` requirements at a milestone are gating; the milestone does not ship without them.
- `SHOULD` requirements may be deferred with explicit sign-off from product and engineering leadership.

---

## Design Decisions

- Requirement IDs are stable across document revisions to enable traceability in PRs and incident postmortems.
- Verification methods are declared up front to constrain test planning (`19-testing-strategy.md`).
- Requirements are decoupled from implementation to allow for technical evolution without revising functional contracts.

---

## Open Questions

- Should certain `MUST` requirements graduate to compliance-track (SOC2, GDPR DPA) at V2?
- Should we adopt the EARS notation ("The system shall, when X, do Y") in a future revision for higher precision?

---

## Future Enhancements

- Auto-generate a requirements traceability matrix from this file plus the backlog.
- Linter that fails CI if a story is added without referencing an FR ID.
- Public-facing "What we promise" page derived from `MUST` requirements.

---

## Cross-References

- Stories → `04-user-stories.md`
- Non-functional → `06-nonfunctional-requirements.md`
- Backlog → `24-backlog.md`
- Roadmap → `23-milestone-roadmap.md`
- Test plan → `19-testing-strategy.md`
