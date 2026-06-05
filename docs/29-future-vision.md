# 29 — Future Vision

> Where Vibe goes after V3. The long arc of the product, the platform, and the company.

---

## Purpose

This document captures the future vision for Vibe beyond the V3 horizon. It articulates the north star, the major bets, and the long-term direction so that current decisions can be made with the future in mind.

The vision is a destination, not a roadmap. Concrete work is captured in the roadmap (`23-milestone-roadmap.md`) and the backlog (`24-backlog.md`). The vision keeps the work pointed in the right direction.

---

## Scope

In scope:

- Long-term product direction
- Long-term platform direction
- Long-term company direction
- Bets we are willing to make
- Bets we explicitly avoid

Out of scope:

- Specific feature proposals (these are backlog candidates)

---

## North Star

> Vibe becomes the default partner for small and mid-sized businesses to keep their web presence modern, fast, and discoverable — without ever needing to hire an agency again.

The job-to-be-done is "modernize my website" and we make it feel like a one-click action that pays for itself.

---

## Strategic Bets

### 1. The modernized site is a living artifact, not a one-time output

The first generation is the beginning. A modernized site should improve itself: better content, better SEO, better performance, better accessibility, all driven by ongoing signals. The customer should perceive the platform as a partner that takes care of their site, not a vendor that delivers a PDF and disappears.

Implications:

- A "concierge" mode where the platform schedules periodic re-scans and proposes improvements.
- A "trust dashboard" that shows the customer's site health over time.
- A "competitive awareness" feature that surfaces what similar businesses are doing well.

### 2. SEO and AI discoverability are the wedge

The `llms.txt` initiative is a long-term differentiator. As AI agents increasingly mediate discovery, sites that expose themselves well to those agents will win. Vibe's bet: by owning this surface, we become the de facto modernization partner for the AI-first web.

Implications:

- Continued investment in `llms.txt` and the surrounding ecosystem (e.g., AI-visible schemas, agent-readable site maps).
- Partnerships with the major AI platforms to be their preferred modernization partner.
- A research arm that publishes insights on AI discoverability.

### 3. The agent platform itself is a product

The same agent orchestration that modernizes websites can be applied to other high-volume, content-heavy knowledge work: legal document modernization, knowledge base maintenance, real-estate listing modernization, franchise site deployment at scale. The agent platform becomes a horizontal product.

Implications:

- A clear separation between the "modernization product" and the "agent platform" in code and ops.
- A multi-tenant platform that supports custom agent definitions.
- A pricing model that scales with value, not just volume.

### 4. Vertical depth unlocks premium tiers

Generic modernization is the entry point. Vertical depth — restaurants, law firms, medical practices, franchises — drives the premium tier. The vertical templates are not cosmetic; they encode domain knowledge (NAP for local SEO, schema.org for medical practices, accessibility for public-sector sites).

Implications:

- A templating layer in the generation engine that supports vertical knowledge packs.
- A research effort on each vertical we enter.
- Compliance posture evolves (HIPAA, public-sector) as verticals demand it.

### 5. The source code is always the customer's, and that's a feature

Vibe never holds customers hostage. The code lives in their GitHub; the deploys live in their Vercel. This is a long-term trust asset. We compete on value, not lock-in.

Implications:

- Continued investment in clean, idiomatic code generation.
- Easy export and self-hosting of the entire pipeline.
- A "no Vibe, just the code" option for the most paranoid customers.

---

## Long-Term Product Direction

### Vibe 4 — Industry-Tuned Modernization (post-V3)

- Vertical templates for top 5 industries.
- Continuous improvement loops enabled by default for paid plans.
- A "site health" score that customers can display as a trust badge.

### Vibe 5 — The AI-Visible Web (post-V4)

- Per-tenant `llms.txt` registry; customer-facing analytics on AI visits (where supported by AI platforms).
- Tools to author "AI answers about my business" content.
- Integration with major AI assistants' indexing systems.

### Vibe 6 — Multi-Modal Modernization (post-V5)

- Modernize not just text but video, audio, and image content.
- Multilingual generation as a first-class feature.
- Personalized site variants for different visitor segments.

### Vibe 7 — The Autonomous Site (post-V6)

- The site improves itself nightly, with human approval for material changes.
- Customer spends zero hours on maintenance.
- The platform becomes the operating system for the customer's web presence.

---

## Long-Term Platform Direction

### Multi-Region

- Multi-region active-active by V5.
- Customer data residency controls.
- Per-region compliance posture.

### Multi-Cloud

- AWS primary; GCP secondary for specific regions (e.g., data residency).
- The orchestrator remains portable (Temporal, Kubernetes-friendly).

### Real-Time

- Live preview during modernization.
- Customer can see changes as they happen, not at the end.

### Composition

- Third parties can author agents and themes via a public marketplace.
- The platform's value is amplified by an ecosystem.

### Data

- A unified customer data layer.
- Insights across customers, with strict privacy boundaries, drive product improvements.

---

## Long-Term Company Direction

### Mission

> Make the web a better, faster, more accessible place — one small business at a time.

### Values

1. **Customer obsession.** Every decision is filtered through "what would the customer want?"
2. **Boring infrastructure.** The platform must be reliable, not flashy.
3. **Honest pricing.** We charge for value, not for lock-in.
4. **Sustainable pace.** The team is a long-term system, not a sprint.
5. **Open standards.** We bet on `llms.txt`, schema.org, Web Vitals, and open formats.

### Markets

- Initial: English-speaking SMBs in North America.
- Expansion: Western Europe, Australia, English-speaking Asia.
- Future: Latin America, parts of Africa, India, with localization as a precondition.

### Funding Posture

- Bootstrapped or seed-funded for the first two years.
- Series A in year 3 to fund multi-region and enterprise.
- Profitability target by year 4.

### Team

- 5–10 in year 1 (founders + 3–8 engineers).
- 20–30 in year 2 (adding go-to-market).
- 50+ in year 3 (multi-region, enterprise).
- Distributed-first, with hubs in 2–3 cities.

---

## Bets We Explicitly Avoid

### A General-Purpose Website Builder

We are not Squarespace, Wix, or Webflow. Customers come to us to *leave* a builder, not to start in one. We are the modernization layer; builders are the alternative to leaving, not our competition.

### A Hosting Business

We do not host sites. Vercel does. We do not sell domains. Customers keep theirs. We stay focused on the intelligence and the integration.

### A Pure Agency

We do not manually build sites. The value is in the automation. We scale through code, not consultants.

### A "Replace Your Dev Team" Platform

We deliver modernized sites, not custom software. Customers who need ongoing engineering buy engineering. We are the first 80%, not the last 20%.

### A Marketplace for Templates

Third-party templates risk quality dilution. We may allow vetted partners in V7+, but we will not be a template marketplace in the early years.

---

## Asks of Future Engineering

- **Schema-first development.** Cross-language contracts are the priority. New features land in `schemas/` first.
- **Cost as a first-class metric.** Every LLM-using feature has a cost envelope and a dashboard.
- **Reversibility.** Schemas are forward-only; data structures are versioned; APIs are versioned.
- **Privacy by default.** PII is treated as a special class; redaction is the default.
- **Documentation as code.** Docs are reviewed in PRs; docs are tested.

---

## Asks of Future Product

- **Customer feedback is the input.** Roadmap adjustments come from customers, not from internal preferences.
- **Pricing follows capability.** We do not charge for what we cannot deliver.
- **Experiments are the norm.** Major features ship behind flags and are measured.
- **The free tier is honest.** A customer who only uses the free tier is not a problem; they are a future customer or a happy evangelist.

---

## The Long Game

Vibe is building a company that, in a decade, should be known as the partner that made the web better for a million small businesses. That is a long game. The roadmap is a year. The vision is ten.

We will be tempted to chase short-term growth at the cost of long-term trust. We will resist. The bet is that trust compounds, and that compounding trust compounds revenue.

---

## Open Questions

- How will AI agents' discovery models evolve, and how do we stay ahead?
- When does a vertical investment make sense, and which verticals first?
- How do we maintain a "small team" feel as we scale?
- What is our stance on M&A — are we a buyer, a target, or neither?

---

## Future Enhancements

- A quarterly vision-review process.
- Public-facing thought leadership on AI discoverability.
- An annual customer conference.

---

## Cross-References

- Roadmap → `23-milestone-roadmap.md`
- Risks → `25-risk-analysis.md`
- Vision (current) → `00-project-overview.md`
