# 02 — System Architecture

> The authoritative architecture description. Defines the boundaries, responsibilities, and protocols of every component in the platform.

---

## Purpose

This document defines the system architecture of Go Up Level Vibe Coding. It describes the platform at four levels of detail using the C4 model (Context, Container, Component, Code), supplemented with data flow diagrams.

It is the contract that subsequent technical documents (`07-technical-specification.md`, `08-database-design.md`, `09-api-specification.md`, and the engine documents) elaborate.

If an implementation choice in any other document contradicts a constraint in this architecture document, this document wins until updated by a corresponding ADR.

---

## Scope

In scope:

- C4 Level 1 — System Context
- C4 Level 2 — Containers
- C4 Level 3 — Components (per major container)
- C4 Level 4 — Code-level diagrams for the most consequential modules
- Data flow diagrams for the canonical job lifecycle
- Deployment topology
- Failure boundaries and isolation strategy

Out of scope:

- Specific framework versions (see `07-technical-specification.md`)
- Schema definitions (see `08-database-design.md`)
- API endpoint contracts (see `09-api-specification.md`)
- Operational runbooks (see `27-production-deployment.md`)

---

## Architectural Principles

The architecture is constrained by the following principles. They are listed in priority order. When two principles conflict, the higher principle wins.

1. **Autonomous operation is the default.** No component requires a human in the loop to complete normal work.
2. **Failure must be observable and replayable.** Every step produces durable artifacts. Every step can be rerun from its inputs.
3. **Boundaries are explicit and contractual.** Components communicate through documented interfaces, never through shared mutable state.
4. **Stateless workers, stateful storage.** Workers can be killed and restarted. State lives in the database, object store, or queue.
5. **Per-job isolation.** Each job has its own workspace, its own logs, and its own credential scope.
6. **Idempotent operations.** Any operation can be safely retried without producing duplicate side effects.
7. **Versioned interfaces.** All cross-component contracts are versioned. Breaking changes require a major version bump.
8. **One reason to change per component.** Components are organized by responsibility, not by technical layer.

---

## C4 Level 1 — System Context

```mermaid
C4Context
    title System Context — Go Up Level Vibe Coding

    Person(customer, "Customer", "Small business owner, consultant, or agency.")
    Person(operator, "Operator", "Vibe team member running the platform.")
    System_Ext(api_caller, "API Caller", "Partner system that programmatically submits jobs.")

    System(vibe, "Vibe Platform", "Autonomous website modernization platform.")

    System_Ext(source_site, "Source Website", "The customer's existing website being modernized.")
    System_Ext(github, "GitHub", "Version control hosting for delivered code.")
    System_Ext(vercel, "Vercel", "Deployment platform for the modernized site.")
    System_Ext(stripe, "Stripe", "Payments processor.")
    System_Ext(llm, "LLM Provider", "OpenAI, Anthropic, or compatible reasoning backend.")
    System_Ext(email, "Email Provider", "Transactional email for notifications.")
    System_Ext(dns, "DNS Provider", "Optional, for custom domain provisioning.")

    Rel(customer, vibe, "Submits URL, pays, receives report")
    Rel(operator, vibe, "Operates dashboard, reviews failures")
    Rel(api_caller, vibe, "POSTs jobs, receives webhooks")

    Rel(vibe, source_site, "Captures content over HTTP/HTTPS")
    Rel(vibe, github, "Creates repos, pushes code")
    Rel(vibe, vercel, "Creates projects, triggers deploys")
    Rel(vibe, stripe, "Processes payments")
    Rel(vibe, llm, "Sends prompts, receives completions")
    Rel(vibe, email, "Sends transactional email")
    Rel(vibe, dns, "Validates and provisions custom domains")
```

### External Dependencies

| Dependency | Purpose | Failure Mode |
|------------|---------|--------------|
| Source Website | Input | Job aborts with `capture_failed` if unreachable. |
| GitHub | Code delivery | Job pauses with `delivery_blocked` if API rate limit or outage. Retried with exponential backoff. |
| Vercel | Deployment | Job pauses with `deploy_blocked`. Retried. |
| Stripe | Billing | Job creation is blocked at intake. |
| LLM Provider | Reasoning | Engines retry with fallback provider if available. |
| Email Provider | Notifications | Notifications buffered and retried. Does not block job. |
| DNS Provider | Custom domains | Optional. Job delivers with default Vercel subdomain if DNS fails. |

---

## C4 Level 2 — Containers

A "container" here is a runtime-deployable unit (a service, a worker pool, a database, a queue).

```mermaid
C4Container
    title Container View — Vibe Platform

    Person(customer, "Customer")
    Person(operator, "Operator")
    System_Ext(api_caller, "API Caller")

    Container_Boundary(vibe, "Vibe Platform") {
        Container(web, "Web Frontend", "Next.js", "Marketing site, customer dashboard, status page.")
        Container(admin, "Admin Dashboard", "Next.js", "Operator-facing console.")
        Container(api, "Public API", "Python / FastAPI", "Intake, job control, reports, billing webhooks.")
        Container(orch, "Orchestrator", "Python / Temporal", "Schedules engines, manages job state machine.")
        Container(cap_worker, "Capture Workers", "Python + Playwright", "Headless browser fleet.")
        Container(ana_worker, "Analysis Workers", "Python", "Static and AI-driven analysis.")
        Container(gen_worker, "Generation Workers", "Python + Node", "Next.js generation, build, validate.")
        Container(seo_worker, "SEO Workers", "Python", "Metadata, structured data, llms.txt.")
        Container(dep_worker, "Deployment Workers", "Python", "GitHub + Vercel integration.")
        ContainerDb(pg, "PostgreSQL", "AWS RDS", "Jobs, users, billing, reports, audit.")
        ContainerDb(redis, "Redis", "AWS ElastiCache", "Cache, rate limit, ephemeral state.")
        ContainerDb(s3, "Object Store", "AWS S3", "HAR files, screenshots, generated code archives, reports.")
        ContainerDb(queue, "Job Queue", "Temporal / SQS fallback", "Durable workflow execution.")
        Container(obs, "Observability Stack", "OpenTelemetry + Grafana", "Logs, metrics, traces.")
    }

    System_Ext(source_site, "Source Website")
    System_Ext(github, "GitHub")
    System_Ext(vercel, "Vercel")
    System_Ext(stripe, "Stripe")
    System_Ext(llm, "LLM Provider")
    System_Ext(email, "Email Provider")

    Rel(customer, web, "HTTPS")
    Rel(operator, admin, "HTTPS")
    Rel(api_caller, api, "HTTPS / REST")
    Rel(web, api, "HTTPS / REST")
    Rel(admin, api, "HTTPS / REST")
    Rel(api, pg, "TCP/SQL")
    Rel(api, redis, "TCP")
    Rel(api, orch, "gRPC / Temporal client")
    Rel(orch, queue, "Workflow tasks")
    Rel(orch, cap_worker, "Activity dispatch")
    Rel(orch, ana_worker, "Activity dispatch")
    Rel(orch, gen_worker, "Activity dispatch")
    Rel(orch, seo_worker, "Activity dispatch")
    Rel(orch, dep_worker, "Activity dispatch")
    Rel(cap_worker, source_site, "HTTPS")
    Rel(cap_worker, s3, "HTTPS")
    Rel(ana_worker, s3, "HTTPS")
    Rel(ana_worker, llm, "HTTPS")
    Rel(gen_worker, s3, "HTTPS")
    Rel(gen_worker, llm, "HTTPS")
    Rel(seo_worker, llm, "HTTPS")
    Rel(dep_worker, github, "HTTPS")
    Rel(dep_worker, vercel, "HTTPS")
    Rel(api, stripe, "HTTPS")
    Rel(api, email, "HTTPS")
    Rel_Back(obs, web, "OTLP")
    Rel_Back(obs, api, "OTLP")
    Rel_Back(obs, orch, "OTLP")
```

### Container Catalogue

| Container | Runtime | Scaling | Owns |
|-----------|---------|---------|------|
| Web Frontend | Next.js on Vercel | Vercel autoscaling | Marketing, customer dashboard. |
| Admin Dashboard | Next.js on Vercel | Vercel autoscaling | Operator console. Internal auth only. |
| Public API | FastAPI on AWS ECS Fargate | Horizontal, target CPU 60% | Stateless HTTP surface. |
| Orchestrator | Temporal Cluster (self-hosted or Temporal Cloud) | Vertical for the cluster, horizontal for workers | Durable workflow execution. |
| Capture Workers | Python + Playwright on ECS Fargate (or dedicated EC2 for browser pool) | Horizontal, queue-depth-based | Browser automation. |
| Analysis Workers | Python on ECS Fargate | Horizontal, queue-depth-based | Static + AI analysis. |
| Generation Workers | Python + Node on ECS Fargate | Horizontal, queue-depth-based | Code generation, build, validate. |
| SEO Workers | Python on ECS Fargate | Horizontal | Metadata, structured data, llms.txt. |
| Deployment Workers | Python on ECS Fargate | Horizontal | GitHub + Vercel calls. |
| PostgreSQL | AWS RDS Multi-AZ | Vertical, read replicas | Transactional state. |
| Redis | AWS ElastiCache | Cluster mode | Cache, rate limit, locks. |
| Object Store | AWS S3 with versioning | Native | Artifacts. |
| Queue / Workflow | Temporal or SQS fallback | Native | Durable job execution. |
| Observability | Grafana Cloud or self-hosted Loki/Tempo/Prom | Native | Telemetry. |

---

## C4 Level 3 — Components

### Public API

```mermaid
flowchart TB
    subgraph API [Public API]
        AUTH[Auth Middleware]
        RATE[Rate Limiter]
        ROUTE[Router]
        VAL[Request Validators]
        JOBS[Jobs Controller]
        REPORTS[Reports Controller]
        BILLING[Billing Controller]
        WEBHOOKS[Webhooks Controller]
        DB[(PostgreSQL)]
        REDIS[(Redis)]
        WORKFLOW[Temporal Client]
    end

    AUTH --> RATE --> ROUTE
    ROUTE --> VAL
    VAL --> JOBS
    VAL --> REPORTS
    VAL --> BILLING
    VAL --> WEBHOOKS
    JOBS --> DB
    JOBS --> WORKFLOW
    REPORTS --> DB
    BILLING --> DB
    WEBHOOKS --> DB
    JOBS --> REDIS
```

Components:

- **Auth Middleware** — Validates JWT (customer) or API key (partner). Resolves the tenant context.
- **Rate Limiter** — Token-bucket per tenant in Redis. Enforces tier-specific limits (see `17-security-model.md`).
- **Router** — FastAPI routing.
- **Validators** — Pydantic models. All request and response shapes are typed.
- **Jobs Controller** — Creates jobs, exposes status, cancels jobs.
- **Reports Controller** — Returns signed URLs to report bundles.
- **Billing Controller** — Stripe checkout sessions, webhook ingestion, quota updates.
- **Webhooks Controller** — Receives provider callbacks (Stripe, Vercel, GitHub).

### Orchestrator

```mermaid
flowchart TB
    subgraph ORCH [Orchestrator — Temporal Workflows]
        IN[Job Workflow]
        S1[Capture Activity]
        S2[Analyze Activity]
        S3[Generate Activity]
        S4[SEO Activity]
        S5[QualityReview Activity]
        S6[Deploy Activity]
        S7[Deliver Activity]
        FAIL[Failure Handler]
        SAGA[Compensation Saga]
    end
    IN --> S1 --> S2 --> S3 --> S4 --> S5 --> S6 --> S7
    S1 -. on error .-> FAIL
    S2 -. on error .-> FAIL
    S3 -. on error .-> FAIL
    S4 -. on error .-> FAIL
    S5 -. on error .-> FAIL
    S6 -. on error .-> FAIL
    FAIL --> SAGA
```

The orchestrator is a Temporal workflow that owns the lifecycle of one job. Each engine appears as an `Activity`. Activities dispatch to worker pools that run the actual engine code.

Workflow features:

- Per-activity retry policies (configured per engine).
- Per-activity timeouts.
- Saga-style compensation (e.g., delete created GitHub repo if Vercel deploy fails irrecoverably).
- Versioned workflow definitions to support safe rollouts.

### Capture Worker

```mermaid
flowchart TB
    subgraph CAP [Capture Worker]
        IN[Capture Request]
        PRE[URL Validation]
        BROWSER[Browser Pool]
        HAR[HAR Recorder]
        SHOT[Screenshot Engine]
        CRAWL[Link Crawler]
        ASSET[Asset Downloader]
        PACK[Archive Packer]
        OUT[Capture Manifest]
    end
    IN --> PRE --> BROWSER
    BROWSER --> HAR
    BROWSER --> SHOT
    BROWSER --> CRAWL
    CRAWL --> ASSET
    HAR --> PACK
    SHOT --> PACK
    ASSET --> PACK
    PACK --> OUT
```

See `10-capture-engine.md` for full detail.

### Analysis Worker

```mermaid
flowchart TB
    subgraph ANA [Analysis Worker]
        IN[Capture Manifest]
        PARSE[HTML / CSS Parser]
        INV[Page Inventory Builder]
        CONTENT[Content Extractor]
        NAV[Navigation Graph]
        SEO[SEO Scanner]
        A11Y[Accessibility Scanner]
        LLM[LLM Summarizer]
        MODEL[SiteModel Composer]
        OUT[SiteModel JSON]
    end
    IN --> PARSE
    PARSE --> INV
    PARSE --> CONTENT
    PARSE --> NAV
    PARSE --> SEO
    PARSE --> A11Y
    CONTENT --> LLM
    INV --> MODEL
    CONTENT --> MODEL
    NAV --> MODEL
    SEO --> MODEL
    A11Y --> MODEL
    LLM --> MODEL
    MODEL --> OUT
```

See `11-analysis-engine.md` for full detail.

### Generation Worker

```mermaid
flowchart TB
    subgraph GEN [Generation Worker]
        IN[SiteModel]
        SCAFFOLD[Scaffold App]
        ROUTES[Route Generator]
        COMP[Component Generator]
        STYLE[Tailwind Theme]
        DATA[Content Data Layer]
        ASSET[Asset Pipeline]
        BUILD[Next Build]
        TEST[Self-Test Runner]
        OUT[Generated Workspace]
    end
    IN --> SCAFFOLD
    SCAFFOLD --> ROUTES
    SCAFFOLD --> COMP
    SCAFFOLD --> STYLE
    SCAFFOLD --> DATA
    SCAFFOLD --> ASSET
    ROUTES --> BUILD
    COMP --> BUILD
    STYLE --> BUILD
    DATA --> BUILD
    ASSET --> BUILD
    BUILD --> TEST
    TEST --> OUT
```

See `12-generation-engine.md` for full detail.

### Deployment Worker

```mermaid
flowchart TB
    subgraph DEP [Deployment Worker]
        IN[Generated Workspace]
        REPO[GitHub Repo Provisioner]
        PUSH[Code Push]
        VRC[Vercel Project Provisioner]
        TRIG[Trigger Deploy]
        VER[Verify Deploy]
        REPORT[Deployment Report]
    end
    IN --> REPO --> PUSH --> VRC --> TRIG --> VER --> REPORT
```

See `14-deployment-engine.md`, `15-github-integration.md`, `16-vercel-integration.md`.

---

## Data Flow — Canonical Job Lifecycle

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant API as Public API
    participant DB as PostgreSQL
    participant O as Orchestrator
    participant C as Capture
    participant A as Analysis
    participant G as Generation
    participant S as SEO
    participant D as Deployment
    participant GH as GitHub
    participant V as Vercel
    participant E as Email

    U->>API: POST /v1/jobs { url }
    API->>DB: INSERT Job (status=queued)
    API->>O: StartWorkflow(job_id)
    API-->>U: 202 Accepted { job_id }

    O->>C: CaptureActivity(url)
    C->>C: Browser, HAR, screenshots, crawl
    C-->>O: capture_manifest_url
    O->>DB: UPDATE Job (status=captured)

    O->>A: AnalyzeActivity(capture_manifest_url)
    A->>A: Parse, scan, LLM summarize
    A-->>O: site_model_url
    O->>DB: UPDATE Job (status=analyzed)

    O->>G: GenerateActivity(site_model_url)
    G->>G: Scaffold, generate, build, test
    G-->>O: workspace_url
    O->>DB: UPDATE Job (status=generated)

    O->>S: SeoActivity(workspace_url)
    S-->>O: workspace_url (enriched)
    O->>DB: UPDATE Job (status=seo_enriched)

    O->>D: DeployActivity(workspace_url)
    D->>GH: Create repo, push code
    D->>V: Create project, trigger deploy
    V-->>D: deployment_url
    D-->>O: { repo_url, deployment_url }
    O->>DB: UPDATE Job (status=deployed)

    O->>E: Send delivery email
    O->>DB: UPDATE Job (status=delivered)
    E-->>U: Email with links
```

---

## Failure Boundaries

A failure boundary is a place where a fault cannot cross without a deliberate handoff.

| Boundary | Why It Exists |
|----------|---------------|
| Per-job workspace | Isolates corrupted or malicious source content. |
| Per-worker container | Browser crashes do not take down the platform. |
| Per-tenant credential scope | A leak in one tenant does not expose others. |
| Per-engine activity | A retry in one engine does not redo prior engines. |
| Per-deployment GitHub App installation | A token revocation by one customer does not affect others. |

---

## Deployment Topology

```mermaid
flowchart LR
    subgraph EDGE [Edge / CDN]
        VERCEL[Vercel Edge]
    end
    subgraph AWS [AWS Region us-east-1]
        ALB[Application Load Balancer]
        subgraph ECS [ECS Fargate Cluster]
            APIPOD[API Tasks]
            ORCHPOD[Orchestrator Workers]
            CAPPOD[Capture Workers]
            ANAPOD[Analysis Workers]
            GENPOD[Generation Workers]
            SEOPOD[SEO Workers]
            DEPPOD[Deployment Workers]
        end
        RDS[(RDS Postgres Multi-AZ)]
        ELASTIC[(ElastiCache Redis)]
        S3[(S3 Buckets)]
        SECRETS[Secrets Manager]
        TEMPORAL[Temporal Cluster]
    end
    USER[User] --> VERCEL
    VERCEL --> ALB
    ALB --> APIPOD
    APIPOD --> RDS
    APIPOD --> ELASTIC
    APIPOD --> TEMPORAL
    APIPOD --> S3
    APIPOD --> SECRETS
    ORCHPOD --> TEMPORAL
    CAPPOD --> S3
    ANAPOD --> S3
    GENPOD --> S3
    DEPPOD --> S3
```

Notes:

- The web frontend runs on Vercel for SEO and performance benefits.
- All backend services run on AWS in a single primary region with cross-AZ redundancy.
- A V3 milestone introduces multi-region for capture latency and DR.

---

## Cross-Cutting Concerns

### Configuration

- All configuration is environment-variable driven.
- Secrets live in AWS Secrets Manager and are mounted to ECS tasks at boot.
- Application config is versioned in a `config/` directory in the monorepo and validated by a Pydantic schema at boot.

### Observability

- All services emit OpenTelemetry traces, metrics, and structured logs.
- Trace context propagates through Temporal workflows via custom interceptors.
- See `18-observability.md`.

### Security

- TLS in transit, KMS-encrypted at rest.
- Customer artifacts are partitioned by tenant prefix in S3.
- Workers run with the least privilege IAM role that satisfies their function.
- See `17-security-model.md`.

### Versioning

- All cross-component contracts (REST APIs, Temporal workflow signatures, S3 artifact schemas) are versioned.
- A deprecation policy applies to API v1 surfaces from V2 onward.

---

## Assumptions

- AWS is the primary cloud. The architecture is portable to GCP but not designed for multi-cloud day one.
- Temporal is acceptable as a managed dependency. If not, the orchestrator falls back to SQS + a custom state machine (see ADR-005).
- A single region is acceptable for MVP latency and reliability targets.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| C4 model for documentation | Industry-standard, agent-readable, scales. |
| Mermaid diagrams | Source-controllable, diff-able. |
| Temporal for orchestration | Durable state, retries, versioning out of the box. |
| Per-job workspace | Failure isolation, security, replayability. |
| Stateless workers | Horizontal scaling, no in-process state. |
| Object storage for all artifacts | Cheap, durable, addressable. |

---

## Open Questions

- Should we run our own Temporal cluster or use Temporal Cloud for V1?
- Is RDS Postgres the right default versus Aurora Serverless v2?
- At what scale does the browser pool warrant a dedicated EC2 fleet versus Fargate?
- Do we need a separate "preview" environment per pull request, or is staging sufficient?

---

## Future Enhancements

- Multi-region active-active for capture workers (lower latency to source sites).
- A dedicated browser pool service (something like Browserless) extracted out of capture workers.
- A managed `SiteModel` schema registry (e.g., Buf Schema Registry) for cross-language consumers.
- An event bus (Kafka or Kinesis) for downstream analytics consumers.

---

## Cross-References

- Technical choices → `07-technical-specification.md`
- Per-engine deep dives → `10-capture-engine.md` through `14-deployment-engine.md`
- Database schema → `08-database-design.md`
- API surface → `09-api-specification.md`
- ADRs → `ADR/`
