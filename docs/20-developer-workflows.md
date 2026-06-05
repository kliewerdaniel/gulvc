# 20 — Developer Workflows

> How code travels from a developer's keyboard to production. The contract every contributor follows.

---

## Purpose

This document defines the developer workflows used by the Vibe team. It codifies the daily practices — branching, committing, reviewing, releasing — that allow a small team to ship safely and at speed.

It is the operating manual. The expectations here are the floor, not the ceiling.

---

## Scope

In scope:

- Branching strategy
- Commit hygiene
- Pull request process
- Code review standards
- Release process
- Hotfix process
- Local development
- On-call handoff

Out of scope:

- Coding style (see `21-coding-standards.md`)
- Production deployment internals (see `27-production-deployment.md`)

---

## Philosophy

1. **Trunk is sacred.** `main` is always deployable. PRs are small and merge fast.
2. **Review is for shared ownership.** Comments are teaching, not gatekeeping.
3. **CI is the source of truth.** Local-only tests do not exist; everything runs in CI.
4. **Reversibility is a feature.** Ship in small batches; rollback in seconds.
5. **Documentation is a deliverable.** A change is not done until the docs are updated.

---

## Repository Topology

- One monorepo (`gouplevelvibecoding/`) containing all source.
- Workspace-managed with `pnpm` for TS and `uv` for Python.
- A `tools/` directory hosts the platform's CLI, codegen, and scripts.

---

## Branching

We practice a trunk-based flow with short-lived feature branches.

- Default branch: `main`.
- Feature branches: `feat/<scope>-<short>`.
- Bug fix branches: `fix/<scope>-<short>`.
- Hotfix branches: `hotfix/<scope>-<short>` (use sparingly; explain in PR).
- Release branches: `release/<semver>` (created for cut, deleted on merge).

Naming conventions:

- Lowercase, hyphens, no issue numbers in branch names.
- Scope examples: `api`, `capture`, `analysis`, `generation`, `seo`, `deploy`, `docs`, `infra`.

Branch lifetime:

- Aim for ≤ 2 business days from branch open to merge.
- Rebase regularly; merge is a fast-forward or squash.

---

## Commits

### Format

```
<scope>: <imperative summary>

<optional body explaining the why>

<optional footer with ref / co-author / BREAKING CHANGE>
```

Example:

```
capture: dedupe navigation graph by canonical URL

Avoids a class of cycles when the same page is linked from
multiple navigation patterns.

Refs: VIBE-1423
```

### Rules

- Subject ≤ 72 chars; body wrapped at 100.
- One logical change per commit; no "WIP" commits on shared branches.
- Sign commits with GPG (see `15-github-integration.md`).
- Commit messages are written in the imperative ("add", not "added").
- Reference the issue or story ID in the body when applicable.
- Use `BREAKING CHANGE:` in the footer to flag breaking API changes.

---

## Pull Requests

### Lifecycle

1. **Branch open** — push the branch, open a draft PR.
2. **Ready for review** — convert to non-draft, assign reviewers.
3. **Review** — at least one approval required; for sensitive changes, two.
4. **Checks** — CI must be green.
5. **Merge** — squash-merge into `main`. The PR title becomes the commit subject.

### Required Checks

- All CI gates from `19-testing-strategy.md`.
- Documentation updated (validated by a doc-bot that scans the diff for missing updates against the doc index).
- Migration plan attached if schema changes.
- Rollback plan attached if the change has user-visible risk.

### Size Guidance

- Aim for ≤ 400 lines of diff (excluding generated code).
- Larger changes must be split, or accompanied by a "why we can't split" rationale in the PR template.

### Templates

PRs follow a template (`/.github/PULL_REQUEST_TEMPLATE.md`) with sections:

- Summary
- Why
- How
- Test plan
- Risk and rollback
- Docs updated
- Issue link

### Reviewer SLA

- First response within 4 business hours.
- Approval within 1 business day for standard changes.
- For changes labeled `urgent`, first response within 1 hour during business hours.

---

## Code Review

### Reviewer Responsibilities

- Check correctness, safety, performance, testability, and clarity.
- Verify the diff matches the PR description.
- Confirm CI is green.
- For domain-sensitive code (security, billing, tenancy, generation), apply the relevant review checklist.

### Author Responsibilities

- Reviewer comments are addressable; if you disagree, explain, don't argue.
- After addressing, push the resolution and resolve threads.
- Re-request review after substantive changes.

### Review Checklists

- **Security review checklist** (`apps/security/review/security.md`): every PR touching auth, secrets, multi-tenancy, or any external I/O.
- **Agent review checklist** (`apps/agents/review/agent.md`): every PR touching agents.
- **Data review checklist** (`apps/data/review/data.md`): every PR touching Postgres or migrations.

---

## Releases

### Versioning

Semantic versioning for the public API and CLI. Internal services version per build SHA.

### Cadence

- Continuous delivery: every merge to `main` produces a staging deploy.
- Production releases: weekly on Tuesdays at 16:00 UTC, with a release train.
- Ad-hoc releases for hotfixes.

### Release Process

1. Cut a `release/x.y.z` branch.
2. Run the release checklist (`27-production-deployment.md`).
3. Tag the commit; CI builds artifacts.
4. Deploy to production via progressive rollout.
5. Monitor SLOs for 30 minutes; complete or roll back.
6. Merge the release branch back to `main`.
7. Announce in `#releases` with a summary and the changelog.

### Changelog

- Generated from conventional commits via `release-please`.
- Hosted at `CHANGELOG.md` in the monorepo.
- Customer-facing changelog at `vibe.dev/changelog` is curated monthly from the internal changelog.

---

## Hotfixes

Used only for:

- SEV1/S2 incidents.
- Security patches.
- Billing or data integrity issues.

Process:

1. Branch `hotfix/...` from the latest production tag.
2. Apply the minimal change; add a test that reproduces the bug.
3. Expedited review: one approver is enough for a hotfix, but the reviewer is a code owner for the affected area.
4. CI runs the full suite; if a non-blocking check is broken, note and follow up.
5. Tag and release as `x.y.(z+1)`.
6. Back-merge into `main` (and any active release branch) within the same day.

---

## Code Owners

`CODEOWNERS` defines the owners for each area. Approval by an owner is required for:

- `apps/api/**` → API team
- `apps/capture/**` → Capture team
- `apps/agents/**` → Agents team
- `apps/generation/**` → Generation team
- `apps/deploy/**` → Deploy team
- `infra/**` → Platform team
- `docs/**` → Docs team
- Anything touching tenancy, billing, or auth → Eng leads

---

## Local Development

See `26-local-development.md` for the full environment setup. Highlights:

- `make bootstrap` provisions everything.
- `make up` brings services up; `make down` tears down.
- `make test`, `make lint`, `make typecheck` are the standard loop.
- A `vibe` CLI exists for one-off tasks: `vibe jobs run`, `vibe agent replay`, `vibe docs serve`.

---

## Issue Tracking

- Primary: GitHub Issues.
- Workflow labels: `triage`, `design`, `ready`, `in-progress`, `in-review`, `blocked`, `done`, `wontfix`.
- Priority labels: `p0`, `p1`, `p2`, `p3`.
- Area labels: `area/api`, `area/capture`, `area/agents`, `area/generation`, `area/deploy`, `area/docs`, `area/infra`.
- An issue may not be closed without a linked PR or an explicit `wontfix` comment from a code owner.

---

## Communication

- **Daily standup:** 15 min, focused on blockers, scope, and risks.
- **Weekly planning:** issues are groomed; milestones are committed.
- **Weekly demo:** internal and customer-facing releases are demoed.
- **Async-first:** decisions are written in docs or RFCs; meetings confirm or refine.
- **Channels:**
  - `#eng` — general engineering
  - `#incidents` — firehose during incidents
  - `#releases` — release announcements
  - `#ops` — production observations

---

## On-Call Handoff

Weekly on Monday at 11:00 local time, the on-call pair meets for 15 min:

- Outstanding alerts
- Recent incidents
- Open follow-ups
- Current runbook gaps
- Upcoming maintenance

Notes are committed to `apps/runbooks/handoff/<date>.md`.

---

## Engineering Hygiene

- All repos use Dependabot for security updates.
- Renovate for non-security dependency bumps.
- Stale branches (no commits in 14 days) are auto-deleted after a warning.
- Untagged Docker images are retained 7 days; tagged images are retained 90 days.
- Secrets must never be pasted into chat.

---

## Assumptions

- The team has the operational discipline to keep PRs small.
- CI is fast enough to keep the feedback loop short.
- On-call coverage is staffed.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Trunk-based with short-lived branches | Lower merge pain, faster feedback. |
| Squash-merge | Linear history; one commit per change. |
| Required CI green on merge | Defends trunk. |
| Weekly release train | Predictable rhythm. |
| Code owners by area | Domain expertise gates risky changes. |

---

## Open Questions

- Should we adopt merge queues for stricter trunk protection at V2?
- Should we use ephemeral preview environments for every PR?

---

## Future Enhancements

- Auto-generated "risk score" on PRs based on changed paths.
- AI-assisted PR summaries.
- Code review of generated code as part of the agent suite.

---

## Cross-References

- Standards → `21-coding-standards.md`
- Monorepo → `22-monorepo-structure.md`
- Testing → `19-testing-strategy.md`
- Releases → `27-production-deployment.md`
- Risk → `25-risk-analysis.md`
