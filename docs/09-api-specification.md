# 09 — API Specification

> The public and internal HTTP surface of the platform. The contract between callers and the system.

---

## Purpose

This document specifies the HTTP API exposed by Vibe. It is the canonical reference for:

- The public REST API (`/v1/...`)
- Webhooks delivered to customers
- Authentication and authorization
- Error semantics
- Pagination, filtering, sorting
- Rate limiting
- Versioning

The machine-readable counterpart is an OpenAPI 3.1 document published at `https://api.vibe.dev/openapi.json`. This document is the human-readable companion and the source of design decisions behind the OpenAPI surface.

---

## Scope

In scope:

- API contracts for `POST /capture`, `POST /analyze`, `POST /generate`, `POST /deploy`, `GET /projects`, `GET /projects/{id}`, `GET /reports/{id}`, plus the canonical job-oriented endpoints
- Authentication schemes
- Error model
- Webhook contracts
- Versioning policy

Out of scope:

- Internal RPC between services (in-process and Temporal activity calls, see `02-system-architecture.md`)
- Admin-only endpoints (covered by `apps/admin-api/openapi.yaml`)
- SDK design (V3)

---

## API Design Principles

1. **Resource-oriented.** Endpoints map to nouns; HTTP verbs map to operations.
2. **JSON-only.** Requests and responses are `application/json; charset=utf-8`. Errors are `application/problem+json`.
3. **Versioned by URL prefix.** All routes live under `/v1/`. Breaking changes require `/v2/`.
4. **Idempotent writes.** All `POST` endpoints honor `Idempotency-Key`.
5. **Async by default.** Long-running operations return `202 Accepted` and a status URL.
6. **Cursor pagination.** No offset pagination.
7. **Strict input validation.** Pydantic schemas; unknown fields rejected.
8. **HATEOAS-light.** Resources include `_links` for next steps but the API is not strictly hypermedia.
9. **No PATCH on jobs.** Jobs are immutable once created; state transitions happen internally.
10. **Webhooks are signed.** HMAC-SHA256 with timestamp; replay-protected.

---

## Endpoint Map

### Canonical Job-Oriented Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/v1/jobs` | Create a new modernization job |
| `GET` | `/v1/jobs` | List jobs |
| `GET` | `/v1/jobs/{id}` | Retrieve a job |
| `POST` | `/v1/jobs/{id}/cancel` | Cancel a job |
| `POST` | `/v1/jobs/{id}/replay` | Replay a job from a given phase |
| `GET` | `/v1/jobs/{id}/artifacts` | List artifacts for a job |
| `GET` | `/v1/jobs/{id}/events` | List events for a job (SSE supported) |
| `GET` | `/v1/reports/{id}` | Retrieve a job's report |
| `GET` | `/v1/projects` | List projects (alias for completed jobs grouped by tenant intent) |
| `GET` | `/v1/projects/{id}` | Retrieve a project |

### Phase-Scoped Endpoints (advanced / partner-facing)

These allow API callers to invoke individual phases against existing artifacts. Useful for partners who want to bring their own capture data or re-run a single phase.

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/v1/capture` | Run capture against a URL (no full job) |
| `POST` | `/v1/analyze` | Run analysis against a capture manifest |
| `POST` | `/v1/generate` | Run generation against a site model |
| `POST` | `/v1/deploy` | Run deployment against a workspace archive |

These are MVP-optional and shipped in M6.

### Account & Auth

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/v1/auth/magic-link` | Request a sign-in magic link |
| `POST` | `/v1/auth/verify` | Verify a magic link token |
| `POST` | `/v1/auth/refresh` | Refresh access token |
| `POST` | `/v1/auth/logout` | Revoke a session |
| `GET` | `/v1/me` | Retrieve current user |
| `GET` | `/v1/tenants/{id}` | Retrieve tenant |
| `PATCH` | `/v1/tenants/{id}` | Update tenant settings |
| `GET` | `/v1/api-keys` | List API keys |
| `POST` | `/v1/api-keys` | Create API key |
| `DELETE` | `/v1/api-keys/{id}` | Revoke API key |

### Billing

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/v1/billing/checkout` | Create a Stripe checkout session |
| `GET` | `/v1/billing/portal` | Get a Stripe customer portal URL |
| `POST` | `/v1/billing/webhooks/stripe` | Stripe webhook ingest |

### Webhooks (outgoing configuration)

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/v1/webhooks` | List configured endpoints |
| `POST` | `/v1/webhooks` | Register an endpoint |
| `DELETE` | `/v1/webhooks/{id}` | Remove an endpoint |
| `POST` | `/v1/webhooks/{id}/rotate-secret` | Rotate HMAC secret |

---

## Authentication

Two schemes are supported.

### Bearer JWT (customer dashboard)

```
Authorization: Bearer eyJhbGciOi...
```

- Issued by `/v1/auth/verify` after magic-link verification.
- Access token lifetime: 15 minutes.
- Refresh tokens (HTTP-only cookies) lifetime: 30 days, sliding.
- JWT carries `sub` (user id), `tenant` (tenant id), `role`, `exp`.

### API Key (partner / programmatic)

```
Authorization: Bearer vibe_live_<random>
```

- Created via `/v1/api-keys`.
- Tied to a tenant; scoped via `scopes` (e.g., `jobs.write`, `jobs.read`).
- Argon2id-hashed at rest. Plaintext shown once at creation.

API key prefixes:

- `vibe_live_...` — live environment
- `vibe_test_...` — test mode (no charges, sandbox deployments)

---

## Common Headers

### Request

| Header | Purpose |
|--------|---------|
| `Authorization` | Bearer JWT or API key |
| `Idempotency-Key` | UUID for idempotent writes (24 h window) |
| `X-Request-Id` | Optional caller-supplied request id (echoed in response) |
| `Content-Type` | `application/json` for write requests |
| `Accept` | `application/json` or `text/event-stream` for SSE |

### Response

| Header | Purpose |
|--------|---------|
| `X-Request-Id` | Server-assigned (or echoed) request id |
| `X-RateLimit-Limit` | Tier's per-route limit |
| `X-RateLimit-Remaining` | Remaining quota in window |
| `X-RateLimit-Reset` | Unix epoch seconds when quota resets |
| `RateLimit` / `RateLimit-Policy` | RFC 9210 draft headers |
| `Deprecation` | `true` if endpoint is deprecated |
| `Sunset` | Date the endpoint will be removed |

---

## Error Model

All errors use `application/problem+json` per RFC 7807.

```json
{
  "type": "https://vibe.dev/errors/job-quota-exceeded",
  "title": "Quota exceeded",
  "status": 402,
  "detail": "Tenant has used 100/100 jobs in the current billing period.",
  "instance": "/v1/jobs",
  "request_id": "req_01HZX...",
  "errors": [
    { "field": null, "code": "quota_exceeded" }
  ]
}
```

### Status Code Conventions

| Code | Meaning |
|------|---------|
| 200 | Successful read |
| 201 | Resource created (synchronous) |
| 202 | Accepted, processing async |
| 204 | Successful with no body |
| 400 | Validation error |
| 401 | Missing or invalid credentials |
| 403 | Authenticated but not authorized |
| 404 | Resource not found |
| 409 | Conflict (e.g., idempotency mismatch) |
| 422 | Semantic validation error |
| 429 | Rate limit exceeded |
| 500 | Internal error |
| 502 | Upstream provider failure |
| 503 | Service unavailable / degraded |
| 504 | Upstream timeout |

### Error Code Catalogue (excerpt)

| Code | HTTP | Description |
|------|------|-------------|
| `invalid_url` | 400 | Submitted URL is malformed or unsupported |
| `private_address_forbidden` | 400 | URL resolves to a private address |
| `domain_blocked` | 400 | Domain is on the platform denylist |
| `quota_exceeded` | 402 | Tier quota exhausted |
| `payment_required` | 402 | No active payment method or subscription |
| `unauthenticated` | 401 | Missing credentials |
| `forbidden` | 403 | Not authorized for this resource |
| `not_found` | 404 | Resource not found |
| `idempotency_conflict` | 409 | `Idempotency-Key` reused with different payload |
| `validation_failed` | 422 | Request body failed schema validation |
| `rate_limited` | 429 | Too many requests |
| `provider_unavailable` | 502 | An upstream provider is failing |

---

## Pagination

Cursor-based.

Request:

```
GET /v1/jobs?limit=50&cursor=eyJ...
```

Response:

```json
{
  "data": [ /* items */ ],
  "page": {
    "cursor": "eyJ...",
    "next_cursor": "eyJ...",
    "has_more": true
  }
}
```

Limits:

- `limit` default 25, max 100.
- Cursors are opaque, signed, and short-lived (24 h).

---

## Filtering & Sorting

Filters:

```
GET /v1/jobs?status=failed&created_after=2026-01-01T00:00:00Z
```

Supported filter operators are explicit in the OpenAPI spec per field. Free-form query languages are not supported.

Sorting:

```
GET /v1/jobs?sort=-created_at
```

Sort fields are explicitly enumerated.

---

## Rate Limiting

- Token-bucket per `(tenant, route_group)` in Redis.
- Default limits per tier are documented in the customer dashboard.
- Limits returned via `X-RateLimit-*` headers.
- 429 responses include `Retry-After` (seconds).

Indicative defaults (free tier):

| Route Group | Limit |
|-------------|-------|
| `POST /v1/jobs` | 5 / hour |
| `GET /v1/jobs/*` | 60 / minute |
| Auth | 10 / minute |

Indicative defaults (api_volume tier):

| Route Group | Limit |
|-------------|-------|
| `POST /v1/jobs` | 200 / minute |
| `GET /v1/jobs/*` | 600 / minute |

---

## Endpoint Reference

The following sections give request and response shapes for the core endpoints. Field types follow Pydantic conventions.

---

### `POST /v1/jobs`

Create a new modernization job.

**Request**

```http
POST /v1/jobs HTTP/1.1
Host: api.vibe.dev
Authorization: Bearer vibe_live_abc123
Content-Type: application/json
Idempotency-Key: 7c2a3e3c-9b0d-4d6a-9c45-1f5b1e6a2c2a

{
  "url": "https://example.com",
  "crawl_depth": 2,
  "max_pages": 25,
  "template": "default",
  "callback_url": "https://partner.example/vibe/webhooks",
  "metadata": {
    "external_id": "ext-789",
    "client_name": "Acme Plumbing"
  }
}
```

**Response — 202 Accepted**

```json
{
  "id": "job_01HZX5K2Q9...UUIDV7",
  "status": "queued",
  "url": "https://example.com",
  "canonical_url": null,
  "created_at": "2026-06-05T11:18:00Z",
  "_links": {
    "self": "/v1/jobs/job_01HZX5K2Q9...",
    "events": "/v1/jobs/job_01HZX5K2Q9.../events",
    "cancel": "/v1/jobs/job_01HZX5K2Q9.../cancel"
  }
}
```

**Errors**

| Code | Reason |
|------|--------|
| 400 `invalid_url` | URL is malformed |
| 400 `private_address_forbidden` | Hostname resolves to private IP |
| 400 `domain_blocked` | Denylisted domain |
| 402 `quota_exceeded` | Tier quota used up |
| 422 `validation_failed` | Body failed schema |

---

### `GET /v1/jobs`

List jobs for the authenticated tenant.

**Request**

```
GET /v1/jobs?status=delivered&limit=25
```

**Response — 200 OK**

```json
{
  "data": [
    {
      "id": "job_01HZX5...",
      "url": "https://example.com",
      "status": "delivered",
      "created_at": "2026-06-05T11:18:00Z",
      "deployment_url": "https://example-com-vibe.vercel.app",
      "repo_url": "https://github.com/acme/example-com"
    }
  ],
  "page": { "cursor": null, "next_cursor": null, "has_more": false }
}
```

---

### `GET /v1/jobs/{id}`

Retrieve a single job.

**Response — 200 OK**

```json
{
  "id": "job_01HZX5...",
  "tenant_id": "ten_01HZ...",
  "url": "https://example.com",
  "canonical_url": "https://www.example.com",
  "status": "delivered",
  "spec": {
    "crawl_depth": 2,
    "max_pages": 25,
    "template": "default"
  },
  "metadata": { "external_id": "ext-789" },
  "phases": [
    { "name": "capture",   "status": "ok", "started_at": "...", "ended_at": "...", "cost_usd": 0.21 },
    { "name": "analyze",   "status": "ok", "started_at": "...", "ended_at": "...", "cost_usd": 0.55 },
    { "name": "generate",  "status": "ok", "started_at": "...", "ended_at": "...", "cost_usd": 1.74 },
    { "name": "seo",       "status": "ok", "started_at": "...", "ended_at": "...", "cost_usd": 0.28 },
    { "name": "deploy",    "status": "ok", "started_at": "...", "ended_at": "...", "cost_usd": 0.07 }
  ],
  "deployment": {
    "url": "https://example-com-vibe.vercel.app",
    "provider": "vercel",
    "custom_domain": null
  },
  "repo": {
    "url": "https://github.com/acme/example-com",
    "default_branch": "main"
  },
  "report": {
    "id": "rpt_01HZ...",
    "url": "/v1/reports/rpt_01HZ..."
  },
  "cost_usd": 2.85,
  "created_at": "2026-06-05T11:18:00Z",
  "completed_at": "2026-06-05T11:48:14Z"
}
```

---

### `POST /v1/jobs/{id}/cancel`

Cancel an in-flight job.

**Response — 200 OK**

```json
{ "id": "job_01HZX5...", "status": "cancelled" }
```

**Errors**

- 409 `not_cancellable` if job is past `deploying`.

---

### `POST /v1/jobs/{id}/replay`

Replay a job from a specified phase.

**Request**

```json
{ "from_phase": "generate" }
```

**Response — 202 Accepted**

```json
{
  "id": "job_01HZX5XYZ...",
  "parent_id": "job_01HZX5...",
  "status": "queued",
  "from_phase": "generate"
}
```

---

### `GET /v1/jobs/{id}/events`

Server-Sent Events stream of job state transitions. Useful for the dashboard and partner UIs.

**Request**

```
GET /v1/jobs/job_01HZX5.../events
Accept: text/event-stream
```

**Response**

```
event: job.captured
data: {"id":"job_01HZX5...","at":"2026-06-05T11:25:11Z"}

event: job.analyzed
data: {"id":"job_01HZX5...","at":"2026-06-05T11:28:42Z"}
```

Stream terminates on a terminal status.

---

### `GET /v1/reports/{id}`

Retrieve a report.

**Response — 200 OK**

```json
{
  "id": "rpt_01HZ...",
  "job_id": "job_01HZX5...",
  "summary": "Modernized example.com. Improved Lighthouse Performance from 31 to 97 on mobile.",
  "lighthouse": {
    "before": { "performance": 31, "accessibility": 78, "seo": 73, "best_practices": 70 },
    "after":  { "performance": 97, "accessibility": 96, "seo": 100, "best_practices": 100 }
  },
  "pages": 14,
  "changes": [
    { "category": "metadata", "description": "Added unique titles and descriptions to all 14 pages." },
    { "category": "structured_data", "description": "Added LocalBusiness JSON-LD with hours and phone." },
    { "category": "performance", "description": "Replaced 4.2 MB of unoptimized images with WebP equivalents." }
  ],
  "bundle_url": "https://artifacts.vibe.dev/...signed-url..."
}
```

---

### `POST /v1/capture` (Phase-scoped)

Run only the capture phase.

**Request**

```json
{
  "url": "https://example.com",
  "crawl_depth": 1,
  "max_pages": 10
}
```

**Response — 202 Accepted**

```json
{
  "phase_run_id": "phr_01HZ...",
  "status": "running",
  "_links": { "self": "/v1/phase-runs/phr_01HZ..." }
}
```

On completion, the phase run's status becomes `completed` and an artifact URI is exposed.

---

### `POST /v1/analyze` (Phase-scoped)

Run analysis against a previously captured manifest.

**Request**

```json
{ "capture_manifest_uri": "s3://vibe-artifacts/.../capture_manifest.json" }
```

---

### `POST /v1/generate` (Phase-scoped)

Run generation against a site model.

**Request**

```json
{ "site_model_uri": "s3://...", "template": "default" }
```

---

### `POST /v1/deploy` (Phase-scoped)

Run deployment against a workspace archive.

**Request**

```json
{
  "workspace_archive_uri": "s3://...",
  "repo_visibility": "private",
  "github_org": "acme",
  "vercel_team": "acme-team"
}
```

---

## Webhook Contracts

Vibe delivers events to customer-registered webhook endpoints.

### Headers

```
POST /your/webhook/endpoint HTTP/1.1
Content-Type: application/json
Vibe-Event-Id: evt_01HZ...
Vibe-Event-Type: job.delivered
Vibe-Timestamp: 1717584000
Vibe-Signature: t=1717584000,v1=hex_hmac_sha256(secret, "1717584000.<body>")
```

### Payload

```json
{
  "id": "evt_01HZ...",
  "type": "job.delivered",
  "created_at": "2026-06-05T11:48:14Z",
  "data": {
    "job_id": "job_01HZX5...",
    "url": "https://example.com",
    "deployment_url": "https://example-com-vibe.vercel.app",
    "repo_url": "https://github.com/acme/example-com",
    "report_url": "https://app.vibe.dev/reports/rpt_01HZ..."
  }
}
```

### Signature Verification

Pseudocode:

```
expected = hmac_sha256(secret, f"{timestamp}.{raw_body}")
verify constant-time(expected, provided_signature_v1)
```

Replay protection: reject events whose `Vibe-Timestamp` is more than 5 minutes outside the receiver's clock.

### Retry Policy

- Initial delivery within 1 second of event emission.
- On non-2xx: exponential backoff with jitter, max 24 hours, then dead-letter.
- Dead-lettered events visible in the customer dashboard.

### Event Types

| Type | Triggered When |
|------|----------------|
| `job.created` | A job is queued |
| `job.captured` | Capture phase completes |
| `job.analyzed` | Analysis phase completes |
| `job.generated` | Generation phase completes |
| `job.deployed` | Deployment phase completes |
| `job.delivered` | Delivery phase completes |
| `job.failed` | Job enters a terminal failure state |
| `job.cancelled` | Job is cancelled |

---

## Versioning Policy

- `/v1/` is stable. Additive changes (new fields, new endpoints) do not require a new version.
- Backwards-incompatible changes require `/v2/`.
- A `/v1/` endpoint may be deprecated by setting `Deprecation: true` and `Sunset: <date>`.
- Minimum support window after `Sunset`: 6 months.

A change log of API revisions is published at `https://docs.vibe.dev/api/changelog`.

---

## Idempotency

Every `POST` accepts an `Idempotency-Key` header.

- The platform stores `(key, request_hash, response)` for 24 hours.
- Reuse with the same body returns the original response.
- Reuse with a different body returns 409 `idempotency_conflict`.

---

## Security Notes

- API keys must never be sent as URL parameters; they go in the `Authorization` header.
- Webhook secrets are tenant-scoped and rotated by the customer at will.
- All credential changes invalidate dependent sessions.

See `17-security-model.md`.

---

## Assumptions

- Customers prefer cursor pagination over offset.
- HTTP/2 is the baseline transport.
- TLS is terminated at the edge (ALB or CloudFront).

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| URL-prefixed versioning | Simple, cache-friendly, well-understood. |
| RFC 7807 problem+json | Standardized error shape; better tooling. |
| Cursor-only pagination | Avoids offset pitfalls on large tables. |
| Idempotency on all writes | Required for safe retries from partner systems. |
| Phase-scoped endpoints as separate API | Allows partners to bring their own capture data. |
| SSE for live job updates | Simpler than WebSocket; widely supported. |

---

## Open Questions

- Should we expose GraphQL for the dashboard while keeping REST for partners?
- Should we adopt JSON:API or remain on our custom envelope?
- Should webhooks support OAuth2 client credentials in addition to HMAC?
- Should phase-scoped endpoints be MVP or strictly V3?

---

## Future Enhancements

- TypeScript SDK and Python SDK (V3).
- Async batch endpoints (`POST /v1/jobs/batch`) for bulk submission.
- Streaming generation results (`GET /v1/jobs/{id}/stream`) for very fast feedback in V3.
- Granular per-key scopes (e.g., `jobs.read:tenant-id`).

---

## Cross-References

- Database → `08-database-design.md`
- Security → `17-security-model.md`
- User stories → `04-user-stories.md`
- Functional requirements → `05-functional-requirements.md`
