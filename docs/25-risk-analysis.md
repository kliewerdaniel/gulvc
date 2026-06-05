# 25 — Risk Analysis

> The risks the platform faces, their likelihood, their impact, and the mitigations that keep them at bay.

---

## Purpose

This document catalogues the risks Vibe faces across product, technology, security, business, and operations. For each risk, it records likelihood, impact, mitigations, owners, and triggers.

Risk management is continuous. The register is reviewed at every milestone gate and updated as the platform and its environment evolve.

---

## Scope

In scope:

- Product and customer risks
- Technical and architecture risks
- Security and compliance risks
- Operational risks
- Business and market risks
- Vendor and provider risks

Out of scope:

- General company-level risks (handled by leadership)

---

## Methodology

Risks are scored on two axes:

- **Likelihood**: 1 (very low) to 5 (very high)
- **Impact**: 1 (very low) to 5 (very high)
- **Score** = Likelihood × Impact, range 1–25
- **Tier**:
  - Score ≥ 16: Tier 1 (must mitigate, monitored continuously)
  - 8 ≤ Score < 16: Tier 2 (mitigate, monitored)
  - Score < 8: Tier 3 (accept, monitored)

Each risk has:

- Owner
- Trigger (what we look for)
- Mitigation
- Residual risk after mitigation

---

## Risk Register

### R-001 — LLM cost blow-up

- **Category:** Cost / Vendor
- **Likelihood:** 4
- **Impact:** 3
- **Score:** 12 (Tier 2)
- **Owner:** Agents team
- **Trigger:** LLM cost per job increases 20% over 7 days; per-tenant LLM spend grows abnormally.
- **Mitigation:** Per-agent cost envelopes, per-tenant daily spend cap, provider failover, prompt templating, output length caps, response caching for repeated inputs, model selection per phase.
- **Residual:** 6 (Tier 3)

### R-002 — Generation quality regression

- **Category:** Product
- **Likelihood:** 4
- **Impact:** 4
- **Score:** 16 (Tier 1)
- **Owner:** Generation team
- **Trigger:** Lighthouse median drops 5 points over 7 days; build success rate drops below 90%; self-repair rate climbs above 30%.
- **Mitigation:** Quality gate enforces SLOs; per-prompt regression tests in CI; canary rollouts; golden dataset eval nightly; immediate revert path.
- **Residual:** 8 (Tier 2)

### R-003 — SSRF / egress abuse

- **Category:** Security
- **Likelihood:** 3
- **Impact:** 5
- **Score:** 15 (Tier 2)
- **Owner:** Security + Capture
- **Trigger:** A new IP range is observed in logs; an outbound connection to metadata service; an unusual URL pattern.
- **Mitigation:** SSRF defense in URL normalizer, private-IP filter, IMDSv2 with hop-limit 1, egress firewall, sandboxed containers, dedicated test harness.
- **Residual:** 6 (Tier 3)

### R-004 — Cross-tenant data exposure

- **Category:** Security
- **Likelihood:** 2
- **Impact:** 5
- **Score:** 10 (Tier 2)
- **Owner:** Security
- **Trigger:** A query in audit log touches a tenant without authorization; a smoke test fails the isolation check.
- **Mitigation:** Tenant filter enforced at the repository layer with a test-asserted SQLAlchemy event hook; defense-in-depth RLS in V2; fuzzing harness weekly; customer isolation review on every PR touching repositories.
- **Residual:** 4 (Tier 3)

### R-005 — GitHub App compromise

- **Category:** Security / Vendor
- **Likelihood:** 2
- **Impact:** 5
- **Score:** 10 (Tier 2)
- **Owner:** Security
- **Trigger:** Anomalous API activity from the App; suspicious webhook traffic.
- **Mitigation:** Private key in Secrets Manager; minimal scope; webhook secret rotation; per-repo access scope; immediate uninstall path; rate limits at GitHub and at our API.
- **Residual:** 4 (Tier 3)

### R-006 — Vercel outage

- **Category:** Vendor
- **Likelihood:** 3
- **Impact:** 3
- **Score:** 9 (Tier 2)
- **Owner:** Deployment
- **Trigger:** Vercel status page shows degradation for relevant regions; build error rate climbs.
- **Mitigation:** Retry with exponential backoff; surfacing in customer report; status page incident; backlog jobs continue and deploy when Vercel recovers; no SLO credits at MVP.
- **Residual:** 6 (Tier 3)

### R-007 — GitHub outage

- **Category:** Vendor
- **Likelihood:** 3
- **Impact:** 3
- **Score:** 9 (Tier 2)
- **Owner:** Deployment
- **Trigger:** GitHub status degraded; webhook lag > 1 min.
- **Mitigation:** Idempotent job state; jobs continue to capture/analyze/generate; deploy phase waits; retries.
- **Residual:** 6 (Tier 3)

### R-008 — LLM provider outage

- **Category:** Vendor
- **Likelihood:** 4
- **Impact:** 4
- **Score:** 16 (Tier 1)
- **Owner:** Agents
- **Trigger:** Provider returns 5xx > 1% over 5 min; latency p95 > 5 s.
- **Mitigation:** Multi-provider routing with failover; recorded fallback responses; per-phase model choice; per-agent cost envelope includes a buffer; customer notification.
- **Residual:** 8 (Tier 2)

### R-009 — Bot-walled source

- **Category:** Product
- **Likelihood:** 4
- **Impact:** 2
- **Score:** 8 (Tier 2)
- **Owner:** Capture
- **Trigger:** Manifest completeness < 70% for a sample set; many tenants hit the same domain with bot walls.
- **Mitigation:** Detect and report; manual retry with optional residential IP (V2); partial delivery is acceptable; clear report.
- **Residual:** 4 (Tier 3)

### R-010 — LLM hallucination in analysis or generation

- **Category:** Quality
- **Likelihood:** 3
- **Impact:** 4
- **Score:** 12 (Tier 2)
- **Owner:** Agents + Generation
- **Trigger:** Quality gate flags content divergence; self-repair rate climbs; customer escalations about wrong content.
- **Mitigation:** Grounding via extracted content; low temperature; structured outputs; spot-check diffs in review; per-prompt evals; LLM output validator as a separate agent.
- **Residual:** 6 (Tier 3)

### R-011 — Self-repair loop divergence

- **Category:** Quality
- **Likelihood:** 2
- **Impact:** 3
- **Score:** 6 (Tier 3)
- **Owner:** Generation
- **Trigger:** A self-repair attempt fails in a way that introduces a new error not in the original.
- **Mitigation:** Cap at 3 attempts; isolate generated code from prior artifact between attempts; log every attempt; fail closed.
- **Residual:** 4 (Tier 3)

### R-012 — Custom domain DNS issues

- **Category:** Operations
- **Likelihood:** 4
- **Impact:** 2
- **Score:** 8 (Tier 2)
- **Owner:** Deployment
- **Trigger:** Verification fails after the customer applies DNS.
- **Mitigation:** Clear DNS instructions with checks; verification retries; default Vercel domain if customer DNS times out at 24 h; status updates to the customer.
- **Residual:** 4 (Tier 3)

### R-013 — Stripe webhook drop

- **Category:** Billing
- **Likelihood:** 2
- **Impact:** 4
- **Score:** 8 (Tier 2)
- **Owner:** Billing
- **Trigger:** Subscription state desync with Stripe; webhook delivery backlog grows.
- **Mitigation:** Replay tooling; daily reconciliation; idempotency keys; observability for delivery; manual fix runbook.
- **Residual:** 4 (Tier 3)

### R-014 — API abuse

- **Category:** Security / Cost
- **Likelihood:** 3
- **Impact:** 3
- **Score:** 9 (Tier 2)
- **Owner:** API
- **Trigger:** Single IP or API key generating excessive requests; LLM cost per key climbs.
- **Mitigation:** Per-IP, per-tenant, per-API-key rate limits; progressive backoff; lockout; human review.
- **Residual:** 4 (Tier 3)

### R-015 — Subprocess / RCE in generated code

- **Category:** Security
- **Likelihood:** 2
- **Impact:** 5
- **Score:** 10 (Tier 2)
- **Owner:** Generation
- **Trigger:** Generated code contains `eval`, `Function`, `child_process`, raw `dangerouslySetInnerHTML`.
- **Mitigation:** Static scanner for dangerous patterns; safe defaults; review checklist; CSP.
- **Residual:** 4 (Tier 3)

### R-016 — Privacy / PII leak via LLM

- **Category:** Privacy
- **Likelihood:** 3
- **Impact:** 4
- **Score:** 12 (Tier 2)
- **Owner:** Privacy + Agents
- **Trigger:** A prompt is observed to contain unredacted PII; an LLM provider logs a customer email.
- **Mitigation:** Redact known PII patterns before LLM calls; configure provider data-retention to no-train/no-retain; data processing addendum; periodic audit.
- **Residual:** 6 (Tier 3)

### R-017 — Compliance gap (SOC 2)

- **Category:** Compliance
- **Likelihood:** 3
- **Impact:** 4
- **Score:** 12 (Tier 2)
- **Owner:** Compliance
- **Trigger:** Audit findings; control gaps revealed in incident retros.
- **Mitigation:** Continuous compliance evidence collection (V2); policy library; quarterly internal audit; pen-test annually.
- **Residual:** 6 (Tier 3)

### R-018 — Key talent loss

- **Category:** Business
- **Likelihood:** 3
- **Impact:** 4
- **Score:** 12 (Tier 2)
- **Owner:** Leadership
- **Trigger:** Departure of a key engineer or product owner.
- **Mitigation:** Documentation discipline; pair sessions; bus factor > 1 in every area; runbooks; recorded design decisions.
- **Residual:** 6 (Tier 3)

### R-019 — Customer concentration

- **Category:** Business
- **Likelihood:** 3
- **Impact:** 3
- **Score:** 9 (Tier 2)
- **Owner:** Leadership
- **Trigger:** Top 5 customers exceed 50% of revenue.
- **Mitigation:** Diversify pipeline; land-and-expand motion; pricing tiers; case studies from smaller customers.
- **Residual:** 6 (Tier 3)

### R-020 — Competitor with deeper pockets

- **Category:** Market
- **Likelihood:** 3
- **Impact:** 3
- **Score:** 9 (Tier 2)
- **Owner:** Product
- **Trigger:** New entrant with similar positioning; major platform announcement.
- **Mitigation:** Defensible moat: depth of generation, depth of SEO/llms.txt, cost leadership, integration breadth, customer love.
- **Residual:** 6 (Tier 3)

### R-021 — Schema migration breaks production

- **Category:** Operations
- **Likelihood:** 2
- **Impact:** 4
- **Score:** 8 (Tier 2)
- **Owner:** Platform
- **Trigger:** Migration error in CI; failed deploy; data drift.
- **Mitigation:** Forward-only migrations with online techniques; deploy migrations before app; smoke tests after; rollback drill.
- **Residual:** 4 (Tier 3)

### R-022 — Observability cost blow-up

- **Category:** Cost
- **Likelihood:** 2
- **Impact:** 3
- **Score:** 6 (Tier 3)
- **Owner:** Platform
- **Trigger:** Tempo/Loki/Mimir bill increases 30% over 30 days.
- **Mitigation:** Sampling; cardinality limits; retention tuning; dashboards-as-code reviews.
- **Residual:** 4 (Tier 3)

### R-023 — Botched handoff between agents

- **Category:** Quality
- **Likelihood:** 3
- **Impact:** 3
- **Score:** 9 (Tier 2)
- **Owner:** Agents
- **Trigger:** Quality review reveals gaps between capture and analysis; between analysis and generation.
- **Mitigation:** Schema-versioned contracts; contract tests; clear error reporting on mismatch; rejected artifacts halt the pipeline.
- **Residual:** 4 (Tier 3)

### R-024 — Customer data subject access request (DSAR) missed

- **Category:** Compliance
- **Likelihood:** 2
- **Impact:** 3
- **Score:** 6 (Tier 3)
- **Owner:** Compliance
- **Trigger:** A DSAR is open > 7 days.
- **Mitigation:** SLA-tracked queue; automated export; weekly review.
- **Residual:** 4 (Tier 3)

### R-025 — Founder dependency

- **Category:** Business
- **Likelihood:** 3
- **Impact:** 3
- **Score:** 9 (Tier 2)
- **Owner:** Leadership
- **Trigger:** Founder is single point of contact for a key decision; key relationships are founder-led.
- **Mitigation:** Document decisions; distribute customer relationships; cross-train in architecture.
- **Residual:** 6 (Tier 3)

---

## Top Risks (Tier 1)

- R-002 — Generation quality regression
- R-008 — LLM provider outage

These receive continuous attention: weekly review, dedicated dashboards, on-call drills, and runbook updates.

---

## Mitigation Patterns

The platform uses several repeated patterns:

- **Cost envelope + buffer:** every LLM-using agent has a hard cap, and the per-job total has a buffer.
- **Failover and circuit breakers:** external providers always have a fallback.
- **Idempotency everywhere:** every external action is idempotent under a deterministic key.
- **Compensation registry:** every multi-step workflow can be rolled back step by step.
- **Defensive defaults:** security and privacy defaults are the safe ones.
- **Observability as backstop:** every action is traceable.

---

## Risk Review Cadence

- **Per PR:** security review checklist for security-relevant code; agent review for agent code; data review for data code.
- **Weekly:** engineering risk review (Tiers 1 and 2).
- **Monthly:** full risk review with leadership.
- **Quarterly:** external pen-test; threat model review.
- **Annually:** full risk register refresh.

---

## Triggers and Response

| Trigger | Initial Response |
|---------|------------------|
| New critical CVE in a dependency | Patch within 7 days; emergency hotfix if exploited risk is high. |
| Provider outage > 15 min | Page on-call; failover; status page update. |
| Cost anomaly > 20% | Throttle affected tenants; investigate. |
| Quality gate failure spike | Halt rollout; revert; analyze. |
| Cross-tenant data smell | Page on-call; lock affected tenant; forensic review. |

---

## Assumptions

- The risk register reflects reasonable foresight, not exhaustive prediction.
- Mitigations are resourced; under-resourced mitigations are not mitigations.
- Risks change with the platform; the register is alive.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Tier 1 risks get a dashboard | Continuous visibility. |
| Every risk has an owner | Accountability. |
| Risks are reviewed at milestone gates | Tied to delivery. |
| Mitigations are tested | Tested mitigations are real mitigations. |

---

## Open Questions

- Should we run a public bug bounty at V2?
- Should we add a separate risk register for third-party libraries?

---

## Future Enhancements

- Auto-detected risks from observability anomalies.
- Risk-adjusted roadmap prioritization.

---

## Cross-References

- Security → `17-security-model.md`
- Observability → `18-observability.md`
- Roadmap → `23-milestone-roadmap.md`
- Cost → `28-cost-model.md`
