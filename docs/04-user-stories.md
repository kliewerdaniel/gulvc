# 04 — User Stories

> The narrative bridge between the PRD and the engineering backlog. Tells the story of every interaction the platform must support.

---

## Purpose

This document captures the user stories that the platform must satisfy. It complements:

- The PRD (`01`), which defines what the product is.
- The functional requirements (`05`), which define what the system must do.
- The backlog (`24`), which decomposes stories into implementable engineering tasks.

This document is the place where personas, behaviors, and acceptance expectations live in the language of the user.

---

## Scope

In scope:

- Stories for all V0/V1 (MVP), V2, and V3 surfaces
- Stories for every persona defined in the PRD
- Stories for the autonomous-caller persona (V3)
- Stories grouped by epic

Out of scope:

- Engineering task decomposition (see `24-backlog.md`)
- Acceptance test cases at the system level (see `19-testing-strategy.md`)

---

## Story Format

Each story follows the canonical form:

> **As** a [persona]
> **I want** [capability]
> **So that** [outcome]
>
> **Acceptance Criteria:**
> - Criterion 1
> - Criterion 2

Stories are identified by `US-<epic>-<n>` where epic is a short code (e.g., `INTAKE`, `CAPTURE`).

---

## Epic Index

| Code | Epic | Volume |
|------|------|--------|
| `INTAKE` | Customer intake and onboarding | High |
| `BILLING` | Payments, quota, subscriptions | High |
| `CAPTURE` | Source-site capture | High |
| `ANALYSIS` | Site analysis and modeling | High |
| `GEN` | Code generation | High |
| `SEO` | SEO and AI discoverability | High |
| `DEPLOY` | Deployment to GitHub + Vercel | High |
| `REPORT` | Report generation and delivery | Medium |
| `DASH` | Customer dashboard | Medium |
| `ADMIN` | Operator console | Medium |
| `WHITELABEL` | Agency / partner features | Medium |
| `API` | Public API for partners | Medium |
| `MAINTAIN` | Continuous improvement subscription | Medium |
| `SUPPORT` | Customer support and disputes | Low |

---

## Epic: INTAKE

### US-INTAKE-001 — Submit a URL for free preview

> As Theresa, a small-business owner,
> I want to enter my website URL and see what would change,
> So that I can decide whether modernization is worth paying for.

**Acceptance Criteria:**
- Public homepage has a single visible URL input.
- Submitting a valid URL returns a preview within 30 seconds.
- Preview includes: current Lighthouse score, a one-paragraph diagnosis, a thumbnail screenshot, and an estimated projected score.
- Invalid URLs are rejected with a clear message.

### US-INTAKE-002 — See preview without creating an account

> As any prospective customer,
> I want to view my preview before committing to anything,
> So that I do not have to give an email to evaluate the product.

**Acceptance Criteria:**
- No account required to view a free preview.
- An email is requested only at purchase time.

### US-INTAKE-003 — Capture a URL that requires `www`

> As a customer,
> I want the platform to figure out the canonical URL,
> So that I do not have to guess the correct form.

**Acceptance Criteria:**
- The platform attempts `https://`, `https://www.`, and `http://` variants in order.
- The first reachable variant is used.
- The resolved canonical URL is shown to the user.

### US-INTAKE-004 — Reject URLs that violate policy

> As an operator,
> I want the platform to refuse jobs for prohibited domains,
> So that we avoid legal and abuse risk.

**Acceptance Criteria:**
- A configurable denylist of domains is enforced at intake.
- Categories of prohibited content (adult, gambling, sanctioned regimes) are rejected with a generic message.
- Rejections are logged for audit.

### US-INTAKE-005 — Receive a clear error for unreachable sites

> As a customer,
> I want to know if my site is down,
> So that I do not blame the platform.

**Acceptance Criteria:**
- DNS failures, 5xx responses, and timeouts produce distinct, customer-friendly messages.
- The message suggests next steps (check DNS, contact host).

### US-INTAKE-006 — Bulk CSV intake (V2)

> As Marcus, a consultant,
> I want to upload a CSV of multiple URLs at once,
> So that I can modernize many client sites in one operation.

**Acceptance Criteria:**
- CSV accepts up to 500 rows per upload.
- Each row may include `url`, `client_id`, `notes`.
- The system schedules one job per row.
- A bulk-progress view is available.

---

## Epic: BILLING

### US-BILLING-001 — Pay for a single modernization

> As Theresa,
> I want to pay once for my site upgrade,
> So that I am not subscribed to anything.

**Acceptance Criteria:**
- Stripe checkout supports cards, Apple Pay, Google Pay.
- After successful payment, the job is queued automatically.
- A receipt is emailed.

### US-BILLING-002 — Subscribe to continuous improvement

> As Theresa,
> I want to pay monthly to keep my site current,
> So that I never have to think about it again.

**Acceptance Criteria:**
- Subscription is offered at delivery time.
- Cancel-anytime is honored without prorated refunds (initially).
- Subscription state is reflected in the customer dashboard.

### US-BILLING-003 — Receive a tax-compliant invoice

> As an EU customer,
> I want a VAT-compliant invoice,
> So that I can claim it as a business expense.

**Acceptance Criteria:**
- Stripe Tax is enabled.
- Invoices are downloadable from the dashboard.
- VAT IDs may be entered at checkout.

### US-BILLING-004 — Enforce per-tenant quota

> As an operator,
> I want the platform to refuse work above a tenant's quota,
> So that abuse cannot saturate our COGS.

**Acceptance Criteria:**
- Quota is checked synchronously at intake.
- A clear error is returned when quota is exhausted.
- Quota resets per the tier's policy.

### US-BILLING-005 — Dispute a failed job

> As Theresa,
> I want my money back if the platform fails,
> So that I am not financially harmed by a defect.

**Acceptance Criteria:**
- Failed jobs trigger automatic full refund within 24 hours.
- The refund is reflected in Stripe.
- The customer is notified.

### US-BILLING-006 — Upgrade tier mid-cycle

> As Marcus,
> I want to upgrade my agency tier,
> So that I can ingest more clients without waiting.

**Acceptance Criteria:**
- Upgrades take effect immediately.
- Prorated charges are calculated and shown before confirmation.

---

## Epic: CAPTURE

### US-CAPTURE-001 — Capture all pages reachable within depth N

> As the platform,
> I want to crawl internal links to a configurable depth,
> So that I capture the meaningful surface of the site.

**Acceptance Criteria:**
- Default depth is 2, configurable per tier.
- Crawl respects `robots.txt`.
- Up to `max_pages` pages are captured.

### US-CAPTURE-002 — Record network traffic as HAR

> As the platform,
> I want to record HAR for each page,
> So that downstream agents can analyze requests and assets.

**Acceptance Criteria:**
- One HAR file per page.
- HAR includes request and response headers, bodies for HTML/CSS/JS.
- Binary asset bodies are excluded for size but referenced by URL.

### US-CAPTURE-003 — Take desktop and mobile screenshots

> As the platform,
> I want both desktop and mobile screenshots per page,
> So that analysis can compare responsive behavior.

**Acceptance Criteria:**
- 1440×900 (desktop) and 390×844 (mobile) viewports.
- Full-page screenshots, not just above-the-fold.
- PNG format, optimized.

### US-CAPTURE-004 — Detect and handle JavaScript-rendered content

> As the platform,
> I want to capture content rendered by JavaScript,
> So that SPAs are correctly represented.

**Acceptance Criteria:**
- The browser waits for network idle or up to 10 seconds before capture.
- Hydration-dependent text is captured.
- A warning is emitted for sites that exceed the wait threshold.

### US-CAPTURE-005 — Respect robots.txt

> As the platform,
> I want to respect site policies,
> So that we operate as a good citizen.

**Acceptance Criteria:**
- Disallowed paths are skipped.
- If the root is disallowed, the job aborts with a clear reason.

### US-CAPTURE-006 — Identify a bot wall

> As the platform,
> I want to detect captchas or anti-bot pages,
> So that we do not pretend to have captured the real site.

**Acceptance Criteria:**
- Common bot-wall signatures (Cloudflare challenge, hCaptcha, reCAPTCHA) are detected.
- Job aborts with `capture_blocked_bot_wall`.
- The operator dashboard surfaces these for review.

### US-CAPTURE-007 — Capture multi-language sites

> As a customer with a bilingual site,
> I want both languages captured,
> So that my new site preserves both audiences.

**Acceptance Criteria:**
- The crawler follows `hreflang` and language-prefixed paths.
- Each language is recorded in the manifest with its locale.

### US-CAPTURE-008 — Capture site with authentication (V3)

> As a customer with a member portal,
> I want the platform to log in and capture protected pages,
> So that the full site is modernized.

**Acceptance Criteria:**
- Credentials are stored encrypted, used once, and shredded.
- Session cookies are not exported in the deliverable.
- The crawler stops at logout boundaries.

---

## Epic: ANALYSIS

### US-ANALYSIS-001 — Produce a page inventory

> As the platform,
> I want a structured list of every captured page with its role,
> So that generation can reason about page types.

**Acceptance Criteria:**
- Each page has a URL, a role (home, services, contact, blog post, etc.), and a title.
- Roles are inferred from URL, title, and content.

### US-ANALYSIS-002 — Build a navigation graph

> As the platform,
> I want to know how pages link to each other,
> So that generated navigation is faithful.

**Acceptance Criteria:**
- Primary nav, footer nav, and in-content links are distinguished.
- Cycles are detected and represented.

### US-ANALYSIS-003 — Extract content with structure preserved

> As the platform,
> I want to preserve the heading hierarchy and content order,
> So that meaning is not lost in regeneration.

**Acceptance Criteria:**
- `h1`–`h6` are preserved as a tree.
- Paragraphs, lists, and tables are preserved as structured blocks.
- Image positions are anchored to their surrounding text.

### US-ANALYSIS-004 — Audit existing SEO posture

> As the platform,
> I want to know what SEO problems exist on the source site,
> So that I can quantify the improvement we will deliver.

**Acceptance Criteria:**
- Each page has a list of SEO findings (missing title, missing meta description, missing OG tags, no canonical, etc.).
- Findings are scored by severity.

### US-ANALYSIS-005 — Audit accessibility posture

> As the platform,
> I want to know the accessibility issues on the source site,
> So that I can fix them in regeneration.

**Acceptance Criteria:**
- axe-core rules are run on captured HTML.
- Each finding records the rule, severity, and offending selector.

### US-ANALYSIS-006 — Classify the site's industry

> As the platform,
> I want to know the site is for a dentist, restaurant, or law firm,
> So that I can pick the right template and structured-data type.

**Acceptance Criteria:**
- An industry label is produced with a confidence score.
- Low-confidence cases fall back to a generic template.

### US-ANALYSIS-007 — Detect contact methods

> As the platform,
> I want to find phone, email, address, and hours,
> So that the new site can present them prominently.

**Acceptance Criteria:**
- Phones in E.164 format are extracted.
- Emails are extracted.
- Postal addresses are extracted and geocoded (optional).
- Business hours are extracted from common formats.

---

## Epic: GEN

### US-GEN-001 — Generate a buildable Next.js app

> As the platform,
> I want every job to produce code that `next build` succeeds on,
> So that delivered code is not broken.

**Acceptance Criteria:**
- `pnpm install && pnpm next build` succeeds.
- No TypeScript errors.
- Lint passes.

### US-GEN-002 — Generate one route per source page

> As the platform,
> I want a 1:1 mapping between source pages and routes by default,
> So that link integrity is preserved.

**Acceptance Criteria:**
- Source URL paths are preserved.
- Where source uses query strings for routing, paths are normalized.

### US-GEN-003 — Generate semantic, accessible components

> As the platform,
> I want generated markup to use semantic HTML,
> So that accessibility is built in.

**Acceptance Criteria:**
- Headings, landmarks (`header`, `nav`, `main`, `footer`), and lists are used correctly.
- All interactive elements have accessible names.
- Color contrast meets WCAG AA.

### US-GEN-004 — Apply a responsive Tailwind theme

> As the platform,
> I want consistent responsive styling,
> So that the result works on phones and desktops.

**Acceptance Criteria:**
- Tailwind config encodes a theme derived from the source site's palette where possible.
- All routes pass a mobile-viewport smoke test.

### US-GEN-005 — Self-repair build failures

> As the platform,
> I want generation to fix its own broken code,
> So that humans are not pulled in for trivial errors.

**Acceptance Criteria:**
- On build error, the generation agent retries with the error message as context.
- Up to 3 self-repair attempts per build.

### US-GEN-006 — Optimize images

> As the platform,
> I want generated sites to serve optimized images,
> So that performance scores are high.

**Acceptance Criteria:**
- Images are converted to WebP/AVIF where supported.
- Next.js `<Image>` is used with explicit dimensions.

### US-GEN-007 — Preserve forms

> As a customer with a contact form,
> I want the new site to have a working contact form,
> So that I do not lose lead capture.

**Acceptance Criteria:**
- Forms submit to a configurable endpoint (Formspree, the customer's existing endpoint, or a Vibe-managed default).
- Validation is preserved.

---

## Epic: SEO

### US-SEO-001 — Generate per-page metadata

> As the platform,
> I want every page to have unique title and description,
> So that the site indexes well.

**Acceptance Criteria:**
- Titles are unique and concise.
- Descriptions are 50–160 characters.
- Descriptions are derived from page content, not boilerplate.

### US-SEO-002 — Emit JSON-LD structured data

> As the platform,
> I want appropriate Schema.org types per page,
> So that rich results are eligible.

**Acceptance Criteria:**
- Local businesses get `LocalBusiness` schema with NAP.
- Articles get `Article` schema.
- Products get `Product` schema where detected.
- All JSON-LD validates.

### US-SEO-003 — Generate a sitemap

> As the platform,
> I want a `sitemap.xml`,
> So that search engines crawl efficiently.

**Acceptance Criteria:**
- `sitemap.xml` is generated and referenced in `robots.txt`.
- Up to 50,000 URLs per sitemap; split if larger.

### US-SEO-004 — Generate `llms.txt`

> As the platform,
> I want AI assistants to discover and understand the site,
> So that the site is visible in the AI era.

**Acceptance Criteria:**
- `/llms.txt` is served at the site root.
- It contains a business summary and a list of meaningful pages with descriptions.
- It complies with the emerging llms.txt convention.

### US-SEO-005 — Generate Open Graph and Twitter cards

> As a customer,
> I want my site to look good when shared on social media,
> So that link previews drive traffic.

**Acceptance Criteria:**
- Per-page OG image, title, description.
- Twitter card metadata included.

### US-SEO-006 — Generate `robots.txt`

> As the platform,
> I want a sensible `robots.txt`,
> So that crawl behavior is controlled.

**Acceptance Criteria:**
- Default allows all crawlers.
- References `sitemap.xml`.
- Customizable via job spec.

---

## Epic: DEPLOY

### US-DEPLOY-001 — Create a GitHub repository

> As the platform,
> I want a clean repo with the generated code,
> So that the customer owns the source.

**Acceptance Criteria:**
- Repo is created in the configured org.
- README, LICENSE, and `.gitignore` are present.
- Initial commit is signed by the Vibe GitHub App.

### US-DEPLOY-002 — Push generated code

> As the platform,
> I want the entire workspace pushed,
> So that the repo is buildable as cloned.

**Acceptance Criteria:**
- All generated files are committed.
- Lockfiles are committed.
- Commit message follows the conventional commits standard.

### US-DEPLOY-003 — Create a Vercel project

> As the platform,
> I want the repo linked to a Vercel project,
> So that production deploys are automatic.

**Acceptance Criteria:**
- Vercel project is created and linked.
- Build settings are correct (framework auto-detected, build command set).

### US-DEPLOY-004 — Trigger an initial deployment

> As the platform,
> I want a verified deployment URL,
> So that the customer receives a working link.

**Acceptance Criteria:**
- Initial deployment is triggered and watched.
- On success, deployment URL is recorded.
- On failure, build logs are captured and routed back to Generation.

### US-DEPLOY-005 — Set up a custom domain (V2)

> As a customer,
> I want my new site on my own domain,
> So that I do not have to give up my brand.

**Acceptance Criteria:**
- Customer provides domain.
- DNS instructions are issued.
- DNS validation is polled.
- On success, domain is attached to the Vercel project.

### US-DEPLOY-006 — Transfer repository ownership

> As Marcus,
> I want repos pushed into my client's GitHub org,
> So that ownership lives with the right party.

**Acceptance Criteria:**
- Job spec includes target org.
- The Vibe GitHub App must be installed on that org.
- Push uses installation-scoped tokens.

---

## Epic: REPORT

### US-REPORT-001 — Receive a delivery email

> As Theresa,
> I want a single email with everything I need,
> So that I do not have to dig for information.

**Acceptance Criteria:**
- Email includes preview URL, GitHub URL, and a link to the full report.
- Email is delivered within 5 minutes of job completion.

### US-REPORT-002 — Download a PDF report

> As Theresa,
> I want a PDF to share with my partner,
> So that I can explain what was done.

**Acceptance Criteria:**
- PDF is generated and downloadable from the dashboard.
- PDF includes scores, before/after, and a summary.

### US-REPORT-003 — Per-job before/after Lighthouse

> As a customer,
> I want to see the measurable improvement,
> So that the value is concrete.

**Acceptance Criteria:**
- Source-site Lighthouse scores are captured at intake.
- Delivered-site Lighthouse scores are captured post-deploy.
- The delta is highlighted.

### US-REPORT-004 — Per-job content diff

> As a customer,
> I want to know what content changed,
> So that I can verify nothing was lost.

**Acceptance Criteria:**
- A diff view shows source vs. delivered content per page.
- The diff is summarized in plain language.

---

## Epic: DASH

### US-DASH-001 — See all my jobs

> As a customer,
> I want a dashboard listing all my jobs,
> So that I can find a past delivery.

**Acceptance Criteria:**
- Sortable by date, status, URL.
- Filterable by status.

### US-DASH-002 — Track a job in real time

> As Theresa,
> I want to see my job progress,
> So that I know it is working.

**Acceptance Criteria:**
- Dashboard shows current phase (Capturing, Analyzing, Generating, etc.).
- Updates are pushed via WebSocket or polling.

### US-DASH-003 — Re-run a job

> As a customer,
> I want to re-run a job for a price,
> So that I can iterate without contacting support.

**Acceptance Criteria:**
- Re-run button on every job page.
- Re-run consumes one job quota.

### US-DASH-004 — Manage payment methods

> As a customer,
> I want to update my card,
> So that subscriptions continue uninterrupted.

**Acceptance Criteria:**
- Stripe customer portal is embedded.
- Saved methods are shown.

### US-DASH-005 — Invite teammates (V2)

> As Marcus,
> I want to invite my staff,
> So that they can manage jobs without my account.

**Acceptance Criteria:**
- Role-based access: admin, operator, viewer.
- Invitations sent by email.

---

## Epic: ADMIN

### US-ADMIN-001 — See all platform jobs

> As an operator,
> I want a console of every job,
> So that I can triage failures.

**Acceptance Criteria:**
- Internal-only access.
- Filter by status, tenant, age, error code.
- Drilldown to traces and artifacts.

### US-ADMIN-002 — Replay a failed job from any phase

> As an operator,
> I want to retry from analysis without re-capturing,
> So that I can fix bugs cheaply.

**Acceptance Criteria:**
- Per-phase replay action.
- Replays use stored artifacts as input.

### US-ADMIN-003 — Quarantine a tenant

> As an operator,
> I want to suspend a tenant immediately,
> So that I can respond to abuse.

**Acceptance Criteria:**
- One-click suspension.
- Suspension halts in-flight jobs.
- Audit log records who, when, why.

### US-ADMIN-004 — Override quality gates

> As an operator,
> I want to ship a job despite a quality warning,
> So that customers are not blocked by an edge case.

**Acceptance Criteria:**
- Override requires a reason.
- Overrides are audit-logged.

### US-ADMIN-005 — Refund a job

> As an operator,
> I want to issue a refund directly,
> So that I do not need to switch to Stripe.

**Acceptance Criteria:**
- Refund button on every paid job.
- Triggers a Stripe refund and a customer email.

---

## Epic: WHITELABEL

### US-WL-001 — Brand reports with my logo

> As Marcus,
> I want reports to carry my agency's logo,
> So that I can present them to clients.

**Acceptance Criteria:**
- Logo and color upload.
- Reports rendered with custom branding.

### US-WL-002 — Hide Vibe attribution

> As Priya,
> I want generated code to not advertise Vibe,
> So that my agency owns the perceived value.

**Acceptance Criteria:**
- Generated code does not include the word "Vibe" in user-facing strings.
- An anonymous credit remains in code comments by default; removable on Pro tier.

### US-WL-003 — Use my GitHub App installation

> As Marcus,
> I want repos pushed into my GitHub org,
> So that the source code lives in my account.

**Acceptance Criteria:**
- Connect-GitHub flow for the agency.
- Subsequent jobs default to the agency's installation.

### US-WL-004 — Use my Vercel team

> As Marcus,
> I want deployments to land in my Vercel team,
> So that billing and ops are mine.

**Acceptance Criteria:**
- Connect-Vercel flow.
- Per-tenant Vercel team selection.

---

## Epic: API

### US-API-001 — Submit a job via API

> As an API caller,
> I want to POST a URL and get a job ID,
> So that I can integrate Vibe into my workflow.

**Acceptance Criteria:**
- `POST /v1/jobs` returns 202 with `job_id` and `status_url`.
- Validates `url`, `callback_url`, optional `metadata`.

### US-API-002 — Receive a job-complete webhook

> As an API caller,
> I want a signed webhook on completion,
> So that I do not poll.

**Acceptance Criteria:**
- HMAC-SHA256 signature header.
- Replay protection via timestamp window.

### US-API-003 — List my jobs via API

> As an API caller,
> I want to list jobs,
> So that I can reconcile state.

**Acceptance Criteria:**
- Cursor-paginated.
- Filter by status, created window.

### US-API-004 — Cancel a job via API

> As an API caller,
> I want to cancel a job,
> So that I can stop work I no longer need.

**Acceptance Criteria:**
- `POST /v1/jobs/{id}/cancel`.
- Cancellable only in `queued`, `captured`, `analyzed`.

### US-API-005 — Versioned API contracts

> As an API caller,
> I want stable API versions,
> So that my integration does not break.

**Acceptance Criteria:**
- All API URLs are prefixed with `/v1/`.
- Breaking changes require `/v2/`.
- Deprecation notices precede removal by 6 months.

---

## Epic: MAINTAIN

### US-MAINTAIN-001 — Monthly recapture

> As a subscriber,
> I want my site re-captured each month,
> So that changes I made are noticed.

**Acceptance Criteria:**
- Scheduled job per active subscription.
- Diff highlights what changed.

### US-MAINTAIN-002 — Auto-PR for improvements

> As a subscriber,
> I want recommended improvements as PRs,
> So that I retain control.

**Acceptance Criteria:**
- PRs are opened against the customer's repo.
- PR description explains rationale and expected impact.

### US-MAINTAIN-003 — Auto-merge when safe (opt-in)

> As Marcus,
> I want safe PRs merged automatically,
> So that my clients' sites stay current without my time.

**Acceptance Criteria:**
- Customer opts in.
- Safe categories defined (dependency patch, alt-text fix, sitemap regen).
- All other PRs await human approval.

### US-MAINTAIN-004 — Pause and resume subscription

> As a subscriber,
> I want to pause for a season,
> So that I do not pay during dormant periods.

**Acceptance Criteria:**
- Pause halts scheduled jobs.
- Resume restores cadence.

---

## Epic: SUPPORT

### US-SUPPORT-001 — Open a ticket from any job

> As Theresa,
> I want a "report a problem" button,
> So that I do not have to find an email address.

**Acceptance Criteria:**
- Button opens a modal pre-filled with job ID.
- Submits to support inbox / helpdesk.

### US-SUPPORT-002 — Operator can reproduce a customer's job locally

> As an operator,
> I want to download all artifacts for a job,
> So that I can debug offline.

**Acceptance Criteria:**
- Single-click bundle download.
- Bundle includes capture manifest, site model, generated workspace, logs.

### US-SUPPORT-003 — Customer can request data deletion

> As a customer,
> I want my data deleted on request,
> So that I exercise my GDPR rights.

**Acceptance Criteria:**
- Self-service delete in dashboard.
- Tombstones retained per data-retention policy.

---

## Stories Coverage Map

| Persona | Epics Covered |
|---------|---------------|
| Theresa | INTAKE, BILLING, REPORT, DASH, MAINTAIN, SUPPORT |
| Marcus | INTAKE, BILLING, WHITELABEL, DASH, MAINTAIN |
| Priya | WHITELABEL, ADMIN-equivalent, API, MAINTAIN |
| Aiden | INTAKE, BILLING, API |
| API Caller | API, REPORT |
| Operator | ADMIN, SUPPORT |

---

## Assumptions

- All stories assume English-language UI for MVP.
- All stories assume web access via a modern browser.
- Email is the default notification channel for MVP; webhooks added in V3.

---

## Design Decisions

- Stories are persona-scoped, not feature-scoped, to anchor scope to user value.
- Acceptance criteria are deliberately functional, not implementation-specific.
- Sub-story granularity is delegated to the backlog (`24-backlog.md`).

---

## Open Questions

- Should the dashboard be a Next.js app on Vercel or part of the admin app on AWS?
- Should bulk CSV intake be V2 or accelerated to MVP for early agency design partners?
- Should webhook authentication use HMAC, mTLS, or both?

---

## Future Enhancements

- Stories for a Slack app integration.
- Stories for browser extensions ("Modernize this site" from any tab).
- Stories for a public gallery of modernized sites.

---

## Cross-References

- Functional decomposition → `05-functional-requirements.md`
- Engineering backlog → `24-backlog.md`
- API surface → `09-api-specification.md`
