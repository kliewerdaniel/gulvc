# ADR-001 — Monorepo with pnpm + uv workspaces

- **Status:** Accepted
- **Date:** 2026-06-01
- **Deciders:** Engineering leads
- **Supersedes:** —
- **Superseded by:** —

---

## Context

Vibe is a multi-service platform spanning Python (agents, workers, orchestrator integrations) and TypeScript (API, web, integrations). Early decisions need to address:

- How do we keep the codebase navigable as services and packages grow?
- How do we share code (models, types, utilities) without copy-paste drift?
- How do we ensure consistent tooling, lint, and types across languages?
- How do we keep CI fast?
- How do new contributors get up to speed quickly?

The two main alternatives were: (1) a polyrepo with one repository per service, or (2) a monorepo with shared tooling.

---

## Decision

Adopt a **monorepo** at the repository root (`gouplevelvibecoding/`), with:

- **pnpm workspaces** for TypeScript packages and apps.
- **uv workspaces** for Python packages and apps.
- **Turborepo** as the build orchestrator for TypeScript.
- **A unified `Makefile`** for cross-language entry points.
- **Strict dependency direction** enforced by linters: `apps → services → packages → schemas`.
- **`CODEOWNERS`** defining ownership per area.
- **Schema-first development**: cross-language types originate in `schemas/`.

---

## Rationale

### Why monorepo

- **Atomic refactors.** Changing a shared model updates all consumers in one PR.
- **Unified versioning.** Internal packages move together; the release train is simpler.
- **Single source of truth.** Docs, schemas, infra, and code in one place.
- **Discoverability.** New contributors can see the whole system, not slices.
- **Tooling consistency.** Lint, format, and type rules apply across languages via shared hooks.

### Why pnpm + uv

- **Best-in-class per ecosystem.** pnpm is fast, deterministic, and content-addressable; uv is the fastest Python package manager and supports workspaces natively.
- **Language-specific.** We do not force Python into a JS-centric tool or vice versa.
- **Lockfile-driven determinism.** Both ecosystems ship lockfiles that are committed.

### Why Turborepo

- **Affected-only builds.** CI runs only what changed, plus downstream.
- **Remote cache.** Developer machines and CI share build outputs.
- **Pipeline as code.** `turbo.json` makes the build graph explicit and reviewable.

### Why a unified Makefile

- **One entry point.** New contributors learn one command (`make bootstrap`) regardless of language.
- **Discoverability.** `make help` is the manpage.

### Why strict dependency direction

- **Architectural drift is the silent killer.** Linters catch it at PR time.

### Why schema-first

- **One source of truth across languages.** Pydantic and TypeScript types are generated from the same JSON Schema.
- **No translation bugs.** When the schema changes, both languages update together.

---

## Consequences

### Positive

- A new contributor can clone, bootstrap, and ship in their first week.
- Cross-cutting refactors (e.g., tenant filter enforcement) are one PR.
- The build graph is visible, fast, and cached.
- Documentation and code live together, reducing drift.

### Negative

- **Larger clone.** Mitigated by sparse-checkout in V2.
- **CI complexity.** Mitigated by affected-only builds and remote cache.
- **Discipline required.** Monorepos degrade into "everything depends on everything" without strict rules. Mitigated by linters and code review.
- **Build orchestration can become a bottleneck** at scale. Mitigated by Turborepo's caching and a future option to swap in `nx` if needed.

### Neutral

- We accept that some Python services will look "different" from a typical Python monorepo. The trade-off is worth it for cross-language atomicity.

---

## Alternatives Considered

### Polyrepo

- **Pro:** independent deploys, independent versioning.
- **Con:** shared-code drift, cross-repo refactors, more CI pipelines, harder onboarding. **Rejected.**

### Monorepo with Nx

- **Pro:** strong graph, code generators, consistent task runners.
- **Con:** heavy for our scale; opinionated; weaker Python support. **Rejected for MVP**, considered for V2 if Turborepo scaling fails.

### Bazel

- **Pro:** extremely powerful, language-agnostic.
- **Con:** high learning curve, expensive setup, overkill at our size. **Rejected.**

### Single-language (Python-only)

- **Pro:** simpler tooling, one type system.
- **Con:** forced into Python for the dashboard; loses Vercel/Next.js benefits; weaker fit for the customer-facing surface. **Rejected.**

---

## Notes

- The monorepo contains roughly: 12 apps, 8 services, 30 packages, 5+ schemas, 200+ GitHub Actions jobs.
- We will revisit Turborepo scaling at the M5 milestone.
- A monorepo "health" report is generated weekly to detect dependency cycles and orphaned packages.
