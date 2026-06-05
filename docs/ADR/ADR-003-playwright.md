# ADR-003 — Playwright as the Capture Engine

- **Status:** Accepted
- **Date:** 2026-06-01
- **Deciders:** Capture team, Eng leads
- **Supersedes:** —
- **Superseded by:** —

---

## Context

The capture engine is the entry point of every job. It must:

- Render modern JavaScript-heavy sites faithfully.
- Produce high-fidelity, real-browser output (HTML, CSS, screenshots, HAR).
- Be safe against SSRF and bot-wall mitigation.
- Operate at scale with thousands of jobs per day.
- Be cost-effective.
- Be controllable (timeout, page cap, asset cap, depth limit).

Alternatives considered: headless Chromium via Puppeteer, Playwright, Selenium, a managed scraping API, and a custom build on top of Chrome DevTools Protocol (CDP).

---

## Decision

Adopt **Playwright** (Chromium) as the primary capture engine.

- A managed **browser pool** in a separate Fargate service (`services/browser-pool`).
- Workers request a context per job; the pool provides it via a queue.
- Per-job limits enforced by the orchestrator and by the worker itself.
- A custom URL normalization and SSRF guard fronting the engine.

---

## Rationale

### Why Playwright

- **Mature, multi-language, multi-browser.** We use Chromium for MVP but can extend to Firefox/WebKit if a site requires it.
- **Modern web fidelity.** Handles SPAs, hydration waits, and modern JS APIs cleanly.
- **Strong automation primitives.** `waitForLoadState('networkidle')`, custom `waitForFunction`, HAR recording, tracing.
- **Active development.** Backed by Microsoft; regular releases; large community.
- **TypeScript SDK** aligns with our internal types.
- **Better defaults** than Puppeteer for our needs (auto-wait, retry, timeouts).

### Why not Puppeteer

- **Pro:** widely used; good for simple cases.
- **Con:** less mature for complex automation; lower-level for our needs; no first-class multi-browser.
- **Verdict:** Playwright is the better fit; the cost of learning Puppeteer is similar.

### Why not Selenium

- **Con:** heavy, slow, brittle, and the SMB marketing site target is not Selenium's sweet spot.
- **Verdict:** rejected.

### Why not a managed scraping API

- **Examples:** Browserless, ScrapingBee, Apify.
- **Pro:** offload browser ops.
- **Con:** vendor lock-in, cost at scale, less control over fingerprint and SSRF defense, harder to debug.
- **Verdict:** rejected for MVP. May revisit at V2 if the cost-benefit shifts.

### Why not a custom build on raw CDP

- **Con:** high engineering cost; we would reimplement a browser automation library.
- **Verdict:** rejected.

### Why a managed pool (not per-task browser)

- **Boot time** of a Chromium instance is 1–3 seconds. A warm pool eliminates this from the critical path.
- **Memory cost** of an idle browser is significant; sharing a small pool keeps the bill sane.
- **Lifecycle management** (context creation, isolation, teardown) is centralized.

### Why SSRF defense at the engine, not at the network

- The defense-in-depth approach is: deny at the URL normalizer, then deny at the egress firewall, then deny at IMDSv2 hop limit. Multiple layers; one alone is insufficient.

### Why HAR + screenshots, not just HTML

- The analysis engine needs both the rendered DOM and the wire-level interactions. HAR captures redirects, response codes, and asset bytes. Screenshots are a trust artifact for the customer.

---

## Consequences

### Positive

- High-fidelity capture across modern sites.
- Predictable cost at scale (warm pool + caps).
- Strong SSRF and egress defenses.
- Operationally simple: one engine, one pool.

### Negative

- **Chromium-only at MVP.** Some sites behave differently in Firefox or WebKit. We accept this trade-off.
- **Pool sizing is a tuning exercise.** Mitigated by autoscaling on queue depth and capacity tests.
- **Bot walls remain a risk.** Mitigated by detection and a single retry; full residential IP support is V2.
- **Browser ops are non-trivial.** A dedicated capture team owns the engine.

### Neutral

- The capture engine is one of the few places where our system intentionally executes untrusted code (in a sandbox). The risk is documented in the threat model.

---

## Alternatives Considered

| Option | Verdict |
|--------|---------|
| Playwright (Chromium) | **Chosen** |
| Puppeteer | Rejected — less mature for complex needs |
| Selenium | Rejected — heavy, brittle |
| Managed scraping API | Rejected — vendor cost and lock-in |
| Raw CDP | Rejected — high engineering cost |
| A "lite" HTTP-only capture | Rejected — cannot render modern sites |

---

## Notes

- We do not store the rendered HTML long-term; we re-derive or summarize at the analysis phase.
- The browser pool is its own service to enable independent scaling.
- A "headless-disabled" mode (visible browser in dev) is available for local debugging.
- We will revisit multi-browser support (Firefox, WebKit) when telemetry shows material coverage gaps.
