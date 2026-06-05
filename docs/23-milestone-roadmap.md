# 23 — Milestone Roadmap

> The staged delivery of Vibe from MVP through autonomous fleet. What we build, in what order, and why.

---

## Purpose

This document defines the milestone roadmap for Vibe. It sequences the work into discrete milestones, each with a clear definition of done, a value hypothesis, and exit criteria. It is the canonical answer to "when does this exist?"

The roadmap is intentional. A milestone is not done when the team is tired of working on it; it is done when the exit criteria are met.

---

## Scope

In scope:

- Milestone definitions
- Capabilities per milestone
- Exit criteria
- Pricing and packaging milestones
- Risk-to-milestone mapping

Out of scope:

- Long-term vision beyond V3 (`29-future-vision.md`)
- Backlog items (`24-backlog.md`)

---

## Guiding Principles

1. **Customer value first.** Each milestone produces a feature that customers can use.
2. **End-to-end early.** A thin vertical slice ships before we add breadth.
3. **Boring infrastructure first.** Observability, security, and CI exist before feature surface expands.
4. **Pricing follows capability.** We do not charge for things we do not have.
5. **Reversible commitments.** Architecture is chosen to permit upgrade without rewrite.

---

## Milestone Overview

| ID | Name | Target | Headline Capability |
|----|------|--------|---------------------|
| M0 | Foundations | Internal | Repo, infra, CI/CD, observability, security baseline |
| M1 | Capture | Beta | URL → manifest, with deterministic crawl |
| M2 | Analysis | Beta | Manifest → SiteModel with LLM summarization |
| M3 | Generation | GA | SiteModel → Next.js workspace, single-page output |
| M4 | SEO + Quality | GA | SEO engine, Lighthouse gate, llms.txt |
| M5 | Deployment | GA | GitHub + Vercel integration, full lifecycle |
| M6 | Reporting + Delivery | GA | Reports, webhooks, customer notifications |
| M7 | Self-serve Billing | GA | Stripe, plans, quotas, usage metering |
| M8 | Public API | GA | Public REST API with keys |
| M9 | Multi-page Sites | GA | Full multi-page modernization with nav |
| M10 | Whitelabel + SSO | V2 | Vibe Connect GitHub App, branded output, SSO |
| M11 | Migration Path | V2 | Data + URL migration from old to new |
| M12 | Continuous Improvement | V2 | Nightly retrains of prompt templates; agent eval |
| M13 | Autonomous Fleet | V3 | Multi-tenant optimization, schedulers, SLAs |

---

## M0 — Foundations

### Capabilities

- Monorepo skeleton (`22-monorepo-structure.md`).
- CI pipeline; pre-commit; lint, typecheck, test, security scans.
- Local development environment (`26-local-development.md`).
- Staging environment with full observability stack.
- Public status page (manual updates acceptable).
- Security baseline: secrets management, audit log, RLS-ready schema.
- Threat model skeleton; security review checklist.
- Pricing tiers defined but inert (no billing yet).

### Exit Criteria

- A new contributor can clone, bootstrap, and run a "hello job" within 30 minutes.
- All CI gates green on `main`.
- A security review of the baseline passes.
- An architecture review of the baseline passes.

### Risk Owner

- Platform team.

---

## M1 — Capture

### Capabilities

- URL intake: validation, denylist, normalization, SSRF defense.
- Capture worker pool with Playwright.
- Crawl policy: same-origin, depth limit, page cap, time cap, asset cap.
- HAR per page; manifest v1 (`10-capture-engine.md`).
- Storage: S3 layout with tenant prefix.
- Tenant creation, auth (magic link), basic dashboard.

### Exit Criteria

- 95% of a representative set of marketing sites capture to a valid manifest.
- Bot-walled sites are recognized and reported as partial.
- No SSRF or egress policy violations in adversarial tests.
- Latency p95 for a 30-page capture ≤ 10 minutes.

### Risk Owner

- Capture team.

---

## M2 — Analysis

### Capabilities

- HTML parser with content extraction.
- Navigation graph construction.
- Page role classification.
- SEO and a11y audits.
- Tech detection.
- LLM summarization with cost guard.
- SiteModel v1 emitted (`11-analysis-engine.md`).

### Exit Criteria

- SiteModel quality reviewed by humans for a sample of 30 sites; ≥ 90% judged usable.
- LLM cost per analysis within budget.
- LLM provider failover works in tests.

### Risk Owner

- Agents team.

---

## M3 — Generation (Single-Page)

### Capabilities

- Generation engine scaffold (`12-generation-engine.md`).
- Single-page Next.js 14 app generation.
- Theme system with one starter.
- Build pipeline that produces a working `next build` for the fixture.
- Self-repair loop: up to 3 attempts; quarantine on failure.
- S3 workspace storage.

### Exit Criteria

- Generated single-page app builds successfully for 100% of fixture sites.
- Lighthouse Performance ≥ 90 on the median fixture.
- Self-repair rate ≤ 20% on the median fixture.

### Risk Owner

- Generation team.

---

## M4 — SEO + Quality

### Capabilities

- SEO engine (`13-seo-engine.md`): metadata, JSON-LD, OG, sitemap, robots.
- `llms.txt` and `llms-full.txt` generation.
- Quality reviewer agent (`03-agent-architecture.md`).
- Lighthouse + axe gates.
- A11y minimum: WCAG 2.2 AA on the median fixture.

### Exit Criteria

- Lighthouse Performance ≥ 90, SEO = 100, Best Practices = 100, A11y ≥ 95 on the median fixture.
- `llms.txt` validates against the published schema.
- JSON-LD validates against schema.org.

### Risk Owner

- SEO + Agents teams.

---

## M5 — Deployment

### Capabilities

- GitHub App + Connect App foundation (`15-github-integration.md`).
- Vercel integration (`16-vercel-integration.md`).
- Repo creation, push, project creation, deploy.
- Custom domain support with DNS verification.
- Compensation registry for partial failures.
- Idempotent operations.

### Exit Criteria

- 95% of jobs deploy to a Vercel preview URL successfully.
- Compensation registry exercises documented in tests.
- No cross-tenant access in security tests.

### Risk Owner

- Deployment team.

---

## M6 — Reporting + Delivery

### Capabilities

- Report generation: per-job and per-tenant summaries.
- Webhook system with HMAC signatures.
- Customer email notifications at lifecycle milestones.
- Customer dashboard with job history and report download.

### Exit Criteria

- Reports include: original URL, target URL, manifest summary, SiteModel diff highlights, Lighthouse scores, sitemap reference.
- Webhook delivery success ≥ 99% over 7 days in staging.
- Customers receive notification within 1 minute of completion.

### Risk Owner

- Delivery + Web teams.

---

## M7 — Self-Serve Billing

### Capabilities

- Stripe Checkout and Customer Portal integration.
- Subscription state machine.
- Plan-based quotas: jobs/month, LLM spend cap, capture pages.
- Usage metering and overage handling.
- Invoice generation.
- Billing webhook ingestion.

### Exit Criteria

- Customer can sign up, pick a plan, run a job, and be billed correctly.
- Plan upgrades and downgrades work without operator intervention.
- Webhook ingestion is idempotent and survives replay.

### Risk Owner

- Billing team.

---

## M8 — Public API

### Capabilities

- REST API v1 (`09-api-specification.md`).
- API key management.
- Webhook subscription.
- Public SDK (TypeScript).
- Rate limiting per key.
- OpenAPI spec published.

### Exit Criteria

- API passes the published SLA in staging.
- SDK smoke tests succeed for all endpoints.
- Documentation is reviewed by an external developer.

### Risk Owner

- API team.

---

## M9 — Multi-Page Sites

### Capabilities

- Multi-page generation with shared layout and navigation.
- Component library expansion.
- Custom theme authoring for paid tiers.
- Content data layer (MDX/JSON).
- Per-section regeneration without full rebuild.

### Exit Criteria

- 95% of multi-page fixtures build successfully.
- Customer can request a "deep scan" that produces multi-page output.

### Risk Owner

- Generation team.

---

## M10 — Whitelabel + SSO (V2)

### Capabilities

- Vibe Connect GitHub App (`15-github-integration.md`).
- Branded output: customer logo, colors, footer in the generated app.
- SAML/OIDC SSO for enterprise customers.
- Custom Vercel team per tenant.

### Exit Criteria

- Enterprise tenant can self-onboard with SSO and produce a branded site.
- Audit and compliance: SOC 2 Type I evidence collected.

### Risk Owner

- Enterprise team.

---

## M11 — Migration Path (V2)

### Capabilities

- 301 redirect map generation from old URLs to new.
- Asset carry-over (images, PDFs) from source to target.
- DNS cutover playbook.
- Decommissioning of source if hosted with us.

### Exit Criteria

- Migration tested on three real customer migrations.
- Cutover playbook reviewed by external SRE.

### Risk Owner

- Migration team.

---

## M12 — Continuous Improvement (V2)

### Capabilities

- Nightly agent eval using offline traces.
- Prompt template versioning with promotion gates.
- Anomaly detection on per-tenant cost spikes.
- Auto-generated retrospective reports for ops.

### Exit Criteria

- Eval harness reports SLO compliance per agent per week.
- Three prompt improvements promoted via the harness in the first month.

### Risk Owner

- Agents + Platform teams.

---

## M13 — Autonomous Fleet (V3)

### Capabilities

- Continuous improvement loops across the fleet.
- Per-customer SLAs; dynamic agent tuning.
- Fleet-wide rollouts of prompt and config changes with canary.
- Long-running "concierge" jobs that improve a site over time.

### Exit Criteria

- 1% of GA customers opt into autonomous mode.
- Net Promoter Score ≥ 50 for autonomous customers.

### Risk Owner

- Eng leads.

---

## Pricing Milestones

| Milestone | Plan | Price |
|-----------|------|-------|
| M1 | Free preview | $0 — single job, capped output |
| M3 | Starter | $19/month — 3 jobs/month |
| M5 | Growth | $99/month — 20 jobs/month, custom domain |
| M9 | Pro | $299/month — unlimited, multi-page, priority |
| M10 | Enterprise | Custom — SSO, whitelabel, SLA |

See `01-product-requirements-document.md` for the full pricing rationale.

---

## Capability Matrix (Excerpt)

| Capability | M1 | M3 | M5 | M7 | M9 | M10 |
|------------|----|----|----|----|----|-----|
| Single URL capture | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Single-page generation |   | ✓ | ✓ | ✓ | ✓ | ✓ |
| Multi-page generation |   |   |   |   | ✓ | ✓ |
| GitHub + Vercel deploy |   |   | ✓ | ✓ | ✓ | ✓ |
| Custom domain |   |   | ✓ | ✓ | ✓ | ✓ |
| Reports + webhooks |   |   |   | ✓ | ✓ | ✓ |
| Stripe billing |   |   |   | ✓ | ✓ | ✓ |
| White-label |   |   |   |   |   | ✓ |
| SSO |   |   |   |   |   | ✓ |

---

## Risk Mapping (High-Level)

| Risk | First Addressed |
|------|-----------------|
| SSRF / egress abuse | M1 |
| LLM cost blow-up | M2 |
| Generation quality | M3 |
| SEO regression | M4 |
| Deploy failures | M5 |
| Webhook delivery | M6 |
| Billing fraud | M7 |
| API abuse | M8 |
| Tenant isolation | M10 |

See `25-risk-analysis.md` for the full risk register.

---

## Assumptions

- The team can sustain a monthly cadence for milestones.
- Customer demand follows the demonstrated value curve.
- The pricing strategy is acceptable to the market.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Thin vertical slice per milestone | Customer feedback comes early. |
| Pricing appears only when capability exists | Trust-building. |
| Whitelabel and SSO are V2 | Concentrate MVP on core product. |
| Autonomous mode is V3 | Avoid over-promising at MVP. |

---

## Open Questions

- Should M5 include custom domain or defer to M6 to reduce M5 scope?
- Should M8 include the public SDK or defer to V2?

---

## Future Enhancements

- Continuous discovery loop: post-M13, the platform proposes new features to customers.
- Industry-specific templates (restaurants, law firms, etc.) starting V3.

---

## Cross-References

- Product requirements → `01-product-requirements-document.md`
- Risk register → `25-risk-analysis.md`
- Backlog → `24-backlog.md`
- Future vision → `29-future-vision.md`
