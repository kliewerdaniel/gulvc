# 28 — Cost Model

> How we think about cost, what a job costs us, what a customer costs us, and what we charge.

---

## Purpose

This document defines the cost model for Vibe. It explains how we measure cost, what a single job costs to run, what each customer segment costs at scale, and how pricing maps to gross margin.

The cost model is the bridge between engineering choices and business outcomes. Every team can read it; every team can contribute to keeping it honest.

---

## Scope

In scope:

- Cost categories (compute, storage, LLM, third-party, overhead)
- Per-job cost derivation
- Per-tenant cost models
- Per-tier break-even and margin
- Forecasts and sensitivities
- Cost control mechanisms

Out of scope:

- Marketing and sales cost (handled by leadership)
- Pricing strategy and packaging (covered in `01-product-requirements-document.md`)

---

## Cost Categories

The platform's cost of goods sold (COGS) has four major categories.

| Category | Examples | Driver |
|----------|----------|--------|
| Compute | ECS Fargate, RDS, ElastiCache, S3, Vercel | Job volume, capture depth, generation time |
| LLM | LLM API spend | Per-phase token usage |
| Third-party SaaS | Stripe fees, Sentry, Vercel team | Job count, customers |
| Overhead | Observability, secrets, backups, support | Customer count |

---

## Per-Job Cost Model (MVP)

The reference job for costing: a 30-page marketing site, single-page generation, GitHub + Vercel deploy.

### Cost Derivation

| Phase | Resource | Estimate | Notes |
|-------|----------|----------|-------|
| Capture | Browser pool (Playwright on Fargate) | $0.40 | 5 min @ 0.25 vCPU, 1 GB, x86 |
| Capture | S3 (HAR, screenshots) | $0.04 | 200 MB |
| Analysis | LLM (small/cheap models) | $0.50 | 1.2M input, 0.3M output tokens |
| Generation | Fargate | $0.20 | 3 min @ 0.5 vCPU, 1 GB |
| Generation | LLM (self-repair attempts) | $0.30 | 0.4M input, 0.1M output |
| SEO | LLM | $0.05 | 0.1M input, 0.05M output |
| Quality | Lighthouse + axe (self-hosted) | $0.05 | 2 min compute |
| Deploy | GitHub App | $0.00 | within free tier |
| Deploy | Vercel | $0.00 | within free tier |
| Deploy | Fargate | $0.05 | orchestrator only |
| Storage | S3 (workspace) | $0.02 | 100 MB |
| Delivery | Webhook + email | $0.01 | minimal |
| Overhead | Observability share | $0.10 | amortized |
| **Total per job** | | **$1.72** | |

### Cost Envelope

- The hard ceiling per job is **$5.00** (MVP).
- Soft target is **$1.50–$2.00** for the reference job.
- The per-job envelope is enforced in code: an agent that exceeds its sub-envelope halts and reports `cost_exceeded`.

### LLM Cost Specifics

- Capture agent: no LLM at MVP.
- Analysis agent: dominant cost. Cap at $0.60/job.
- Generation agent: secondary cost. Cap at $0.40/job.
- SEO agent: small. Cap at $0.10/job.
- Total LLM cap per job: **$1.10**, with $0.40 headroom.

### Cost Variation by Job Type

| Job type | Estimated cost |
|----------|----------------|
| Free preview (single page, low fidelity) | $0.40 |
| Starter (single-page modernization) | $1.50 |
| Growth (multi-page, custom domain) | $3.50 |
| Pro (deep modernization, multi-page, custom theme) | $5.00 (capped) |
| Whitelabel enterprise (V2) | $7.00 (capped) |

---

## Per-Tenant Cost Model

A typical tenant runs 3–20 jobs/month. Customer cost is the sum of job costs plus overhead.

| Component | Notes |
|-----------|-------|
| Job cost | Sum of per-job costs |
| Storage | $0.02/GB-month; artifacts retained per NFRs |
| Support | $0.10/tenant-month amortized |
| Overhead | $0.50/tenant-month amortized |

A Growth tenant running 20 jobs at $3.50 each is $70/month gross, with ~$10 overhead = $80/month cost.

---

## Per-Tier Break-Even

| Tier | Price (monthly) | Estimated COGS/tenant | Gross Margin |
|------|-----------------|------------------------|--------------|
| Free | $0 | $0.50 (1 preview) | Loss-leader |
| Starter | $19 | $5 | 74% |
| Growth | $99 | $25 | 75% |
| Pro | $299 | $60 | 80% |
| Enterprise (V2) | Custom | Custom | 70%+ target |

Free-tier COGS is bounded by the preview's hard caps (10 MB/page, 3 pages, no deploy, no custom domain).

---

## Cost Forecasts

Assumptions for year 1 (MVP ramp):

- Months 1–3: 50 customers, 10 paying, 30 free.
- Months 4–6: 200 customers, 50 paying.
- Months 7–9: 600 customers, 200 paying.
- Months 10–12: 1,500 customers, 500 paying.

| Quarter | COGS/month | Revenue/month | Gross margin |
|---------|------------|---------------|--------------|
| Q1 | $1,500 | $1,000 | -50% (investment) |
| Q2 | $4,000 | $5,000 | 20% |
| Q3 | $9,000 | $20,000 | 55% |
| Q4 | $20,000 | $60,000 | 67% |

Crossover to positive gross margin target: Q3.

---

## Cost Controls

### Compute

- Auto-scaling with cooldown.
- Off-hours minimum to zero.
- Reserved capacity for predictable baselines at V2.

### LLM

- Per-agent cost envelopes.
- Per-tenant daily spend cap.
- Provider failover to lower-cost options.
- Prompt templating to reduce token usage.
- Caching of repeated inputs.
- Output length caps.

### Storage

- Lifecycle policies per NFRs.
- Compression for captured assets.
- Tenant-scoped quotas.

### Third-Party

- Annual commits for predictable vendors.
- Spend alerts in Stripe.

### Overhead

- Observability budget is hard-capped per `18-observability.md`.
- Support tier structure evolves with scale.

---

## Cost Observability

- A cost dashboard tracks per-job cost, per-tenant cost, per-tier cost.
- Anomalies trigger alerts.
- Daily reconciliation against AWS Cost Explorer and LLM provider invoices.
- A weekly report is delivered to leadership.

---

## Sensitivity Analysis

What moves the cost most?

| Variable | 10% change → cost change |
|----------|---------------------------|
| LLM token price | ~5% COGS |
| Capture time per page | ~3% COGS |
| Generation build time | ~2% COGS |
| Job volume | linear |
| Storage retention | ~1% COGS |

LLM is the most sensitive variable. We monitor token price changes and provider pricing carefully.

---

## Cost vs. Value

The pricing strategy is anchored in customer value, not cost. A modernized website saves the customer thousands of dollars in agency fees and yields faster, more accessible, more discoverable sites. The cost model is the floor; pricing is positioned for value capture.

---

## Decision: Pricing Anchors

- **Free preview**: drives top-of-funnel; loss-leader by design.
- **Starter**: pays for itself if the customer would have used a freelancer; price point is a no-brainer.
- **Growth**: covers multi-page sites; price reflects ongoing value.
- **Pro**: includes unlimited usage for small businesses; price reflects predictability.
- **Enterprise**: scope-based; includes SSO, whitelabel, SLAs.

---

## Decision: LLM Provider Strategy

- Multi-provider from day 1; the router picks the best model per phase.
- Default to the cheapest model that meets the quality bar.
- A quarterly prompt improvement cycle reduces token usage without quality loss.

---

## Assumptions

- The reference job is representative; large variance is acknowledged.
- The free-tier cap holds abuse to acceptable levels.
- The forecast assumes steady conversion rates and low churn.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Per-job cost as a first-class number | Drives engineering choices. |
| Per-tenant cap on LLM spend | Protects against abuse. |
| Per-tier break-even targeted at 70%+ gross | Sustainable SaaS margin. |
| Multi-provider LLM | Cost, resilience, optionality. |
| Observability as overhead, not feature | Discipline. |

---

## Open Questions

- Should we publish a public cost dashboard for trust?
- Should we offer a "bring your own LLM key" tier in V3?

---

## Future Enhancements

- Per-customer cost allocation in V2 (for cost-plus billing).
- A "spend down" alert when a tenant approaches 80% of plan limits.
- Cost-aware routing: if a job is over budget, the orchestrator downgrades to cheaper models for non-critical phases.

---

## Cross-References

- Roadmap → `23-milestone-roadmap.md`
- Risk → `25-risk-analysis.md`
- Agents → `03-agent-architecture.md`
- Pricing → `01-product-requirements-document.md`
