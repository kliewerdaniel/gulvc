# 18 — Observability

> The instrumentation, signals, dashboards, and alerts that make the platform debuggable and operable.

---

## Purpose

This document defines the observability strategy of Vibe. It specifies what signals every component emits, how those signals are collected, where they are stored, what dashboards expose them, and what alerts page humans.

The principle is simple: if it cannot be observed, it does not exist. Every production behavior must be visible to the team without SSH'ing into a container.

---

## Scope

In scope:

- Telemetry signal model (logs, metrics, traces)
- Instrumentation conventions per language and component
- Storage and query backends
- Dashboards (catalogued)
- SLOs and alerts
- On-call and runbook structure
- Cost-of-observability budget

Out of scope:

- Security audit logging (`17-security-model.md`)
- Customer-facing status page beyond high-level description (`27-production-deployment.md`)

---

## Pillars

1. **OpenTelemetry first.** All instrumentation flows through OTel SDKs. Vendor lock-in is rejected by construction.
2. **Structured everywhere.** Logs are JSON; metrics are typed; traces include semantic attributes.
3. **Correlation by `trace_id` and `job_id`.** Every event in the system can be joined.
4. **Cost discipline.** Sampling, cardinality control, retention windows.
5. **Actionable alerts.** Every alert links to a runbook.
6. **SLO-driven, not vanity-driven.** Dashboards exist to support SLOs, not the other way around.

---

## Signal Model

### Traces

- All services emit OpenTelemetry spans.
- Root spans for inbound HTTP requests and inbound workflow signals.
- Internal spans for engine phases, agent runs, tool calls, LLM calls, database queries.
- Trace context propagated through Temporal workflows via custom interceptors.
- Trace context propagated through HTTP via W3C `traceparent`.

Span naming convention: `<component>.<operation>` (lowercase, dotted).

Example: `capture.page`, `analysis.run`, `generation.self_repair`, `vercel.create_deployment`.

### Logs

- Structured JSON only. No plaintext log lines in production.
- One log emitter per service; uses `structlog` (Python) or `pino` (Node).
- Mandatory fields:
  - `time` (RFC 3339)
  - `level` (`debug`, `info`, `warn`, `error`)
  - `service`
  - `env`
  - `trace_id`, `span_id` (when in a span)
  - `job_id`, `tenant_id` (when available)
  - `message`
- Customer PII never logged in plaintext.
- Secrets never logged.

### Metrics

- Counters, histograms, gauges per OTel conventions.
- Metric names: `vibe.<component>.<measurement>` (lowercase, dotted).
- Units suffixed: `_ms`, `_bytes`, `_total`, `_usd`.
- Cardinality bounded:
  - No raw URLs in labels.
  - No user-controlled strings as labels.
  - Tenant ID label allowed but capped at top-N tenants; rest aggregated.

### Events (Domain)

- Domain events (`job.captured`, etc.) emitted by the orchestrator are first-class observability signals.
- Captured in the `events` table and exported to the metrics backend via the outbox.

---

## Backends

| Signal | Backend | Notes |
|--------|---------|-------|
| Traces | Grafana Tempo (self-hosted) or Tempo Cloud | OTLP gRPC |
| Metrics | Prometheus + Mimir | OTel SDK → Prom remote-write or OTLP |
| Logs | Loki | JSON logs ingested via OpenTelemetry Collector |
| Dashboards | Grafana | All dashboards as code (Grafonnet) |
| Error tracking | Sentry | For exceptions in API, frontend, workers |
| Alerting | Grafana Alerting → PagerDuty | Slack as secondary channel |
| Uptime | Better Uptime or Checkly | External vantage points |
| Real-user metrics | Vercel Analytics (Vibe frontend); opt-in script for delivered sites |

OpenTelemetry Collector runs as a DaemonSet (or sidecar in ECS) per cluster. It scrubs PII, applies sampling, and forwards to the backends.

---

## Instrumentation Conventions

### Python Services

```python
from opentelemetry import trace
from opentelemetry.metrics import get_meter
import structlog

tracer = trace.get_tracer("apps.capture")
meter = get_meter("apps.capture")
logger = structlog.get_logger("apps.capture")

page_counter = meter.create_counter("vibe.capture.pages_total")
page_latency = meter.create_histogram("vibe.capture.page_latency_ms")

async def capture_page(url: str, ctx: JobContext) -> CapturedPage:
    with tracer.start_as_current_span("capture.page") as span:
        span.set_attribute("url", url)
        span.set_attribute("job_id", str(ctx.job_id))
        span.set_attribute("tenant_id", str(ctx.tenant_id))
        start = monotonic()
        try:
            result = await _capture(url)
            page_counter.add(1, {"status": "ok"})
            return result
        except Exception as e:
            page_counter.add(1, {"status": "error"})
            span.record_exception(e)
            span.set_status(Status(StatusCode.ERROR))
            logger.error("capture.page.failed", url=url, exc_info=True)
            raise
        finally:
            page_latency.record((monotonic() - start) * 1000)
```

Conventions:

- Every public function emits a span.
- Every exception path records the exception on the span and emits an error log.
- LLM calls record `gen_ai.system`, `gen_ai.request.model`, `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`, `gen_ai.response.id` per the OTel GenAI conventions.

### TypeScript Services

```ts
import { trace, metrics } from "@opentelemetry/api";
import pino from "pino";

const tracer = trace.getTracer("apps.web");
const meter = metrics.getMeter("apps.web");
const logger = pino({ formatters: { level(label) { return { level: label }; } } });

const requestCounter = meter.createCounter("vibe.web.requests_total");
```

Same conventions apply.

### Temporal Workflows

- A custom interceptor injects `trace_id` and `job_id` into every workflow and activity context.
- Activities are instrumented as standard Python services.
- Workflow-level events (start, completion, failure) emit spans named `workflow.<workflow>.<event>`.

---

## Job-Level Trace

A complete job produces one trace spanning capture, analysis, generation, SEO, quality review, deployment, and delivery. The trace has the following shape:

```
job.run                (root, by orchestrator)
├── capture.run
│   ├── capture.page (×N)
│   │   ├── browser.navigate
│   │   ├── browser.har
│   │   └── browser.screenshot
│   └── capture.upload
├── analysis.run
│   ├── analysis.parse (×N)
│   ├── analysis.llm.summarize (×K)
│   └── analysis.compose
├── generation.run
│   ├── generation.scaffold
│   ├── generation.routes
│   ├── generation.build
│   └── generation.self_repair (×0..3)
├── seo.run
├── quality.review
└── deployment.run
    ├── github.create_repo
    ├── github.push
    ├── vercel.create_project
    └── vercel.deployment
```

Operators can navigate this trace from the admin console.

---

## Dashboards (Catalog)

| Dashboard | Audience | Purpose |
|-----------|----------|---------|
| Platform Overview | Everyone | SLO summary, job throughput, error rate, COGS |
| Job Funnel | Product, Eng | Conversion through capture → delivery |
| Capture Health | Eng | Browser pool utilization, capture latency, bot-wall rate |
| Analysis Health | Eng | LLM cost per analysis, model distribution, token usage |
| Generation Health | Eng | Build success rate, self-repair distribution, build duration |
| Deployment Health | Eng | GitHub + Vercel API latency, deploy failures, compensations |
| API Health | Eng, SRE | Latency, error rate, rate-limit rejections |
| Database Health | Eng, SRE | Connections, slow queries, locks |
| LLM Cost | Eng, Finance | Spend per provider, per model, per job |
| Per-Tenant View | Support, Ops | One tenant's jobs, recent failures, recent spend |

All dashboards are committed under `infra/grafana/dashboards/` as JSON and applied via CI.

---

## SLOs

The platform defines explicit Service Level Objectives. SLO breaches drive alerting.

### SLO Catalog

| SLO | Target | Window | Owner |
|-----|--------|--------|-------|
| Public API availability | 99.5% (MVP) | 30 days rolling | API |
| Public API latency p95 ≤ 300 ms | 99% of 1-min windows | 30 days rolling | API |
| Free preview latency p95 ≤ 30 s | 95% of 5-min windows | 7 days rolling | API + Capture |
| Job completion rate | 95% (MVP) | 7 days rolling | Orchestrator |
| Median job duration ≤ 45 min | 95% of jobs | 7 days rolling | Orchestrator |
| Lighthouse Performance ≥ 90 (median) | 90% of jobs | 7 days rolling | Generation |
| Deployment success | 98% | 7 days rolling | Deployment |
| Webhook delivery success | 99% within 24 h | 7 days rolling | Delivery |

Each SLO has:

- An error budget calculated from the target.
- Burn rate alerts at 2× and 14× rates.
- A runbook for typical causes and mitigations.

---

## Alerts

Alerts are routed through Grafana Alerting → PagerDuty (SEV1/SEV2) and Slack (SEV3, informational).

### Tiered Alert Rules

| Severity | Examples |
|----------|----------|
| SEV1 (page) | API availability < 99% over 10 min; database unreachable; cross-tenant data exposure detected; LLM cost burn > 200% of daily budget |
| SEV2 (page during business hours) | API p95 > 1 s for 15 min; job completion rate < 80% over 1 h; deployment failure rate > 20% over 1 h |
| SEV3 (Slack) | LLM provider failover triggered; rate-limit denials elevated; backup snapshot delayed |

Every alert includes:

- Title
- Severity
- Link to the dashboard
- Link to the relevant runbook (`apps/runbooks/`)
- The owner team

---

## On-Call

- Primary on-call: rotated weekly across engineering.
- Secondary on-call: rotated weekly.
- Hand-off: 11:00 local Monday via the on-call channel; checklist signed.
- Out-of-hours coverage: required for SEV1; business hours otherwise.
- Compensation: per company policy; documented in `26-local-development.md`'s contributor onboarding (or org handbook).

---

## Cost of Observability

The observability stack itself has a budget.

| Component | Monthly budget (MVP) | Mitigations |
|-----------|----------------------|-------------|
| Logs (Loki) | $400 | Drop debug in prod; sample 50% of `info`; retain 14 days |
| Traces (Tempo) | $300 | Head-sample 10%; tail-keep errors and slow traces |
| Metrics (Mimir) | $300 | Cardinality limits; aggregate by hour after 7 days |
| Dashboards (Grafana) | $50 | Single workspace |
| Error tracking (Sentry) | $100 | Sampling for high-volume errors |
| Total | $1,150/month | Roughly 50% of fixed infra at MVP |

This budget is part of the COGS analysis (`28-cost-model.md`).

---

## Runbooks

Runbooks live in `apps/runbooks/`. Each runbook contains:

- Title and summary
- When to use
- Diagnostic queries (Loki / Prometheus / Postgres)
- Mitigation steps
- Rollback steps
- Postmortem template link

Mandatory runbooks (MVP):

- `api-down`
- `database-unreachable`
- `capture-pool-saturated`
- `vercel-deploy-failures`
- `github-rate-limit`
- `llm-provider-outage`
- `cost-burn`
- `cross-tenant-leak-suspected`

---

## Audit Observability

The `audit_log` table is observable in its own right:

- All admin actions visible in the Audit dashboard.
- Anomaly detection on suspicious patterns (e.g., bulk artifact downloads).
- Per-actor query supported.

---

## Customer-Facing Observability

- Public status page at `status.vibe.dev` shows component health, ongoing incidents, and historical uptime.
- The customer dashboard exposes a per-job timeline view (powered by the same trace data).
- The customer dashboard exposes per-month usage and a downloadable report.

---

## Testing the Observability

- "Observability tests" run in CI: synthetic operations verify that traces, metrics, and logs reach the backends with expected shapes.
- A "trace integrity" check runs nightly: random recent jobs are inspected to ensure their full traces exist and are well-formed.

---

## Assumptions

- The team can self-host Grafana Tempo, Loki, Mimir at MVP scale; or move to Grafana Cloud if pricing is favorable.
- OpenTelemetry SDK maturity is sufficient for production use across Python and TypeScript.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| OpenTelemetry as the only API | Vendor portability. |
| `trace_id` + `job_id` as universal correlators | Joins everything. |
| Dashboards as code | Reviewable, reproducible. |
| Alerts must link to runbooks | Reduces MTTR. |
| Cardinality discipline | Keeps costs sane. |

---

## Open Questions

- Should we adopt eBPF-based profiling (Pyroscope) at V2?
- Should we expose customer-readable trace timelines, or aggregate views only?
- Should we publish historical SLO reports as a trust artifact?

---

## Future Enhancements

- Anomaly detection on per-tenant cost spikes.
- Auto-generated incident retrospectives from traces and chat logs.
- Per-customer dashboards in V3 (white-label-branded).

---

## Cross-References

- Security audit logging → `17-security-model.md`
- Production ops → `27-production-deployment.md`
- Testing → `19-testing-strategy.md`
- Cost → `28-cost-model.md`
