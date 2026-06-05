# 21 — Coding Standards

> Conventions every contributor must follow. The style and discipline that make the codebase readable and changeable.

---

## Purpose

This document defines the coding standards used across the Vibe codebase. It is the agreement we make with each other about how code is written, named, organized, tested, and documented.

Consistency is the goal. The rules here are not arbitrary; each has a reason. Where a rule is not the right call, we change the rule for everyone, not the local practice for one PR.

---

## Scope

In scope:

- General language-agnostic rules
- Python standards
- TypeScript / JavaScript standards
- SQL standards
- Documentation in code
- Linting, formatting, type rules
- Project layout

Out of scope:

- Repo layout (`22-monorepo-structure.md`)
- Testing practices (`19-testing-strategy.md`)

---

## General Rules

1. **Readability beats cleverness.** A junior engineer should be able to read the code and understand the intent within a few minutes.
2. **Names are documentation.** Choose precise, intention-revealing names.
3. **Functions do one thing.** If you need "and" to describe it, split it.
4. **Dependencies are explicit.** Every import is intentional. No star imports.
5. **Errors are first-class.** Every error path is reachable, named, and handled.
6. **No dead code.** Unused code is removed, not commented out.
7. **No magic numbers.** Name constants; reference them.
8. **No comments narrating the code.** Comments explain *why*, not *what*.
9. **Side effects are isolated.** Pure functions by default; impure wrappers thin.
10. **Time and randomness are injected.** Never call `datetime.now()` or `random.random()` deep in a function.

---

## Python Standards

### Version and Tooling

- Python 3.12.
- `uv` for dependency management.
- `ruff` for linting and formatting.
- `mypy --strict` for type checking.
- `pytest` for testing.

### Style

- Line length: 100.
- Use double quotes for strings.
- Trailing commas in multi-line constructs.
- Type hints are required on all public functions and most private ones.
- Use `from __future__ import annotations` only where needed; prefer modern syntax.
- Prefer `match` for dispatch where it reads cleanly.
- Prefer `dataclasses` or `pydantic.BaseModel` over ad-hoc dicts.
- Use `enum.StrEnum` for enums.

### Imports

- Order: stdlib, third-party, local. Separated by blank lines.
- Use absolute imports.
- Alias only when necessary.

### Async

- Use `asyncio` consistently. Mixing sync and async requires explicit justification.
- Never block the event loop; offload CPU-bound work with `asyncio.to_thread` or to a worker pool.
- Cancellation must propagate; handle `asyncio.CancelledError` carefully.

### Errors

- Define a small hierarchy of domain exceptions in `apps/<domain>/errors.py`.
- Raise specific exceptions; catch specific exceptions.
- Never catch `Exception` and continue silently.

### Logging

- Use `structlog` everywhere.
- Bind context explicitly: `logger.bind(job_id=..., tenant_id=...)`.
- Never log secrets or raw HTML.

### Type Discipline

- `mypy --strict` must pass.
- Prefer `TypedDict` for unstructured data; `BaseModel` for validated data.
- Generic typing for reusable functions and classes.
- No `Any` unless wrapping an untyped third-party library, with a comment explaining.

### Examples

```python
async def capture_page(
    url: CanonicalUrl,
    ctx: CaptureContext,
    fetcher: PageFetcher,
) -> CapturedPage:
    if not ctx.policy.allows(url):
        raise PolicyViolation(url=url, reason=ctx.policy.reason)
    span = trace.get_current_span()
    span.set_attribute("url", str(url))
    try:
        return await fetcher.fetch(url, ctx=ctx)
    except FetcherTimeout as e:
        span.set_status(Status(StatusCode.ERROR, str(e)))
        logger.warning("capture.timeout", url=str(url), timeout_ms=ctx.timeout_ms)
        raise
```

---

## TypeScript / JavaScript Standards

### Version and Tooling

- TypeScript 5.5; target ES2022.
- `pnpm` for dependency management.
- `biome` for linting and formatting (single tool for speed).
- `tsc --strict` for type checking.
- `vitest` for testing.

### Style

- Line length: 100.
- Single quotes for strings; template literals for interpolation.
- Trailing commas required.
- `const` by default; `let` only when reassignment is needed; never `var`.
- Prefer `function` declarations at top level; arrow functions for callbacks.
- Use `import type` for type-only imports.

### Types

- `strict: true` with `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noImplicitOverride`, `noFallthroughCasesInSwitch`.
- Prefer discriminated unions for state.
- No `any`; use `unknown` and narrow.
- Branded types for IDs: `type JobId = string & { readonly __brand: "JobId" }`.

### React / Next.js

- Function components only.
- Co-locate styles, tests, and stories.
- Use server components by default in the dashboard; client components only when necessary.
- Avoid `useEffect` for derived state; lift state up or use server actions.
- Use `zod` for input validation at boundaries.

### Errors

- Throw `Error` subclasses; never string errors.
- Catch at boundaries; never silently swallow.
- Convert to typed results (`Result<T, E>`) when propagation is preferable to throw.

### Examples

```ts
type CapturePageInput = { url: string; ctx: CaptureContext };

export async function capturePage(
  input: CapturePageInput,
  fetcher: PageFetcher,
): Promise<Result<CapturedPage, CaptureError>> {
  const normalized = canonicalizeUrl(input.url);
  if (!normalized.ok) return { ok: false, error: normalized.error };
  return await fetcher.fetch(normalized.value, input.ctx);
}
```

---

## SQL Standards

- Use `lower_snake_case` for identifiers.
- All tables have a primary key named `id` of type `uuid`.
- All tables have `created_at` and `updated_at` (`timestamptz`).
- Foreign keys named `<table>_id` (e.g., `tenant_id`).
- Index names: `idx_<table>_<columns>`; unique indexes: `uniq_<table>_<columns>`.
- Constraints named: `chk_<table>_<rule>`, `fk_<table>_<ref>`, `pk_<table>`.
- Migrations are forward-only; down migrations are produced only for data corrections and named explicitly.
- Avoid `SELECT *`; select columns explicitly.
- Use CTEs for readability when the query is multi-step.
- Always set a `LIMIT` for ad-hoc exploration queries.

---

## Documentation in Code

- Every public module, class, and function has a docstring or JSDoc.
- Docstrings describe *what* and *why*, not *how*.
- Examples are encouraged for non-obvious APIs.
- Generated code (e.g., codegen, generated TS clients) is excluded.

Python example:

```python
def canonicalize_url(url: str) -> CanonicalUrl:
    """Normalize a URL to a canonical, comparison-safe form.

    Lower-cases host, strips default ports, removes tracking
    parameters, and resolves relative segments.

    Raises:
        InvalidUrlError: if the URL is not parseable.
    """
```

TypeScript example:

```ts
/**
 * Normalize a URL to a canonical, comparison-safe form.
 *
 * Lower-cases host, strips default ports, removes tracking parameters,
 * and resolves relative segments.
 *
 * @throws {InvalidUrlError} if the URL is not parseable.
 */
```

---

## Commit Hygiene (refers to `20-developer-workflows.md`)

- Small, focused commits.
- Imperative subject, body explains why.
- Reference the story ID in the body.

---

## Linting and Type Rules

- Lint must pass with zero warnings in CI.
- Type checking must pass with strict settings in CI.
- Pre-commit hooks run the formatter and the secret scanner.
- A new rule addition is a PR; disabling a rule on a line is a PR with justification.

---

## Code Layout Rules

- Maximum file length: 500 lines (soft), 1000 lines (hard — requires a refactor note).
- Maximum function length: 60 lines (soft).
- Maximum cyclomatic complexity: 12 (soft).
- No circular imports.
- No global mutable state.

---

## Dependency Discipline

- Adding a new dependency is a PR; the body explains why existing options are insufficient.
- All dependencies must be compatible with the project's license.
- Heavy or unusual dependencies (LLM, browser, image processing) require an additional review by a code owner.
- Dependencies are pinned; lockfiles are committed.
- `pip-audit` / `pnpm audit` run in CI.

---

## Performance Conventions

- Avoid N+1 queries; eager-load where possible.
- Cache external HTTP responses when idempotent.
- Stream large payloads rather than buffering.
- Use `EXPLAIN ANALYZE` for any new query that touches > 100k rows.

---

## Security Conventions

- Never log secrets.
- Never `eval` or `exec` untrusted input.
- Validate every input at the boundary.
- Parameterize every SQL query; never string-concatenate.
- Set security headers at the edge and verify in tests.
- For multi-tenant code paths, call the tenancy filter explicitly and add a test that asserts isolation.

---

## Open Source Hygiene

- All source files include the project's license header.
- NOTICE file lists attributions.
- No proprietary code is committed without a license review.

---

## Assumptions

- The team has the discipline to fix linting warnings rather than suppress them.
- Code review is the place to enforce these rules; tooling is the backstop.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| `ruff` for Python | Single tool, fast, no config sprawl. |
| `biome` for TS | Single tool, fast, no Prettier/ESLint split. |
| `mypy --strict` and `tsc --strict` | Catches entire classes of bugs. |
| File length soft caps | Encourages decomposition. |
| Comments explain why | Aligns with readability principle. |

---

## Open Questions

- Should we adopt `ruff format` and remove Black config from older services?
- Should we move to `deno fmt` for tooling consolidation at V2?

---

## Future Enhancements

- Auto-generated code style reports per PR.
- AI-assisted "reviewer for style" in PRs.

---

## Cross-References

- Testing → `19-testing-strategy.md`
- Monorepo → `22-monorepo-structure.md`
- Security → `17-security-model.md`
- Workflows → `20-developer-workflows.md`
