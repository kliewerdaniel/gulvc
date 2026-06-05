# 15 — GitHub Integration

> The contract between the platform and GitHub. Tokens, scopes, webhooks, repo lifecycle.

---

## Purpose

This document specifies how Vibe integrates with GitHub. It is the operational contract for everyone working with the GitHub side of the deployment pipeline.

It complements `14-deployment-engine.md`, which describes the deployment engine end-to-end. This document focuses on the GitHub-specific concerns: app installation, token minting, permissions, webhooks, and repo lifecycle.

---

## Scope

In scope:

- GitHub App design (Vibe-owned)
- Tenant-installed GitHub Apps (white-label)
- Permissions and scopes
- Token minting and rotation
- Webhook ingestion
- Repository lifecycle (create, push, archive, delete)
- Branch protection
- Commit signing
- Per-tenant isolation

Out of scope:

- Vercel integration (`16-vercel-integration.md`)
- Generation of repo contents (`12-generation-engine.md`)
- Security model overall (`17-security-model.md`)

---

## Why a GitHub App (Not a Personal Access Token)

- **Scoped permissions** rather than user-account-wide access.
- **Installation tokens** are short-lived (1 hour) and minted per-call.
- **Audit trails** clearly distinguish "Vibe Bot" from human collaborators.
- **Per-installation isolation** prevents one tenant's compromise from affecting others.
- **Higher rate limits** at the installation level.
- **First-class commit signing** via the App's GPG key.

PATs are explicitly rejected for production use.

---

## App Topology

Vibe operates **two** GitHub App identities:

### Vibe Platform App

- Owner: the Vibe org.
- Installed on: the Vibe org's own repos used for delivery when the customer hasn't connected their own GitHub.
- Default destination for Tier 1 (one-time modernization) customers who do not bring their own GitHub.

### Vibe Connect App (Customer-Installed)

- Owner: the Vibe org.
- Marketplace listing: `Go Up Level Vibe Coding`.
- Installable by any customer in their personal account or org.
- Required for Tier 3 (white-label) deployments into the customer's GitHub.

The platform uses the **Connect App** by default for customer-owned repos. The **Platform App** is used only when no Connect installation exists for the tenant.

---

## Permissions

The Connect App requests the minimum permissions necessary.

### Repository Permissions

| Permission | Level | Why |
|------------|-------|-----|
| Actions | Read | Inspect CI status for redeploy decisions (V2) |
| Administration | Write | Create repos |
| Contents | Write | Push generated code |
| Issues | Write | Open issues for maintenance PRs (V3) |
| Metadata | Read | Required, default |
| Pull requests | Write | Open maintenance PRs (V3) |
| Webhooks | Read | Verify deployments |

### Organization Permissions

| Permission | Level | Why |
|------------|-------|-----|
| Administration | Read | Read org settings for placement decisions |
| Members | Read | Validate the installer's authority to deploy into the org |

### Account Permissions

None requested.

---

## Webhook Subscriptions

The Connect App subscribes to:

- `installation` (created, deleted, suspend, unsuspend)
- `installation_repositories` (added, removed)
- `repository` (created, deleted, archived, transferred)
- `push` (for maintenance subscription, V3)
- `pull_request` (for maintenance subscription, V3)
- `check_suite` and `check_run` (for CI integration, V3)

All webhooks deliver to `https://api.vibe.dev/v1/webhooks/github` with HMAC-SHA256 verification using the App's webhook secret.

---

## Installation Lifecycle

```mermaid
sequenceDiagram
    actor U as Customer
    participant GH as GitHub
    participant V as Vibe
    U->>GH: Install Vibe Connect App on org X
    GH->>V: webhook installation.created (installation_id)
    V->>V: Persist github_installations row, link to tenant
    V-->>U: Connection successful
    U->>V: Configure tenant to deploy to org X
    V->>V: Mark tenant.github_installation_id
```

Suspension and uninstallation flow similarly; the platform marks installations `uninstalled_at` and routes future deployments to the Platform App fallback (or fails if customer required org X).

---

## Token Minting

The platform mints installation tokens just-in-time per side-effecting call.

```python
async def get_installation_token(installation_id: int) -> str:
    # 1. Mint a JWT signed with the App's private key
    jwt_token = mint_app_jwt(VIBE_GITHUB_APP_ID, VIBE_GITHUB_APP_PRIVATE_KEY)
    # 2. Exchange for an installation access token (1 hour TTL)
    async with httpx.AsyncClient() as client:
        resp = await client.post(
            f"https://api.github.com/app/installations/{installation_id}/access_tokens",
            headers={"Authorization": f"Bearer {jwt_token}", "Accept": "application/vnd.github+json"},
        )
        resp.raise_for_status()
    return resp.json()["token"]
```

Implementation rules:

- Cache tokens for ≤ 55 minutes in Redis, keyed by `installation_id`.
- Never log token bodies.
- Refresh on `401` from any subsequent call.
- Use a dedicated Secrets Manager entry for the App's private key.

---

## Rate Limiting

GitHub limits installations to 5,000 requests/hour. The platform stays well below this through:

- Aggressive token caching.
- Bulk operations where the API allows (e.g., creating files in a single tree).
- Per-installation queueing if a tenant's hourly usage exceeds 80% of the limit.

The platform exposes `X-RateLimit-*` metrics from GitHub responses to operators via dashboards.

---

## Repository Lifecycle

### Create

```python
async def create_repo(
    installation_id: int,
    owner: str,
    name: str,
    visibility: Literal["public", "private"] = "private",
) -> CreatedRepo:
    token = await get_installation_token(installation_id)
    async with httpx.AsyncClient() as client:
        if is_user(owner):
            resp = await client.post(
                "https://api.github.com/user/repos",
                json={
                    "name": name,
                    "private": visibility == "private",
                    "auto_init": False,
                    "description": "Generated by Vibe",
                    "homepage": None,
                },
                headers={"Authorization": f"token {token}"},
            )
        else:
            resp = await client.post(
                f"https://api.github.com/orgs/{owner}/repos",
                json={...},
                headers={"Authorization": f"token {token}"},
            )
    resp.raise_for_status()
    return CreatedRepo.parse_obj(resp.json())
```

Conflict handling: on `422 name already exists`, append `-vibe-<short_id>` and retry once.

### Push

- Use git over HTTPS with the installation token as the password.
- Use `x-access-token` as the username (GitHub convention).
- Commits are signed with the App's GPG key.

Example:

```bash
git remote add origin "https://x-access-token:${TOKEN}@github.com/${OWNER}/${REPO}.git"
git push origin main
```

### Branch Protection (V2+)

Apply on `main`:

- Require pull request reviews (1 approval) for V2 maintenance flow.
- Require status checks (Vercel) to pass.
- Restrict who can push directly to `main` (only Vibe Bot for autonomy mode).

### Archive

- Available via admin action.
- Archive does not delete the repo or compensate Vercel.

### Delete

- Used only for compensation when a deploy fails before any external traffic.
- Requires `Administration: write` permission.

---

## Commit Signing

- The App carries a GPG key whose public half is uploaded to GitHub.
- The local Git config in the workspace clone sets:
  ```
  git config user.signingkey <KEY_ID>
  git config commit.gpgsign true
  git config gpg.program gpg
  ```
- The private key is mounted from Secrets Manager at worker boot, scoped per environment.

Result: all commits show "Verified" in the GitHub UI.

---

## Per-Tenant Isolation

- Each tenant's installations are stored in `github_installations` keyed by `(tenant_id, installation_id)`.
- The platform never uses one tenant's installation token to act on another tenant's resources.
- IAM policies on Secrets Manager restrict workers to retrieving only the keys they need.

---

## Failure Mode Matrix

| Failure | Detection | Recovery |
|---------|-----------|----------|
| Installation revoked between calls | 401 / 404 | Mark `uninstalled_at`; route job to fallback or fail. |
| Rate limit | 429 / 403 with `X-RateLimit-Remaining=0` | Backoff to `X-RateLimit-Reset`. |
| Network blip | timeout, 502 | Retry with jitter, max 5. |
| Repo creation conflict | 422 | Append suffix, retry once. |
| Push rejected (non-fast-forward) | git error | Should not occur on initial commit; investigate. |
| GPG key mismatch | git error | Re-fetch key from Secrets Manager. |
| Webhook signature mismatch | HMAC verify fail | Reject, log, alert. |

---

## Operational Procedures

### Rotating the App Private Key

1. Generate a new private key in GitHub.
2. Upload to Secrets Manager under a new version.
3. Update worker config to point to the new ARN.
4. Deploy.
5. After 24 hours, revoke the old key in GitHub.

### Rotating the App Webhook Secret

1. Generate a new secret in GitHub.
2. Upload to Secrets Manager.
3. Update the webhook handler to accept both old and new for 1 hour.
4. Cut over.

### Customer Removes Their Connect Installation

1. We receive `installation.deleted` webhook.
2. We mark `github_installations.uninstalled_at`.
3. Future jobs for the tenant either:
   - Fall back to the Platform App (if tenant's tier allows), or
   - Fail at intake with `github_installation_required`.
4. Customer is notified.

---

## Compliance & Audit

- All API calls are logged (without bodies) with `installation_id`, `endpoint`, `status`, `request_id`, `tenant_id`.
- Webhook receipts are logged with `delivery_id` (GitHub-provided) for cross-correlation.
- Audit log entries are emitted on:
  - Installation created or revoked
  - Repo created or deleted
  - Branch protection changes

---

## Testing Strategy

- **Unit:** token minting, signature verification, slugification, conflict-resolution.
- **Integration:** against a dedicated test GitHub org (`vibe-it-test`). Repos are created and deleted per test run.
- **Contract:** record-replay (VCR) cassettes refreshed quarterly.
- **Chaos:** simulated 401 / 429 / 502 responses verify retries and token refresh.

---

## Observability

- Counter `vibe.github.api_calls_total{endpoint,status}`
- Histogram `vibe.github.api_latency_ms`
- Counter `vibe.github.rate_limit_remaining{installation_id}`
- Span attributes: `github.installation_id`, `github.repo`, `github.commit_sha`

---

## Assumptions

- GitHub App permissions are sufficient for the V1 and V2 lifecycle.
- The customer trusts Vibe to administer (create/delete) the specific repositories it creates.
- GitHub remains the dominant code host for our target customers.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| GitHub App + Installation Tokens | Least privilege, audit, rotation. |
| Two apps (Platform + Connect) | Clean separation between Vibe-owned and customer-owned repos. |
| Commit signing | Trust and auditability for customers. |
| JIT token minting + cache | Performance and safety. |
| Hard limit on which tenants can push to which orgs | Prevents accidental cross-tenant action. |

---

## Open Questions

- Should we offer a GitLab/Bitbucket equivalent at V3?
- Should branch protection be enabled on MVP repos to prevent accidental customer pushes?
- Should we open a tour PR (`tour-of-your-new-site.md`) with onboarding tips for human customers?

---

## Future Enhancements

- A "managed maintenance" workflow that opens PRs for dependency updates and SEO improvements (V3).
- Native support for self-hosted GitHub Enterprise (V3).
- A Slack / Linear integration that surfaces deploy status next to repos.

---

## Cross-References

- Deployment engine → `14-deployment-engine.md`
- Vercel integration → `16-vercel-integration.md`
- Security → `17-security-model.md`
- Database tables → `08-database-design.md` (`github_installations`, `github_repos`)
