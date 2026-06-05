# 06 — Non-Functional Requirements

> The system's qualities, expressed as measurable promises rather than features.

---

## Purpose

This document defines the non-functional requirements (NFRs) of the platform: the qualities the system must possess regardless of which features are present. Each NFR is testable, quantified, and tied to a measurement strategy.

NFRs constrain implementation choices. Where a feature decision could violate an NFR, the NFR wins.

---

## Scope

In scope:

- Performance, scalability, reliability, availability
- Security, privacy, compliance
- Maintainability, testability, deployability
- Cost
- Usability, accessibility (for human-facing surfaces)
- Internationalization
- Data retention and durability

Out of scope:

- Functional behavior (`05-functional-requirements.md`)
- Implementation specifics (`07-technical-specification.md`)
- Specific test cases (`19-testing-strategy.md`)

---

## NFR Format

Each NFR is expressed as:

> **NFR-<area>-<n>** — <statement>
>
> Target: <measurable threshold>
> Measurement: <how we verify>
> Milestone: <when it becomes binding>

---

## Performance

### NFR-PERF-001 — Intake API response time

> The intake API shall return a response within 300 ms p95 and 800 ms p99 under normal load.

- Target: p95 ≤ 300 ms, p99 ≤ 800 ms
- Measurement: OpenTelemetry latency histograms aggregated over 5-minute windows
- Milestone: M1

### NFR-PERF-002 — Free preview latency

> The free preview shall be returned within 30 seconds p95.

- Target: p95 ≤ 30 s
- Measurement: synthetic monitoring + real-user metrics
- Milestone: M1

### NFR-PERF-003 — End-to-end job latency (MVP)

> A standard 10-page job shall complete from intake to delivery within 45 minutes p95.

- Target: p95 ≤ 45 min, p99 ≤ 90 min
- Measurement: job-level histograms
- Milestone: M5

### NFR-PERF-004 — End-to-end job latency (V3)

> A standard 10-page job shall complete within 8 minutes p95.

- Target: p95 ≤ 8 min
- Measurement: job-level histograms
- Milestone: M6

### NFR-PERF-005 — Delivered-site Lighthouse Performance

> The median delivered site shall score ≥ 90 on Lighthouse Performance (mobile).

- Target: median ≥ 90, p10 ≥ 75
- Measurement: per-job post-deploy Lighthouse run
- Milestone: M3

### NFR-PERF-006 — Delivered-site Largest Contentful Paint

> The median delivered site shall have LCP ≤ 2.0 s on a 4G simulated connection.

- Target: median ≤ 2.0 s
- Measurement: Lighthouse + Core Web Vitals from real users (where available)
- Milestone: M3

### NFR-PERF-007 — Delivered-site Cumulative Layout Shift

> The delivered site shall have CLS ≤ 0.1.

- Target: median ≤ 0.1, p95 ≤ 0.25
- Measurement: Lighthouse
- Milestone: M3

### NFR-PERF-008 — Delivered-site Interaction to Next Paint

> The delivered site shall have INP ≤ 200 ms p95.

- Target: p95 ≤ 200 ms
- Measurement: real-user metrics where instrumented
- Milestone: M4

### NFR-PERF-009 — Capture throughput

> A single capture worker shall handle ≥ 1 page per 6 seconds on a typical content site.

- Target: ≥ 10 pages/min/worker
- Measurement: per-worker metrics
- Milestone: M2

### NFR-PERF-010 — Generation throughput

> A single generation worker shall produce a 10-page workspace within 8 minutes p95.

- Target: p95 ≤ 8 min
- Measurement: per-worker metrics
- Milestone: M3

---

## Scalability

### NFR-SCALE-001 — Concurrent jobs (MVP)

> The platform shall support 50 concurrent jobs without queueing delays beyond 1 minute.

- Target: ≥ 50 concurrent jobs
- Measurement: load test
- Milestone: M5

### NFR-SCALE-002 — Concurrent jobs (V2)

> The platform shall support 500 concurrent jobs.

- Target: ≥ 500
- Measurement: load test
- Milestone: M5 → M6 ramp

### NFR-SCALE-003 — Concurrent jobs (V3)

> The platform shall support 5,000 concurrent jobs.

- Target: ≥ 5,000
- Measurement: load test
- Milestone: M6

### NFR-SCALE-004 — Horizontal scaling

> Every worker pool shall scale horizontally without code changes.

- Target: linear scaling up to 4× nominal capacity
- Measurement: synthetic load
- Milestone: M1

### NFR-SCALE-005 — Database connection ceiling

> The database connection count shall remain below 60% of the configured max under nominal load.

- Target: ≤ 60% utilization
- Measurement: RDS metrics
- Milestone: M1

### NFR-SCALE-006 — Object storage

> Object storage shall be sized to support the projected volume for the next 12 months without manual intervention.

- Target: native S3 scalability
- Measurement: design review
- Milestone: M1

---

## Availability & Reliability

### NFR-AVAIL-001 — Public API availability

> The public API shall have ≥ 99.5% monthly availability in MVP and ≥ 99.95% in V3.

- Target (MVP): 99.5% (≤ 3.6 h/month downtime)
- Target (V3): 99.95% (≤ 22 min/month)
- Measurement: external uptime probe (1-minute granularity)
- Milestone: M5 → M6

### NFR-AVAIL-002 — Customer dashboard availability

> The customer dashboard shall have ≥ 99.5% availability.

- Target: 99.5%
- Measurement: external probe
- Milestone: M5

### NFR-AVAIL-003 — Job success rate

> ≥ 95% of submitted jobs (excluding intake rejections) shall complete successfully in MVP, ≥ 99.5% in V3.

- Target (MVP): 95%, (V3): 99.5%
- Measurement: job outcome metrics, daily rollup
- Milestone: M5 → M6

### NFR-AVAIL-004 — Mean time to recovery

> Operator MTTR for a failed job shall be ≤ 4 hours (MVP) and ≤ 5 minutes (V3, automated).

- Target: as above
- Measurement: incident records
- Milestone: M5 → M6

### NFR-AVAIL-005 — Workflow durability

> No in-flight job shall be lost on a worker crash, deploy, or AZ failure.

- Target: 0 lost jobs per quarter
- Measurement: workflow audit reconciliation
- Milestone: M1

### NFR-AVAIL-006 — Idempotent retries

> Any agent or external integration call shall be safe to retry without producing duplicate side effects.

- Target: 100% of side-effecting calls idempotent
- Measurement: code review + chaos testing
- Milestone: M5

### NFR-AVAIL-007 — Backup RPO

> Database backups shall provide RPO ≤ 5 minutes.

- Target: RPO ≤ 5 min
- Measurement: RDS PITR configuration
- Milestone: M1

### NFR-AVAIL-008 — Backup RTO

> Database restore shall have RTO ≤ 1 hour.

- Target: RTO ≤ 1 h
- Measurement: quarterly restore drill
- Milestone: M5

---

## Security

### NFR-SEC-001 — Transport encryption

> All public network traffic shall use TLS 1.3 where supported, never below TLS 1.2.

- Target: 100% TLS, A+ on SSLLabs
- Measurement: SSLLabs scan, weekly
- Milestone: M1

### NFR-SEC-002 — At-rest encryption

> All persistent data (Postgres, S3, EBS) shall be encrypted with KMS-managed keys.

- Target: 100% encrypted
- Measurement: AWS Config rule
- Milestone: M1

### NFR-SEC-003 — Tenant isolation

> A tenant shall not be able to read or affect another tenant's data through any API or internal path.

- Target: 0 cross-tenant incidents
- Measurement: penetration test, automated authorization fuzzing
- Milestone: M5

### NFR-SEC-004 — Secrets handling

> No secret shall be written to logs, traces, source repositories, or build artifacts.

- Target: 0 occurrences
- Measurement: pre-commit secret scan, post-deploy log scan
- Milestone: M1

### NFR-SEC-005 — Vulnerability remediation SLA

> Critical CVEs in production dependencies shall be patched within 7 days; high within 30 days.

- Target: as above
- Measurement: SBOM + vulnerability scanner
- Milestone: M5

### NFR-SEC-006 — Authentication strength

> Operator console access shall require MFA. Customer access shall use passwordless email magic links.

- Target: 100% MFA enforcement on operator
- Measurement: identity provider audit
- Milestone: M5

### NFR-SEC-007 — Audit logging

> Every authentication event, authorization decision, admin action, and tenant data export shall be audit-logged for 365 days.

- Target: 100% coverage; 365-day retention
- Measurement: log audit
- Milestone: M5

### NFR-SEC-008 — Customer data egress

> Customer source content and generated code shall not leave the platform's controlled environment except via documented integrations.

- Target: 100% adherence
- Measurement: egress monitoring, design review
- Milestone: M1

### NFR-SEC-009 — DDoS resilience

> The platform shall sustain 10× nominal request rate for 10 minutes without compromising availability.

- Target: as above
- Measurement: load test
- Milestone: M5

---

## Privacy & Compliance

### NFR-PRIV-001 — Right to deletion

> The platform shall support customer-initiated deletion of personal data within 30 days of request.

- Target: ≤ 30 days
- Measurement: support ticket SLA, automated deletion logs
- Milestone: M5

### NFR-PRIV-002 — Data minimization

> The platform shall collect only data necessary for the service and document each field's purpose.

- Target: 100% fields documented
- Measurement: privacy review
- Milestone: M5

### NFR-PRIV-003 — Regional data residency

> Customer data shall be stored in the customer's selected region. Default is US.

- Target: 100% adherence
- Measurement: storage configuration
- Milestone: M6 (EU residency)

### NFR-PRIV-004 — GDPR compliance

> The platform shall comply with GDPR for EU customers, including DPA, SCCs, and processing records.

- Target: signed DPA, public privacy notice
- Measurement: legal review
- Milestone: M5

### NFR-PRIV-005 — SOC2

> The platform shall achieve SOC2 Type I by V2 and Type II by V3.

- Target: as above
- Measurement: auditor report
- Milestone: M5 (Type I) → M6 (Type II)

---

## Maintainability

### NFR-MAINT-001 — Test coverage

> Unit-test line coverage shall be ≥ 80% for application code, ≥ 90% for shared libraries.

- Target: as above
- Measurement: coverage report in CI
- Milestone: M1

### NFR-MAINT-002 — Build time

> The full repository build shall complete in ≤ 8 minutes on CI under cold cache, ≤ 4 minutes warm.

- Target: as above
- Measurement: CI duration
- Milestone: M1

### NFR-MAINT-003 — Lint and format

> All code shall pass lint and format checks in CI. Pre-commit hooks shall enforce locally.

- Target: 100% adherence
- Measurement: CI status
- Milestone: M1

### NFR-MAINT-004 — Static typing

> All Python code shall pass `mypy --strict`. All TypeScript code shall pass `tsc --strict`.

- Target: 100% strict
- Measurement: CI
- Milestone: M1

### NFR-MAINT-005 — Documentation coverage

> Every public API, every agent contract, every database table shall be documented in the `/docs/` tree.

- Target: 100% coverage
- Measurement: docs lint
- Milestone: M5

### NFR-MAINT-006 — Time-to-first-PR

> A new contributor shall be able to clone, build, test, and open a PR within 1 day.

- Target: ≤ 1 day
- Measurement: contributor survey
- Milestone: M5

### NFR-MAINT-007 — Dependency freshness

> No production dependency shall be more than 12 months behind its current major version without a documented exception.

- Target: ≤ 12 months
- Measurement: Renovate or Dependabot reports
- Milestone: M5

---

## Deployability

### NFR-DEPLOY-001 — Zero-downtime deploys

> All production deployments shall complete with zero customer-visible downtime.

- Target: 0 user-impacting deploys
- Measurement: deploy logs + uptime probe
- Milestone: M1

### NFR-DEPLOY-002 — Deploy frequency

> The team shall be able to deploy to production at least daily.

- Target: ≥ 1/day capability
- Measurement: deployment metrics
- Milestone: M1

### NFR-DEPLOY-003 — Rollback time

> Production rollback shall complete within 5 minutes.

- Target: ≤ 5 min
- Measurement: rollback drill
- Milestone: M5

### NFR-DEPLOY-004 — Environment parity

> Staging shall use the same image, configuration shape, and database engine version as production.

- Target: 100% parity
- Measurement: environment audit
- Milestone: M1

---

## Cost

### NFR-COST-001 — COGS per job (MVP)

> Cost of goods sold per job shall be ≤ $5 (median) and ≤ $10 (p95).

- Target: as above
- Measurement: per-job cost rollup
- Milestone: M5

### NFR-COST-002 — COGS per job (V3)

> COGS per job shall be ≤ $0.75 (median).

- Target: ≤ $0.75
- Measurement: per-job cost rollup
- Milestone: M6

### NFR-COST-003 — Fixed monthly infrastructure (MVP)

> Fixed infrastructure costs (idle baseline) shall be ≤ $2,000/month at MVP volumes.

- Target: ≤ $2,000/month
- Measurement: AWS Cost Explorer
- Milestone: M5

### NFR-COST-004 — LLM cost share

> LLM spend shall be ≤ 40% of COGS by V3.

- Target: ≤ 40%
- Measurement: cost attribution
- Milestone: M6

---

## Usability

### NFR-USE-001 — Time to value (preview)

> A first-time user shall be able to see a free preview within 60 seconds of landing on the homepage.

- Target: median ≤ 60 s
- Measurement: funnel analytics
- Milestone: M5

### NFR-USE-002 — Time to value (delivery)

> A paying customer shall receive a delivered site within 45 minutes (MVP).

- Target: median ≤ 45 min
- Measurement: funnel analytics
- Milestone: M5

### NFR-USE-003 — Dashboard responsiveness

> Dashboard interactions shall feel instantaneous (p95 ≤ 200 ms for client-side interactions, ≤ 1 s for data fetches).

- Target: as above
- Measurement: real-user metrics
- Milestone: M5

### NFR-USE-004 — Error message clarity

> Every customer-facing error shall include a plain-language description and a suggested next action.

- Target: 100% coverage
- Measurement: design review
- Milestone: M5

---

## Accessibility (Customer-Facing Surfaces)

### NFR-A11Y-001 — WCAG conformance

> All customer-facing pages shall conform to WCAG 2.2 Level AA.

- Target: 0 critical or serious axe-core violations
- Measurement: automated + manual audit per release
- Milestone: M5

### NFR-A11Y-002 — Keyboard navigation

> Every interactive element shall be reachable and operable by keyboard.

- Target: 100% coverage
- Measurement: manual test
- Milestone: M5

### NFR-A11Y-003 — Screen reader support

> The dashboard and intake flow shall be fully navigable using a screen reader.

- Target: manual confirmation with NVDA and VoiceOver
- Measurement: manual test
- Milestone: M5

---

## Internationalization

### NFR-I18N-001 — UI localization

> The customer UI shall be localizable; English is the only locale for MVP, Spanish and German added by V2.

- Target: 3 locales by V2
- Measurement: locale inventory
- Milestone: M5 → M6

### NFR-I18N-002 — Source-site language detection

> The platform shall detect the source site's primary language and preserve it in generation.

- Target: 100% detection for top 20 languages
- Measurement: integration test
- Milestone: M3

### NFR-I18N-003 — RTL support

> Generated sites shall correctly render for right-to-left languages.

- Target: 100% RTL fidelity for detected RTL sites
- Measurement: visual diff
- Milestone: M6

---

## Data Retention & Durability

### NFR-DATA-001 — Artifact retention (paid jobs)

> Capture artifacts and generated workspaces shall be retained for ≥ 90 days after job completion.

- Target: ≥ 90 days
- Measurement: lifecycle policy
- Milestone: M1

### NFR-DATA-002 — Artifact retention (free preview)

> Free-preview artifacts shall be retained for 7 days, then deleted automatically.

- Target: 7 days
- Measurement: lifecycle policy
- Milestone: M1

### NFR-DATA-003 — Object storage durability

> Object storage shall provide ≥ 11 nines durability.

- Target: ≥ 99.999999999%
- Measurement: provider SLA (S3 Standard)
- Milestone: M1

### NFR-DATA-004 — Audit log retention

> Security audit logs shall be retained for ≥ 365 days.

- Target: ≥ 365 days
- Measurement: log retention configuration
- Milestone: M1

### NFR-DATA-005 — Customer data export

> Customers shall be able to export all their data within 24 hours of request.

- Target: ≤ 24 h
- Measurement: support SLA
- Milestone: M5

---

## Constraints Cross-Reference

| Constraint Family | Drives | Conflicts With |
|-------------------|--------|----------------|
| Performance | Architecture choices, caching, CDN use | Cost |
| Cost | LLM model choice, retention windows, idle baseline | Performance, latency |
| Security | Encryption, isolation, access patterns | Maintainability (more friction) |
| Compliance | Data residency, retention, audit | Cost, flexibility |
| Maintainability | Test coverage, lint enforcement | Velocity (short term) |

These tensions are explicit. When a feature decision triggers a conflict, the resolution is documented as an ADR.

---

## Assumptions

- AWS regional SLAs are accepted as the baseline reliability.
- LLM provider SLAs are 99% or better; the platform's higher-tier SLAs require provider failover.
- Customers tolerate slower MVP latency (≤ 45 min) in exchange for low price.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Express NFRs as testable thresholds | Avoids "good enough" debates; everything is verifiable. |
| Measure NFRs continuously, not only at release | SLO burn rate alerts catch regressions. |
| Set tighter V3 targets distinct from MVP | Avoids premature optimization while documenting commitment. |
| Document cross-NFR tensions explicitly | Forces architectural decisions to engage tradeoffs. |

---

## Open Questions

- What is the right SLO for the public API at V3 — 99.95% or 99.99%?
- Should the platform offer a paid tier with enhanced NFRs (e.g., 1-hour SLA)?
- Should NFR violations be public on a status page?

---

## Future Enhancements

- Per-tenant NFR SLAs sold as a feature.
- Real-time NFR-burn dashboards exposed to customers.
- Automatic feature degradation when NFR budgets are exhausted.

---

## Cross-References

- Functional → `05-functional-requirements.md`
- Test plan → `19-testing-strategy.md`
- Observability → `18-observability.md`
- Cost model → `28-cost-model.md`
- Security model → `17-security-model.md`
