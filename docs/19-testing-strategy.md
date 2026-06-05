# 19 — Testing Strategy

> How we ensure the platform is correct, fast, safe, and recoverable at every layer.

---

## Purpose

This document defines the testing strategy for Vibe. It enumerates the test layers, the tools used, the responsibilities of contributors, the coverage targets, and the gates that prevent regressions from reaching production.

The platform's value depends on trust: customers must trust that the modernization we deliver is correct, that their data is handled safely, and that the platform recovers from failure. Testing is the evidence for that trust.

---

## Scope

In scope:

- Test taxonomy (unit, integration, contract, end-to-end, agent, performance, security, observability)
- Tooling per layer
- Coverage targets and reporting
- Determinism rules
- Test data and fixtures
- CI gates
- Release gates

Out of scope:

- Customer-side UAT (handled by Support)
- Operational runbooks (covered in `18-observability.md`)

---

## Guiding Principles

1. **Test behavior, not implementation.** Tests should survive refactors.
2. **Determinism first.** A flaky test is worse than a missing test.
3. **Pyramid discipline.** Many fast unit tests, fewer integration tests, a small number of high-value end-to-end tests.
4. **Cost-aware.** Don't burn real LLM or Vercel budgets in CI.
5. **Reproduce → fix → regress.** Every bug fixes a test first.
6. **No production debug.** If a bug escapes to production, a test is required for the fix.

---

## Test Pyramid

| Layer | Speed | Volume | Purpose |
|-------|-------|--------|---------|
| Unit | < 50 ms each | Many | Functions, classes, components in isolation |
| Integration | < 5 s each | Many | Cross-module behavior with real adapters (DB, queue, blob) |
| Contract | < 10 s each | Many | Service contracts (HTTP, message, schema) |
| End-to-end (workflow) | < 15 min each | Few | Full job lifecycle |
| Agent (workflow sub-tree) | < 2 min each | Moderate | Agent behavior with mocked LLM |
| Performance | < 1 h suite | Quarterly | Throughput and latency budgets |
| Security | < 1 h suite | Daily | SAST, DAST, dependency, secret |
| Observability | < 10 min suite | Daily | Telemetry shape and routing |
| Smoke (production) | < 10 min | Continuous | Live production health |

---

## Unit Testing

### Tools

- Python: `pytest`, `pytest-asyncio`, `pytest-randomly`, `freezegun`, `httpx`
- TypeScript: `vitest`, `@testing-library/react`, `happy-dom`

### Conventions

- One test file per module: `module.py` → `tests/test_module.py`.
- Test names describe behavior: `def test_router_rejects_urls_to_private_ips` is acceptable; `def test_router_1` is not.
- Use AAA structure (Arrange-Act-Assert).
- Mock at the boundary; do not mock internal modules.
- Time, randomness, and I/O are all injected or stubbed.
- Test fixtures are local to the file unless shared.

### Coverage Target

- 85% line coverage per package; 80% branch coverage.
- Coverage diff is reported in PR; new code below 85% is not merged.

---

## Integration Testing

### Tools

- Python: `pytest` + `testcontainers` (Postgres, Redis, LocalStack)
- TypeScript: `vitest` + `docker-compose` for services
- Migrations: run as part of the suite setup

### What We Test

- Repository writes against a real Postgres
- Redis queue interactions
- S3-compatible storage (LocalStack) for artifact writes
- HTTP routes with a real `TestClient` (FastAPI)
- Workflow determinism via Temporal's testing sandbox

### Conventions

- Tests are isolated: each test gets a fresh database (truncate or transactional rollback).
- No network egress to real third parties.
- Test data is deterministic.

### Coverage Target

- All repository methods have an integration test.
- All HTTP routes have happy-path + at least one failure-path integration test.
- All workflow activities have a state-mocked integration test.

---

## Contract Testing

### Tooling

- **Pact** for HTTP consumer-driven contracts between internal services and the API.
- **Schema tests** (Pydantic v2) for any JSON crossing a boundary (Temporal activity I/O, LLM structured outputs).
- **OpenAPI snapshot tests** to ensure the public API spec is intentional.

### What We Test

- Public REST API consumers receive the documented shape.
- Internal event payloads match the documented schema.
- LLM tool contracts (function-call schemas) are valid.

---

## End-to-End (Workflow) Testing

### Purpose

Verify that the entire job lifecycle works. The test runs a real job against a sandboxed environment.

### Setup

- Uses the `e2e-sandbox` Vercel team and a sandbox GitHub org.
- Uses sandbox Stripe keys.
- Uses sandbox LLM endpoints (Cheap + Small model) with a recorded response for determinism.
- Uses a deterministic seed site for the input.

### Cases (MVP)

- Happy path: small marketing site → 100% deliverable.
- Large site: 80-page site, with capture throttling.
- Bot-walled site: captures only the partial manifest; reports partial result.
- Broken LLM response: analysis engine retries, succeeds on second attempt.
- Self-repair: generation fails once, repairs, succeeds.

### Cadence

- Runs on PR against `main` for any change to orchestration, agents, or generation.
- Runs nightly.
- Runs on every release candidate.

---

## Agent Testing

Agents combine deterministic logic and LLM-driven decisions. Testing requires special care.

### Layers

- **Prompt unit tests**: verify prompt template renders correctly, contains required sections, and fits the model context.
- **Tool unit tests**: each tool has a unit test against a stub or LocalStack.
- **Decision unit tests**: with a recorded LLM response, the agent's decision function is tested.
- **Cost envelope tests**: a known fixture produces a known cost band; regressions break the test.

### Tools

- `pytest` with the `llm_recorded` fixture; LLM responses are loaded from `tests/fixtures/llm/<agent>/<case>.json`.
- A/V (`vibe test agents`) CLI subcommand runs the agent suite locally.

### LLM in CI

- LLM calls are not made to real providers in CI. All responses are recorded fixtures.
- A "regenerate fixtures" command requires an explicit `--update` flag and runs locally with a developer's key.

### Coverage Target

- Each agent has a test per tool, per decision branch, per documented failure mode.
- Cost envelope is tested.

---

## Performance Testing

### Approach

- Load tests run on a staging environment with synthetic but representative sites.
- k6 scripts under `infra/perf/`; the harness drives the full job lifecycle against mocked external services.

### Targets (MVP)

- API p95 ≤ 300 ms (already in SLO).
- Capture pool handles 50 concurrent jobs.
- Orchestrator sustains 30 jobs/hour.
- Generation build p95 ≤ 8 min.

### Cadence

- Nightly on main.
- Required for any change touching capture, generation, or orchestrator concurrency.

---

## Security Testing

### Layers

- **SAST**: Semgrep + Bandit (Python), Semgrep (TS), in CI.
- **Dependency**: Snyk in CI; failing the build on Critical.
- **Secret scan**: Gitleaks pre-commit + CI.
- **DAST**: scheduled run against staging with OWASP ZAP.
- **SSRF fuzzing**: dedicated harness that probes the capture engine with adversarial URLs.
- **Tenant isolation fuzzing**: tests assert that no cross-tenant access path exists.

### Cadence

- SAST, dependency, secret: every PR.
- DAST: nightly.
- SSRF and isolation fuzzing: weekly + on PR for security-relevant code.

---

## Observability Testing

Tests assert that telemetry flows correctly.

- Synthetic request → assert trace, metrics, log arrive in the backends with expected attributes.
- "Schema conformance" tests parse the OTel output and assert required fields exist.
- Run nightly and on PR for any change to instrumentation.

---

## Smoke (Production)

- Continuous synthetic checks hitting the public API every 30 seconds.
- A synthetic full job runs once per hour against a known source.
- All results stream to the SRE channel and the smoke dashboard.

---

## Data and Fixtures

- `tests/fixtures/` is the single source for shared fixtures.
- Sites used for capture tests live as static HTML/JS bundles in `tests/fixtures/sites/<name>/`.
- LLM responses are recorded in `tests/fixtures/llm/<agent>/<case>.json` and are immutable once merged.
- A re-record run is explicit and requires sign-off from the agent owner.

---

## Determinism Rules

- All random sources are seeded.
- All time sources are injected.
- All network calls are stubbed unless the test is a network test.
- Tests must pass repeatedly with the same result.
- Flaky tests are quarantined for ≤ 5 days; a ticket is opened; an owner assigned.

---

## CI Gates

| Check | Gate |
|-------|------|
| Lint | Required to pass |
| Typecheck | Required to pass |
| Unit tests | Required to pass |
| Integration tests | Required to pass |
| Coverage | ≥ 80% line on new code; coverage cannot drop on the package |
| Secret scan | Required to pass |
| SAST | No Critical |
| Dependency | No Critical vulnerabilities |
| OpenAPI snapshot | Required to pass if API changed |
| Workflow tests | Required if orchestrator or any agent changed |
| Observability tests | Required if instrumentation changed |

---

## Release Gates

| Check | Gate |
|-------|------|
| All CI checks green on `main` | Yes |
| Workflow tests on a release candidate branch | Yes |
| Performance tests within 10% of last release | Yes |
| DAST clean | Yes |
| Threat model updated | Yes if attack surface changed |
| Migration dry-run | Yes |
| Rollback drill | Yes (last release's rollback tested within 30 days) |

---

## Test Environments

| Environment | Purpose | Lifecycle |
|-------------|---------|----------|
| `local` | Developer | Spun up via `make bootstrap` |
| `ci` | Pull requests | Ephemeral |
| `pr-preview` | Long-running PR features | Auto-archived after 7 days |
| `staging` | Mirrors prod | Always warm |
| `prod` | Production | N/A |

Each environment is independent; secrets are scoped; data is segregated.

---

## Test Data

- A `vibe_test_*` prefix identifies test records.
- Test data is purged nightly from staging.
- A dedicated tenant in production (`vibe-qa`) hosts production-realistic tests; never uses real customer data.

---

## Assumptions

- The team has access to a sandbox GitHub org, a sandbox Vercel team, and sandbox Stripe and LLM accounts.
- CI can run testcontainers and LocalStack.
- The performance environment is sized near production.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Pyramid with mandatory E2E | Catches integration risk no unit test sees. |
| Recorded LLM fixtures in CI | Determinism; cost; no leakage of secrets. |
| Coverage threshold gated | Prevents drift. |
| Observability tests are first-class | The platform's debugging capability must not regress. |
| Workflow tests required for agent changes | Agents are the core value; they cannot regress silently. |

---

## Open Questions

- Should we adopt mutation testing at V2 to validate the test suite's strength?
- Should we record LLM traces in a dedicated eval set for prompt regressions?

---

## Future Enhancements

- Property-based testing for URL normalizer and HTML parser.
- Fuzzing in production for input parsers.
- Synthetic job marketplace (an internal data set) at V2.
- Continuous evaluation of agents using offline traces (`25-risk-analysis.md`).

---

## Cross-References

- Coding standards → `21-coding-standards.md`
- Observability → `18-observability.md`
- Security model → `17-security-model.md`
- Milestones → `23-milestone-roadmap.md`
