# 01 — Product Requirements Document

> The product contract. Defines what Vibe is, who it serves, and what success looks like.

---

## Purpose

This Product Requirements Document (PRD) defines the product that the engineering organization, autonomous agents, and external contributors must build. It binds the technical work to a coherent commercial and user-centric objective.

This document is the source of truth for:

- The product's vision and positioning
- The problems being solved and the customers being served
- The user journeys the platform must support
- The competitive context
- The commercial model
- The scope of MVP, V2, and V3

When a feature is ambiguous, this document overrides downstream technical specifications. When this document is ambiguous, the ADRs take precedence over assumptions.

---

## Scope

In scope:

- Product mission and positioning
- Customer personas and primary jobs-to-be-done
- End-to-end user journeys
- Success metrics
- Competitive landscape
- Pricing tiers and packaging
- Versioned scope (MVP, V2, V3)

Out of scope:

- Implementation specifics (see `07-technical-specification.md`)
- API surfaces (see `09-api-specification.md`)
- Engine internals (see documents 10–14)
- Detailed user-story backlog (see `24-backlog.md`)

---

## Vision

> Every website on the public internet should be modern, fast, accessible, AI-discoverable, and continuously improved — without its owner having to think about it.

Vibe achieves this by treating website modernization as an industrial pipeline operated by autonomous software agents. The platform does for small-business websites what continuous integration did for software releases: it removes the human from the routine production loop, replacing manual labor with deterministic, observable, repeatable automation.

Vibe is not a website builder. It is not a design service. It is not an agency. It is a website upgrade utility.

---

## Mission

Make modern web standards accessible to the bottom 99% of the long tail of the web, at a price point and time-to-delivery that human services cannot match.

---

## Problem Statement

There are tens of millions of websites that are technically obsolete but commercially active. Examples include:

- A local plumbing company on a 2014 WordPress theme
- A nonprofit on a static HTML site hand-coded in 2009
- A solo consultant on a Squarespace site they no longer log into
- A real-estate broker on a templated site provided by their MLS
- A regional restaurant on a Wix site that scores 18 on Lighthouse mobile

These sites share characteristics:

- They are slow, often catastrophically so on mobile
- They are poorly indexed by modern search engines
- They are invisible to AI assistants like ChatGPT, Claude, Gemini, and Perplexity
- They lack structured metadata
- They lack accessible markup
- Their owners cannot or will not edit them
- Their owners cannot or will not pay agency rates to rebuild them

The economic gap between what these owners can pay and what a human developer can profitably charge has produced a vast underserved market.

That gap is closing as AI coding capability rises. Vibe is built into that gap.

---

## Why Now

Three trends converge:

1. **AI coding agents are now reliable enough** to produce production-quality web applications when constrained to known templates and given structured inputs.
2. **AI assistants are becoming a primary discovery channel** alongside search engines, creating a new class of optimization (AI discoverability) that legacy sites cannot satisfy.
3. **Modern web frameworks (Next.js on Vercel) have collapsed the operational cost** of running a fast, secure, observable production site to near zero.

A platform that combines all three trends can serve a market that previously required full-time human labor.

---

## Success Metrics

Success is measured along four axes.

### Product Quality Metrics

| Metric | Target (MVP) | Target (V3) |
|--------|--------------|-------------|
| Lighthouse Performance (median delivered site) | ≥ 90 | ≥ 95 |
| Lighthouse Accessibility (median) | ≥ 90 | ≥ 98 |
| Lighthouse SEO (median) | ≥ 95 | ≥ 100 |
| Lighthouse Best Practices (median) | ≥ 90 | ≥ 95 |
| % of jobs completed without human intervention | ≥ 80% | ≥ 99% |
| Mean job duration (capture → delivery) | ≤ 45 min | ≤ 8 min |
| Content fidelity (preserved heading + paragraph coverage) | ≥ 95% | ≥ 99% |

### Operational Metrics

| Metric | Target (MVP) | Target (V3) |
|--------|--------------|-------------|
| Cost of goods sold per job | ≤ $5 | ≤ $0.75 |
| Uptime of intake API | ≥ 99.5% | ≥ 99.95% |
| Mean time to recover from a failed job | ≤ 4 hours | ≤ 5 minutes |
| Customer-visible failure rate | ≤ 5% | ≤ 0.5% |

### Commercial Metrics

| Metric | Target (12 months) | Target (36 months) |
|--------|--------------------|--------------------|
| Paying customers | 500 | 25,000 |
| Monthly recurring revenue | $15K | $750K |
| Self-serve conversion rate | ≥ 4% | ≥ 8% |
| Net revenue retention | ≥ 100% | ≥ 115% |

### Trust Metrics

| Metric | Target |
|--------|--------|
| Customer NPS | ≥ 40 |
| Public deployments with measurable Lighthouse improvement vs source | ≥ 95% |
| Disputes / chargebacks | ≤ 1% |

---

## Customer Personas

### Persona 1 — Theresa, The Small Business Owner

- 52 years old, owns a regional HVAC company
- Inherited her website from a former employee in 2016
- Cannot edit the site herself
- Pays $40/month for hosting she does not understand
- Has been "meaning to update the site for years"
- Discovered Vibe via a Facebook ad

**Jobs-to-be-done:**

- "Make my website look like it was built this year, without me having to do anything."
- "Show up when people in my city Google plumbers."
- "Don't break my phone number or my Google reviews."

**Buying behavior:** One-time payment under $200. Will not subscribe to anything she does not understand.

### Persona 2 — Marcus, The Solo Marketing Consultant

- 34 years old, runs a one-person digital marketing consultancy
- Has 12 small-business clients
- Builds and maintains their websites manually using WordPress and Squarespace
- Spends 6 hours per client per month on routine upkeep
- Discovered Vibe via Hacker News

**Jobs-to-be-done:**

- "Cut my per-client labor by 80%."
- "Offer my clients a 'modern site' add-on without learning Next.js."
- "Keep ownership of the client relationship and the source code."

**Buying behavior:** White-label tier. Pays per modernization, marks up to clients.

### Persona 3 — Priya, The Agency Owner

- 41 years old, runs an 8-person creative agency
- Has 60+ live client sites in WordPress and Webflow
- Loses money on maintenance work
- Wants to migrate clients off legacy CMSes
- Discovered Vibe via a partner referral

**Jobs-to-be-done:**

- "Migrate my book of business to a modern stack without rebuilding 60 sites by hand."
- "Sell ongoing performance and SEO as a service, backed by automation."
- "Differentiate my agency from competitors who still do everything manually."

**Buying behavior:** Annual contract. High volume. Custom SLAs.

### Persona 4 — Aiden, The Indie Hacker

- 28 years old, technical, builds in public
- Owns 4 side-project sites and wants to keep them current
- Comfortable with GitHub and CLIs
- Discovered Vibe via a YouTube developer's review

**Jobs-to-be-done:**

- "Keep my projects looking professional with zero effort."
- "Get the source code so I can extend it myself."
- "Pay only when I actually use the service."

**Buying behavior:** Pay-as-you-go. Usage-based.

### Persona 5 — The Autonomous Caller (V3)

- A software system, not a human
- Could be a domain registrar offering modernization at the point of purchase, a hosting provider migrating customers off legacy infrastructure, or an AI assistant acting on behalf of an end-user
- Authenticates via API key, calls `/jobs` directly
- Consumes machine-readable reports

**Jobs-to-be-done:**

- "Reliably modernize sites at high volume with deterministic SLAs and webhooks."

---

## User Journeys

### Journey A — Self-Serve Single Site (MVP)

1. Theresa lands on `vibe.dev` from an ad.
2. She enters her website URL into the homepage form.
3. The platform performs a free preview capture: a thumbnail, a Lighthouse score, and a one-paragraph diagnosis.
4. She sees that her current site scores 31 on mobile performance and her phone number is not marked up as structured data.
5. She clicks "Modernize for $99". She enters payment details.
6. She receives an email confirmation with a job ID and a tracking link.
7. Within 45 minutes, she receives a second email: "Your new site is ready."
8. The email links to a preview URL on Vercel, a GitHub repository, and a one-page report.
9. She clicks "Approve and Publish". The site moves to her custom domain (DNS already pre-validated during intake).
10. She receives a monthly upkeep email summarizing performance and recommending optional upgrades.

### Journey B — White-Label Bulk (V2)

1. Marcus logs into the white-label dashboard.
2. He uploads a CSV of 12 client URLs, each tagged with a client identifier.
3. He selects the "agency template" branding for the report.
4. The platform schedules 12 jobs in parallel.
5. As each job completes, Marcus reviews the preview URL.
6. He approves 10, requests a regeneration on 2 with a note ("client wants services page as separate routes").
7. Approved sites are pushed to the client's GitHub organization under Marcus's GitHub App installation.
8. Vercel deployments are linked to Marcus's Vercel team.
9. Marcus invoices his clients. Vibe invoices Marcus.

### Journey C — API-First Programmatic (V3)

1. A partner's system calls `POST /v1/jobs` with `{ "url": "...", "callback_url": "..." }`.
2. The platform synchronously returns a `job_id`.
3. Internal pipeline runs.
4. On completion, the platform calls the partner's `callback_url` with a signed payload including the deployment URL, GitHub URL, and report bundle URL.
5. The partner consumes the result and surfaces it to their end user.
6. No human at Vibe touches the job.

### Journey D — Continuous Improvement Subscription (V3)

1. A customer with an existing Vibe-delivered site enrolls in continuous improvement.
2. Monthly, the platform recaptures their site and rescans for regressions.
3. Monthly, the platform rescans best-practice and AI-discoverability changes.
4. When a delta crosses a threshold, the platform automatically opens a pull request against the customer's repository.
5. If autoapproval is enabled, the PR is merged and redeployed.
6. The customer receives a monthly diff report.

---

## Competitive Analysis

| Competitor | Model | Strength | Weakness | How Vibe Differs |
|------------|-------|----------|----------|------------------|
| Wix / Squarespace / Webflow | DIY website builder | Brand, scale, ease of use | Output is not portable. Owner must build. | Vibe accepts an existing site as input. Owner builds nothing. Output is portable source code. |
| Traditional web agency | Human service | High quality, custom | $5K–$50K, weeks to months, not scalable to long tail | Vibe is one to two orders of magnitude cheaper and faster, with no quality cliff for simple sites. |
| Cursor / Replit / Lovable / V0 | AI-assisted website builder | Modern, generative | Requires the user to drive the AI. Starts from a prompt, not a real site. | Vibe is unattended. It starts from an existing URL, not a prompt. |
| Migration services (WP → Headless) | Specialist consultancy | High quality, custom | $10K–$100K, opaque, slow | Vibe productizes the same outcome. |
| Lighthouse CI / Pa11y | Auditing tools | Cheap, automated | Diagnose only. Do not fix. | Vibe both diagnoses and fixes. |

Vibe's defensible position is being the only product that:

1. Accepts an existing URL as the input.
2. Produces production-deployed code as the output.
3. Operates unattended end-to-end.
4. Optimizes for the AI-discoverability era, not only for legacy search.

---

## Pricing Models

Pricing is tiered to match the personas above.

### Tier 1 — One-Time Modernization

Single-job purchase. No subscription.

| SKU | Price | Includes |
|-----|-------|----------|
| `modernize-lite` | $99 | Up to 10 pages. Default branding. Vercel deployment. Source code. Standard report. |
| `modernize-standard` | $199 | Up to 50 pages. Custom domain setup. Enhanced report. Email delivery. |
| `modernize-pro` | $499 | Up to 250 pages. Multi-language detection. White-glove review by a human reviewer. Priority queue. |

### Tier 2 — Continuous Improvement Subscription

Add-on to any Tier 1 SKU. Cancellable monthly.

| SKU | Price | Includes |
|-----|-------|----------|
| `maintain-basic` | $29/month | Monthly recapture, regression scan, dependency updates. PR-based delivery. |
| `maintain-active` | $79/month | Weekly scan, content updates from a customer-managed source, A/B-tested SEO recommendations. |

### Tier 3 — White-Label Agency

Volume discount. Branded reports. Pooled quota.

| SKU | Price | Includes |
|-----|-------|----------|
| `agency-starter` | $499/month + $49/job | 50 jobs included. Custom report branding. Slack + email support. |
| `agency-pro` | $1,999/month + $39/job | 250 jobs included. Dedicated Slack channel. Priority queue. SLA. |

### Tier 4 — API / Platform

For programmatic callers.

| SKU | Price |
|-----|-------|
| `api-pay-as-you-go` | $79 / job. No monthly minimum. |
| `api-volume` | Custom. Starts at $25/job for committed volume. |

---

## MVP Scope

The MVP is the smallest releasable surface that proves a paying customer can complete Journey A end-to-end without human assistance from Vibe.

In scope:

- Single-URL input via web form
- Free preview capture and one-paragraph diagnosis
- Single Tier 1 SKU at $99
- Stripe checkout
- Capture engine: HAR, screenshots, internal crawl to depth 2
- Analysis engine: page inventory, content map, SEO posture, accessibility posture
- Generation engine: Next.js + TypeScript + Tailwind, opinionated default template
- SEO engine: metadata, sitemap, robots.txt, Open Graph, basic structured data, `llms.txt`
- Deployment engine: GitHub repo creation, Vercel deployment
- Email delivery of report and links
- A status page accessible by job ID
- Basic admin dashboard for the operator team

Out of scope for MVP:

- White-label tier
- Continuous improvement subscription
- API for partners
- Authenticated-site capture
- Custom domains beyond Vercel-managed
- Multi-language detection beyond English
- Human-in-the-loop review tier

---

## V2 Scope

Adds revenue surface and operational maturity.

- White-label agency tier with branded reports
- Bulk CSV intake
- Continuous improvement subscription (basic)
- Self-serve account creation
- Multi-tenant dashboard with role-based access
- Custom domain provisioning with DNS validation
- Partner GitHub Apps and Vercel team installations
- Lighthouse delta benchmarking on every job
- Customer-facing job-tracking UI
- Webhooks for job lifecycle events
- Per-tenant rate limiting and quota

---

## V3 Scope

Adds platform extensibility and autonomy.

- Public REST and webhook API
- Continuous improvement subscription (active)
- Template marketplace for generation
- Multi-framework generation targets (Astro, Remix, plain HTML)
- Multi-language detection and generation
- Authenticated-site capture
- Self-improving generation pipeline (telemetry-driven)
- Marketplace of customer-installable agents (e.g., e-commerce adapter)
- On-premises mode for regulated customers
- Fully autonomous billing, support, and customer onboarding

---

## Non-Goals

Even at V3, the platform deliberately does not:

- Provide creative redesign or rebranding services
- Offer custom application development beyond Next.js scope
- Host or operate the customer's domain DNS as a service
- Provide content writing or copywriting beyond what is extracted from the source
- Offer a visual editor or no-code builder
- Build mobile apps

These exclusions are intentional and protect the platform's focus.

---

## Assumptions

- Customers have legal authority to copy and republish content from the URLs they submit.
- Vercel, GitHub, and at least one LLM provider remain available and economically viable.
- The cost curve of LLM inference continues to decline.
- Search engines and AI assistants increasingly reward structured, accessible, semantic markup.
- Customers value source-code ownership.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| One-time pricing as the primary SKU | Lowers the barrier to first purchase. Subscription is upsell. |
| Source code always delivered to the customer | Builds trust. Differentiates from walled-garden builders. |
| Vercel + Next.js as the only initial target | Maximizes generation quality. Avoids fan-out of templates. |
| Free preview gated behind a successful capture | Demonstrates value before payment. Filters non-viable URLs. |
| White-label as a separate SKU, not a feature flag | Forces clear commercial boundaries. |
| No visual editor | Keeps focus on autonomous operation. Editing is a human-driven workflow. |

---

## Open Questions

- Should the free preview include the live Lighthouse score of the source site, the projected score of the modernized site, or both?
- How should the platform handle obvious copyright infringement in source content (e.g., stock images licensed by URL)?
- Should V2 white-label include the right to remove Vibe attribution from generated code comments?
- Is there demand for a "modernize my Shopify storefront" SKU, or does that violate the non-goal of not building e-commerce specializations until V3?

---

## Future Enhancements

- Industry-specific templates (HVAC, dental, legal, restaurants) that improve generation quality on common verticals.
- A "site health score" public ranking that customers can embed as a badge.
- A free public modernization gallery showing before-and-after improvements (with consent).
- Tiered SLAs (4-hour, 1-hour, 15-minute delivery).
- A reputation-weighted referral program for partners.

---

## Cross-References

- Roadmap and milestones → `23-milestone-roadmap.md`
- Backlog of stories implementing this PRD → `24-backlog.md`
- Cost economics that constrain pricing → `28-cost-model.md`
- Long-term direction → `29-future-vision.md`
