# Contributing

Welcome. This is a small team building a large platform; the bar is high but the pace is real. Please read this before opening a PR.

## Read first

- Project entry point: [`docs/00-project-overview.md`](docs/00-project-overview.md)
- Coding standards: [`docs/21-coding-standards.md`](docs/21-coding-standards.md)
- Developer workflows: [`docs/20-developer-workflows.md`](docs/20-developer-workflows.md)
- Monorepo layout: [`docs/22-monorepo-structure.md`](docs/22-monorepo-structure.md)
- Local development: [`docs/26-local-development.md`](docs/26-local-development.md)
- The ledger: [`LEDGER.md`](LEDGER.md)

## Setup

```bash
git clone git@github.com:vibe-dev/gouplevelvibecoding.git
cd gouplevelvibecoding
make bootstrap
```

`make doctor` runs first and fails fast if any required tool is missing or mis-versioned.

## Workflow

1. Pick a story from [`LEDGER.md`](LEDGER.md) or open an issue if none fits.
2. Branch from `main`: `git switch -c <type>/<scope>-<short-desc>`.
3. Make small, focused commits. Imperative subject. Body explains the why. Reference the story ID.
4. Run `make lint typecheck test` locally before pushing.
5. Open a PR using the template. At minimum: what, why, how verified, risk, docs.
6. CI must be green. CODEOWNERS will request review from the right team.
7. Squash-merge with a conventional commit subject.

## Guardrails

- Don't introduce a dependency without justifying it in the PR body.
- Don't disable a lint or type rule without a justification in the PR body.
- Don't bypass the pre-commit hooks (`--no-verify`).
- Don't commit secrets. `.env.local` is local; CI uses GitHub Actions secrets.
- Don't grow a single function past 60 lines or a single file past 500 lines without a refactor note.

## Communication

- Public channels: GitHub Issues and Discussions.
- Security issues: see [`SECURITY.md`](SECURITY.md).
- Code of conduct: see [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md).
