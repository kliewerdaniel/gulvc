# 22 — Monorepo Structure

> The shape of the repository. Where things live, what they depend on, and how they are built.

---

## Purpose

This document defines the monorepo structure for Vibe. It specifies the workspace layout, the build orchestration, the dependency rules between packages, and the conventions for adding new apps and packages.

The monorepo is the platform's source of truth. Its structure is optimized for small teams, fast CI, and clear ownership.

---

## Scope

In scope:

- Workspace tooling and configuration
- Directory layout for apps, packages, services, and infrastructure
- Dependency rules
- Build and task orchestration
- Local development environment
- Versioning and publishing

Out of scope:

- Coding standards (`21-coding-standards.md`)
- Deployment internals (`27-production-deployment.md`)

---

## Goals

1. **One repository, one mental model.** All code, schemas, infra, and docs in one place.
2. **Fast feedback locally.** A single command brings the platform up.
3. **Fast feedback in CI.** Affected-only builds via Turborepo.
4. **Clear ownership.** Every directory has a `OWNERS` file.
5. **Strict boundaries.** Packages cannot import across domain boundaries.
6. **Reproducible builds.** Lockfiles committed; deterministic tooling.

---

## Tooling

| Concern | Tool |
|---------|------|
| TypeScript workspace | `pnpm` 9 with workspaces |
| Python workspace | `uv` 0.4 with workspace members |
| Build orchestration | `turborepo` (TS) + `make` (Python coordination) |
| Linting (TS) | `biome` |
| Linting (Python) | `ruff` |
| Formatting (TS) | `biome` |
| Formatting (Python) | `ruff` |
| Type checking (TS) | `tsc` |
| Type checking (Python) | `mypy` |
| Testing (TS) | `vitest` |
| Testing (Python) | `pytest` |
| Pre-commit | `pre-commit` |
| Versioning (internal) | `changesets` |
| Container build | `docker buildx` |
| IaC | Terraform 1.9 |

A unified `Makefile` provides a thin wrapper for cross-language workflows.

---

## Top-Level Layout

```
gouplevelvibecoding/
├─ apps/                       # Deployable services
├─ packages/                   # Shared libraries
├─ services/                   # Long-running background services (workers)
├─ infra/                      # Terraform, helm-like overlays, CI
├─ tools/                      # Internal CLI and codegen
├─ schemas/                    # Cross-language schema artifacts
├─ tests/                      # Cross-app fixtures and recorded data
├─ docs/                       # Project documentation
├─ scripts/                    # Ad-hoc repo scripts
├─ .github/                    # GitHub Actions workflows
├─ .changeset/                 # Changesets
├─ package.json                # pnpm workspace root
├─ pnpm-workspace.yaml
├─ pyproject.toml              # Python workspace root
├─ uv.lock
├─ turbo.json
├─ Makefile
├─ CODEOWNERS
├─ CONTRIBUTING.md
├─ LICENSE
└─ README.md
```

---

## Apps

`apps/` contains deployable services. Each app is independently deployable but shares libraries via `packages/`.

```
apps/
├─ api/                # FastAPI public API + admin
├─ web/                # Next.js dashboard (Vibe UI)
├─ capture-worker/     # Playwright-based capture
├─ analysis-worker/    # LLM + parser-based analysis
├─ generation-worker/  # Code generation
├─ deploy-worker/      # GitHub + Vercel orchestration
├─ delivery-worker/    # Webhook + report delivery
├─ billing-worker/     # Stripe-driven subscription state
├─ scheduler/          # Cron-like jobs (retention, etc.)
└─ notifier/           # Email + in-app notifications
```

Each app follows a common internal structure:

```
apps/<app>/
├─ src/
│  ├─ <package>/
│  ├─ entrypoints/
│  └─ migrations/      # if the app owns data
├─ tests/
├─ pyproject.toml      # or package.json
├─ Dockerfile
├─ README.md
└─ OWNERS
```

---

## Packages

`packages/` contains shared libraries.

```
packages/
├─ ts/                 # TypeScript libraries
│  ├─ contracts/       # API + webhook types
│  ├─ ui/              # Shared React components
│  ├─ sdk/             # Public client SDK
│  ├─ tokens/          # Design tokens
│  └─ tsconfig/        # Shared tsconfig presets
├─ py/                 # Python libraries
│  ├─ vibe_core/       # Core utilities (config, logging, time, types)
│  ├─ vibe_models/     # Pydantic models shared across services
│  ├─ vibe_db/         # SQLAlchemy base, repositories, migrations helpers
│  ├─ vibe_observability/  # OTel wrappers
│  ├─ vibe_auth/       # JWT, API key verification
│  ├─ vibe_billing/    # Stripe client + plans
│  ├─ vibe_integrations/   # GitHub, Vercel, LLM providers
│  └─ vibe_test/       # Shared pytest fixtures
```

Each Python package:

- Is a `uv` workspace member.
- Has a `pyproject.toml`.
- Exposes a single importable top-level module.
- Depends only on other packages in the workspace, never on apps.

Each TypeScript package:

- Is a `pnpm` workspace member.
- Has a `package.json` with `name` matching `@vibe/<name>`.
- Has its own `tsconfig.json` extending `packages/ts/tsconfig`.
- Depends only on other workspace packages, never on apps.

---

## Services

Long-running services that are not HTTP servers belong in `services/`. They may run as separate processes or be embedded in workers depending on load.

```
services/
├─ browser-pool/       # Playwright pool manager
├─ llm-router/         # Provider failover, cost accounting
├─ rate-limiter/       # Shared in-memory + Redis rate limiter
└─ feature-flags/      # Feature flag evaluation
```

---

## Infrastructure

`infra/` contains everything needed to deploy and operate the platform.

```
infra/
├─ terraform/
│  ├─ modules/
│  └─ envs/
│     ├─ staging/
│     └─ prod/
├─ grafana/
│  ├─ dashboards/
│  └─ alerts/
├─ perf/               # k6 scripts
├─ ci/                 # Shared CI scripts
└─ threat-model/       # Living STRIDE documents
```

---

## Schemas

`schemas/` holds cross-language schemas.

```
schemas/
├─ api/                # OpenAPI definitions
├─ events/             # Domain event schemas
├─ llm/                # LLM tool-call and structured-output schemas
├─ site-model/         # SiteModel v1 schema (JSON Schema)
└─ migration/          # Cross-app migration plans
```

Schemas are the source of truth. Models in `vibe_models` and TypeScript types in `@vibe/contracts` are generated from these definitions.

---

## Tests

```
tests/
├─ fixtures/
│  ├─ sites/           # Static test sites
│  ├─ llm/             # Recorded LLM responses
│  └─ git/             # Replayable git histories
├─ e2e/                # End-to-end workflow tests
├─ perf/               # Long-running performance scripts
└─ fuzz/               # Fuzzing harnesses
```

---

## Dependency Rules

### Allowed Directions

```
apps → services → packages → schemas
apps → packages
apps → schemas
```

### Forbidden

- Apps depending on other apps.
- Packages depending on apps.
- Packages depending on services.
- Services depending on apps.
- Anything depending on `apps/api/internal/*` (private).

These rules are enforced by `dependency-cruiser` (TS) and `import-linter` (Python). CI fails the build on violation.

### Special Cross-Cutting Packages

`vibe_observability` and `vibe_models` are foundational. They may be imported by any other package or service. They may not import anything that is not also a foundational package.

---

## Build Orchestration

### Turborepo

`turbo.json` defines pipelines. The `build` task depends on the `^build` of dependencies; this is the standard cascade.

```
"build": {
  "dependsOn": ["^build"],
  "outputs": ["dist/**"]
}
```

### Affected Detection

- A `turbo run build --filter=...[origin/main]` produces the affected set.
- CI uses this for the PR pipeline; release pipelines build everything.

### Caching

- Local cache: `.turbo/`.
- Remote cache: enabled per developer.
- The CI pipeline uploads the cache; pull requests from forks use `turbo` without remote cache (security).

---

## Versioning and Changesets

- Internal packages version together using `0.0.0` SemVer. Changesets track deltas; releases batch.
- Public packages (`@vibe/sdk`) version independently.
- Python packages version together at MVP, independently at V2.

A release PR generated by `changesets` is reviewed weekly and merged to publish.

---

## Local Development

See `26-local-development.md`. Highlights:

- `make bootstrap` provisions dependencies, runs migrations, and starts services.
- `make up` brings the platform up; `make down` tears down.
- `make test`, `make lint`, `make typecheck`, `make format-check` are standard.

---

## Pre-commit Hooks

- `ruff format`, `biome format`
- `ruff check --fix`, `biome lint --apply`
- `mypy`, `tsc --noEmit` (fast subset)
- `gitleaks`
- A `vibe check` script that runs domain-specific rules (e.g., tenant-filter guard in any PR touching repositories).

---

## CODEOWNERS

`CODEOWNERS` maps paths to teams. Approval by an owner is required to merge a PR touching those paths.

---

## Adding a New App or Package

The procedure:

1. Open an RFC issue describing the boundary, the consumers, and the dependency direction.
2. Get sign-off from a code owner for the area.
3. Create the directory with the standard layout.
4. Add the workspace member to `pnpm-workspace.yaml` or `pyproject.toml`.
5. Add the dependency rules to the linter configs.
6. Add the app to `infra/terraform/envs/<env>/`.
7. Update `docs/22-monorepo-structure.md` (this file) in the same PR.

---

## Assumptions

- The team accepts the cost of monorepo scale (larger clone, more CI config) for the benefit of unified tooling and atomic refactors.
- Build orchestration scales to ~30 apps and ~30 packages at MVP.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| pnpm + uv workspaces | Best-in-class for each ecosystem; deterministic and fast. |
| Apps / packages / services split | Aligns with deployable vs. shared vs. runtime. |
| Turborepo for TS orchestration | Mature, well-supported. |
| Strict dependency rules | Enforced boundaries prevent architectural drift. |
| Schema-first | One source of truth across languages. |
| Changesets for internal versioning | Decoupled release cadence per package. |

---

## Open Questions

- Should we move all Python services to a single repository in V2 to avoid cross-language coordination overhead?
- Should we adopt `nx` if Turborepo scaling becomes a concern?

---

## Future Enhancements

- Auto-generated dependency graph docs.
- "Domain map" rendered from CODEOWNERS for onboarding.
- Mandatory contract tests for any new cross-package import.

---

## Cross-References

- Coding standards → `21-coding-standards.md`
- Local dev → `26-local-development.md`
- Production deployment → `27-production-deployment.md`
- Workflows → `20-developer-workflows.md`
