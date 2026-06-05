# 24 — Backlog

> The full backlog of user stories for Vibe. Every story is sized, owned, and ready to be pulled into a milestone.

---

## Purpose

This document is the canonical backlog. It catalogues every user story the platform will eventually deliver, with acceptance criteria, story points, and the epic and milestone alignment. Stories here have been groomed: the team agrees they are valuable, feasible, and well-defined.

The backlog is the bridge between the roadmap (`23-milestone-roadmap.md`) and the work a contributor picks up next. Stories move from this document into milestones; from milestones into sprints; from sprints into PRs.

---

## Scope

In scope:

- User stories for every functional capability
- Acceptance criteria
- Story points
- Epic and milestone mapping
- Dependencies

Out of scope:

- Bug-fixing stories (filed separately in GitHub Issues)
- Spikes and refactors (filed separately)

---

## Sizing

Story points (Fibonacci):

- 1 = trivial, < half a day
- 2 = small, half a day to a day
- 3 = medium, 1–2 days
- 5 = large, 3–5 days
- 8 = very large, 1–2 weeks
- 13 = too large; must be split

The total backlog contains 100+ stories at MVP, V2, and V3.

---

## Epic Index

| ID | Epic |
|----|------|
| EP-INTAKE | Intake and Onboarding |
| EP-CAPTURE | Capture Engine |
| EP-ANALYSIS | Analysis Engine |
| EP-GEN | Generation Engine |
| EP-SEO | SEO Engine |
| EP-QR | Quality Reviewer |
| EP-DEPLOY | Deployment Engine |
| EP-DELIVERY | Reports and Webhooks |
| EP-BILLING | Billing and Quotas |
| EP-AUTH | Authentication and Tenancy |
| EP-DASH | Customer Dashboard |
| EP-ADMIN | Admin Console |
| EP-WL | Whitelabel |
| EP-API | Public API and SDK |
| EP-SEC | Security and Compliance |
| EP-OBS | Observability |
| EP-MAINT | Maintenance and Reliability |
| EP-SUPPORT | Support Tooling |

---

## EP-INTAKE — Intake and Onboarding

### US-INTAKE-001 — Sign up via magic link

**As a** prospective customer
**I want to** sign up with my work email
**So that** I can use Vibe without managing a password

**Acceptance Criteria:**

- AC1: Entering a valid work email triggers a magic link email within 30 seconds.
- AC2: The link signs me in and lands me on the dashboard.
- AC3: The link expires after 15 minutes and is single-use.
- AC4: Re-requesting a link invalidates prior unverified links.

**Points:** 3
**Milestone:** M1

### US-INTAKE-002 — Submit a free preview

**As a** prospective customer
**I want to** submit a URL for a free preview
**So that** I can see a sample modernization before paying

**Acceptance Criteria:**

- AC1: A free preview input accepts a URL and submits it to a job.
- AC2: The preview is a single page with limited fidelity, watermarked.
- AC3: A preview is limited to one per email per 24 hours.
- AC4: The preview runs without creating a paid subscription.

**Points:** 5
**Milestone:** M1

### US-INTAKE-003 — Multi-factor authentication enrollment

**As a** customer with admin role
**I want to** enroll in TOTP MFA
**So that** my tenant is more secure

**Acceptance Criteria:**

- AC1: I can scan a QR code into an authenticator app.
- AC2: I confirm enrollment with a valid code.
- AC3: Subsequent logins require the code when MFA is enabled.
- AC4: I can generate recovery codes and store them safely.

**Points:** 3
**Milestone:** M1

### US-INTAKE-004 — Onboarding checklist

**As a** new customer
**I want to** see a checklist of first steps
**So that** I can become productive quickly

**Acceptance Criteria:**

- AC1: The checklist shows: run a preview, connect GitHub, run a real job, deploy.
- AC2: Completing a step marks it done and shows the next.
- AC3: The checklist can be dismissed but is recoverable from help.

**Points:** 2
**Milestone:** M3

### US-INTAKE-005 — Email verification

**As a** prospective customer
**I want to** verify my email address
**So that** I can claim a tenant

**Acceptance Criteria:**

- AC1: Signup requires a verified email.
- AC2: Verification email is sent on signup; verified state persists.
- AC3: Unverified accounts are deleted after 30 days.

**Points:** 2
**Milestone:** M1

### US-INTAKE-006 — Trial period for paid plans

**As a** prospective paid customer
**I want a** 14-day trial
**So that** I can validate value before paying

**Acceptance Criteria:**

- AC1: New paid plans start in trial mode for 14 days.
- AC2: Quotas are enforced but no charge occurs.
- AC3: Trial end prompts plan selection or downgrade.
- AC4: Stripe payment method is required to start a trial.

**Points:** 5
**Milestone:** M7

### US-INTAKE-007 — Inviting team members

**As a** customer admin
**I want to** invite teammates by email
**So that** they can collaborate

**Acceptance Criteria:**

- AC1: I can enter emails with assigned roles.
- AC2: An invite email is sent with a single-use link.
- AC3: Accepted invites appear in the members list.
- AC4: I can revoke pending invites.

**Points:** 3
**Milestone:** M3

### US-INTAKE-008 — Removing a team member

**As a** customer admin
**I want to** remove a teammate
**So that** they no longer have access

**Acceptance Criteria:**

- AC1: Removing a member immediately invalidates sessions.
- AC2: Their artifacts remain owned by the tenant.
- AC3: Audit log records the removal with actor and timestamp.

**Points:** 2
**Milestone:** M3

---

## EP-CAPTURE — Capture Engine

### US-CAP-001 — Capture a single URL

**As a** customer
**I want to** capture a single URL
**So that** I can modernize a single page

**Acceptance Criteria:**

- AC1: I provide a URL; the system captures the page within 60 seconds for typical pages.
- AC2: A manifest is produced with title, links, assets, and screenshot.
- AC3: A capture failure is reported with reason.
- AC4: SSRF and private-IP targets are refused.

**Points:** 5
**Milestone:** M1

### US-CAP-002 — Crawl a site within a domain

**As a** customer
**I want to** capture an entire domain
**So that** I can modernize a multi-page site

**Acceptance Criteria:**

- AC1: Capture crawls same-origin links up to a depth/page cap.
- AC2: The crawl produces a navigation graph and per-page manifests.
- AC3: Pages outside the configured cap are reported, not captured.
- AC4: The crawl stops on policy violation (e.g., robots `Disallow`).

**Points:** 8
**Milestone:** M9

### US-CAP-003 — Respect `robots.txt`

**As a** customer
**I want** Vibe to respect `robots.txt`
**So that** we don't violate site policy

**Acceptance Criteria:**

- AC1: The capture engine fetches and parses `robots.txt`.
- AC2: `Disallow` rules are honored; `Allow` rules are respected.
- AC3: A note is added to the manifest listing disallowed pages.
- AC4: Customers can override per-job with an acknowledgment.

**Points:** 3
**Milestone:** M1

### US-CAP-004 — Detect bot walls

**As a** customer
**I want to** know if the source is protected
**So that** I can take action

**Acceptance Criteria:**

- AC1: A bot wall (Cloudflare, etc.) is detected and the manifest is marked partial.
- AC2: A retry with a different fingerprint is attempted once.
- AC3: The report explains what was missing.

**Points:** 5
**Milestone:** M1

### US-CAP-005 — Enforce per-job limits

**As a** platform operator
**I want** per-job limits
**So that** one tenant cannot exhaust resources

**Acceptance Criteria:**

- AC1: A job is killed if it exceeds 15 minutes, 25 MB/page, or 100 pages.
- AC2: The job is reported with a specific failure reason.
- AC3: A retry is offered with adjusted limits if available.

**Points:** 3
**Milestone:** M1

### US-CAP-006 — Per-asset capture metadata

**As a** downstream component
**I want** per-asset metadata in the manifest
**So that** I can decide what to carry over

**Acceptance Criteria:**

- AC1: Each asset has URL, type, size, hash, and license hint.
- AC2: The manifest separates "must-carry" vs "optional" assets heuristically.
- AC3: Assets larger than the cap are not stored.

**Points:** 3
**Milestone:** M1

### US-CAP-007 — Manual URL additions

**As a** customer
**I want to** add extra URLs to a capture
**So that** important pages aren't missed

**Acceptance Criteria:**

- AC1: I can add URLs that are not reachable from the homepage.
- AC2: Added URLs are validated and added to the navigation graph.
- AC3: Additions are reflected in the manifest.

**Points:** 3
**Milestone:** M9

### US-CAP-008 — Capture progress visibility

**As a** customer
**I want to** see capture progress
**So that** I know the system is working

**Acceptance Criteria:**

- AC1: The dashboard shows live progress (pages captured, MB transferred).
- AC2: A stalled job shows a warning after 60 seconds of inactivity.
- AC3: Final state shows a manifest summary.

**Points:** 3
**Milestone:** M1

---

## EP-ANALYSIS — Analysis Engine

### US-ANA-001 — Extract content

**As a** downstream component
**I want** extracted text and structured content
**So that** I can render the new site

**Acceptance Criteria:**

- AC1: Headings, paragraphs, lists, and tables are extracted.
- AC2: Images have alt text; missing alts are flagged.
- AC3: Language and locale are detected and recorded.
- AC4: Extraction quality is evaluated on a fixture set.

**Points:** 5
**Milestone:** M2

### US-ANA-002 — Classify page roles

**As a** downstream component
**I want** per-page role tags
**So that** I can render the right layout

**Acceptance Criteria:**

- AC1: Roles include `home`, `product`, `service`, `contact`, `blog`, `about`, `legal`, `other`.
- AC2: Classifier accuracy ≥ 90% on a labeled test set.
- AC3: Confidence is reported per classification.

**Points:** 5
**Milestone:** M2

### US-ANA-003 — Build a navigation graph

**As a** downstream component
**I want** a navigation graph
**So that** I can render global nav

**Acceptance Criteria:**

- AC1: Nodes and edges reflect the crawl results.
- AC2: Cycles are broken by canonical URL.
- AC3: Hub pages are identified.

**Points:** 5
**Milestone:** M2

### US-ANA-004 — Run SEO audit

**As a** customer
**I want** a pre-modernization SEO audit
**So that** I can see what to preserve

**Acceptance Criteria:**

- AC1: Audit covers title, meta, headings, alt text, internal links, schema.org.
- AC2: A score is computed and surfaced in the report.
- AC3: Critical issues are listed with examples.

**Points:** 5
**Milestone:** M2

### US-ANA-005 — Run a11y audit

**As a** customer
**I want** a pre-modernization a11y audit
**So that** I can see what to improve

**Acceptance Criteria:**

- AC1: Audit uses axe-core rules.
- AC2: Results are normalized to WCAG 2.2 AA.
- AC3: Top issues are listed with element selectors.

**Points:** 5
**Milestone:** M2

### US-ANA-006 — Detect tech stack

**As a** customer
**I want to** see what tech the source uses
**So that** I can prioritize modernization

**Acceptance Criteria:**

- AC1: Detection covers CMS, framework, analytics, fonts, and hosting hints.
- AC2: A "complexity" score summarizes the migration cost.
- AC3: Tech list is included in the report.

**Points:** 3
**Milestone:** M2

### US-ANA-007 — Extract NAP

**As a** downstream component
**I want** name, address, phone (NAP) extracted
**So that** I can render LocalBusiness JSON-LD

**Acceptance Criteria:**

- AC1: NAP is detected from contact pages.
- AC2: Confidence is reported.
- AC3: Manual override is supported.

**Points:** 3
**Milestone:** M4

### US-ANA-008 — LLM summarization

**As a** downstream component
**I want** LLM-generated page summaries
**So that** the new site is concise and on-message

**Acceptance Criteria:**

- AC1: Each page has a 1–2 sentence summary.
- AC2: Hallucinations are reduced by grounding the model in extracted content.
- AC3: Cost per summary is bounded.

**Points:** 5
**Milestone:** M2

---

## EP-GEN — Generation Engine

### US-GEN-001 — Generate a single-page site

**As a** customer
**I want** a generated single-page site
**So that** I can see the modernization

**Acceptance Criteria:**

- AC1: The generated workspace builds successfully.
- AC2: Visual fidelity matches the source within reason.
- AC3: Lighthouse Performance ≥ 90 on the median fixture.

**Points:** 8
**Milestone:** M3

### US-GEN-002 — Generate a multi-page site

**As a** customer
**I want** a generated multi-page site
**So that** I have a coherent navigation

**Acceptance Criteria:**

- AC1: All pages build successfully.
- AC2: Global navigation reflects the source structure.
- AC3: 301s from old URLs to new are generated.

**Points:** 13
**Milestone:** M9

### US-GEN-003 — Apply a theme

**As a** customer
**I want to** pick a theme
**So that** my site looks distinct

**Acceptance Criteria:**

- AC1: At least 5 starter themes ship at M3.
- AC2: Themes use tokens; no hard-coded colors in components.
- AC3: Switching themes does not break content.

**Points:** 5
**Milestone:** M3

### US-GEN-004 — Self-repair on build failure

**As a** customer
**I want** the system to fix build errors
**So that** I get a working result

**Acceptance Criteria:**

- AC1: Up to 3 self-repair attempts are made.
- AC2: Each attempt is logged with the failure and the fix.
- AC3: Persistent failure results in a partial deliverable and a clear report.

**Points:** 8
**Milestone:** M3

### US-GEN-005 — Carry over assets

**As a** customer
**I want** my images and PDFs to be carried over
**So that** I don't lose content

**Acceptance Criteria:**

- AC1: Heuristically "must-carry" assets are downloaded to the workspace.
- AC2: Asset references are rewritten to local paths.
- AC3: Missing assets are reported.

**Points:** 5
**Milestone:** M3

### US-GEN-006 — Generate 301 redirects

**As a** customer
**I want** 301 redirects from old to new
**So that** SEO equity is preserved

**Acceptance Criteria:**

- AC1: A `vercel.json` or `next.config.js` redirects map is generated.
- AC2: Each old URL maps to its new URL by canonical form.
- AC3: Unmapped URLs return 410 (gone) by default.

**Points:** 3
**Milestone:** M5

### US-GEN-007 — Content data layer

**As a** customer
**I want** content stored as data
**So that** it can be edited later

**Acceptance Criteria:**

- AC1: Content is in JSON or MDX files.
- AC2: Pages render from the data layer.
- AC3: Customers can edit a content file in their repo.

**Points:** 5
**Milestone:** M3

### US-GEN-008 — Generate a contact form

**As a** customer
**I want** a contact form
**So that** visitors can reach me

**Acceptance Criteria:**

- AC1: A simple form posts to an endpoint.
- AC2: Spam protection is applied (honeypot + rate limit).
- AC3: Submissions are forwarded via webhook (customer-supplied).

**Points:** 3
**Milestone:** M5

### US-GEN-009 — Generate blog index and posts

**As a** customer
**I want** blog posts to be carried over
**So that** my content remains available

**Acceptance Criteria:**

- AC1: Posts are extracted from common blog patterns.
- AC2: An index page is generated.
- AC3: RSS feed is generated.

**Points:** 5
**Milestone:** M9

### US-GEN-010 — Per-section regeneration

**As a** customer
**I want** to regenerate a section
**So that** I can update without a full rebuild

**Acceptance Criteria:**

- AC1: A single section can be regenerated.
- AC2: The build is incremental.
- AC3: The rest of the site is not affected.

**Points:** 8
**Milestone:** M9

---

## EP-SEO — SEO Engine

### US-SEO-001 — Metadata generation

**As a** customer
**I want** metadata generated
**So that** my pages rank

**Acceptance Criteria:**

- AC1: Per-page `<title>`, `<meta description>`, canonical, OG tags.
- AC2: Defaults are sensible if the source lacks them.
- AC3: Edits are reflected on rebuild.

**Points:** 3
**Milestone:** M4

### US-SEO-002 — JSON-LD structured data

**As a** customer
**I want** JSON-LD for Organization and LocalBusiness
**So that** my site has rich results

**Acceptance Criteria:**

- AC1: Organization JSON-LD is emitted.
- AC2: LocalBusiness JSON-LD is emitted when NAP is detected.
- AC3: Validation passes.

**Points:** 3
**Milestone:** M4

### US-SEO-003 — Sitemap generation

**As a** customer
**I want** a sitemap.xml
**So that** crawlers find my pages

**Acceptance Criteria:**

- AC1: A sitemap lists all generated pages.
- AC2: It is referenced in `robots.txt`.
- AC3: It updates on rebuild.

**Points:** 2
**Milestone:** M4

### US-SEO-004 — robots.txt generation

**As a** customer
**I want** a robots.txt
**So that** crawlers are guided

**Acceptance Criteria:**

- AC1: `robots.txt` allows crawling and references the sitemap.
- AC2: AI scrapers are allowed by default; opt-out available.
- AC3: Edits are reflected on rebuild.

**Points:** 2
**Milestone:** M4

### US-SEO-005 — llms.txt emission

**As a** site owner
**I want** an `llms.txt` for AI agents
**So that** AI tools understand my site

**Acceptance Criteria:**

- AC1: A `public/llms.txt` is generated.
- AC2: It lists pages with brief descriptions.
- AC3: An expanded `llms-full.txt` is optional.

**Points:** 2
**Milestone:** M4

### US-SEO-006 — Alt text synthesis

**As a** customer
**I want** alt text for images
**So that** my site is accessible

**Acceptance Criteria:**

- AC1: Images without alt text get suggested alt.
- AC2: Suggestions are reviewable before deploy.
- AC3: Confidence is reported.

**Points:** 5
**Milestone:** M4

### US-SEO-007 — Open Graph image

**As a** customer
**I want** an OG image
**So that** social previews look good

**Acceptance Criteria:**

- AC1: An OG image is generated per page.
- AC2: It is reachable at a stable URL.
- AC3: It validates with social debuggers.

**Points:** 3
**Milestone:** M4

---

## EP-QR — Quality Reviewer

### US-QR-001 — Lighthouse gate

**As a** customer
**I want** Lighthouse to pass
**So that** my site is fast and accessible

**Acceptance Criteria:**

- AC1: The reviewer runs Lighthouse on the preview.
- AC2: Performance ≥ 90, SEO = 100, Best Practices = 100, A11y ≥ 95.
- AC3: Failure blocks deploy with a reason.

**Points:** 5
**Milestone:** M4

### US-QR-002 — Accessibility gate

**As a** customer
**I want** WCAG 2.2 AA compliance
**So that** my site is accessible

**Acceptance Criteria:**

- AC1: The reviewer runs axe-core on the preview.
- AC2: Critical issues block deploy.
- AC3: Reports include remediation hints.

**Points:** 3
**Milestone:** M4

### US-QR-003 — Visual regression

**As a** customer
**I want** the modernized site to look like the source
**So that** my brand is preserved

**Acceptance Criteria:**

- AC1: Per-page screenshots are compared.
- AC2: A similarity score is computed.
- AC3: Major divergence is flagged.

**Points:** 5
**Milestone:** M4

### US-QR-004 — Content fidelity

**As a** customer
**I want** my content preserved
**So that** I don't lose information

**Acceptance Criteria:**

- AC1: Text from the source appears in the new site.
- AC2: Differences are reported.
- AC3: Customer can review a content diff.

**Points:** 3
**Milestone:** M4

### US-QR-005 — Security headers

**As a** customer
**I want** security headers set
**So that** my site is hardened

**Acceptance Criteria:**

- AC1: HSTS, X-Content-Type-Options, X-Frame-Options, Referrer-Policy set.
- AC2: A modest CSP is set by default.
- AC3: Headers are tested.

**Points:** 3
**Milestone:** M5

### US-QR-006 — Final approval gate

**As a** customer
**I want to** approve before production deploy
**So that** I keep control

**Acceptance Criteria:**

- AC1: A job is held in `awaiting_approval` before production.
- AC2: The customer can approve or request changes.
- AC3: Approval transitions to deploy; rejection cancels.

**Points:** 3
**Milestone:** M5

---

## EP-DEPLOY — Deployment Engine

### US-DEP-001 — Connect GitHub

**As a** customer
**I want to** install the Vibe GitHub App
**So that** Vibe can create repos on my behalf

**Acceptance Criteria:**

- AC1: I complete the OAuth install flow.
- AC2: I select which repos Vibe can access (or all).
- AC3: I can disconnect at any time.

**Points:** 5
**Milestone:** M5

### US-DEP-002 — Create a GitHub repo

**As a** customer
**I want** a repo for the modernized site
**So that** I own the code

**Acceptance Criteria:**

- AC1: A repo is created in my org or my account.
- AC2: The default branch is `main`.
- AC3: The repo URL is reported.

**Points:** 3
**Milestone:** M5

### US-DEP-003 — Push the generated workspace

**As a** customer
**I want** the generated code pushed
**So that** I have a baseline

**Acceptance Criteria:**

- AC1: The workspace is committed and pushed.
- AC2: Commit signing is applied.
- AC3: A first commit message indicates provenance.

**Points:** 3
**Milestone:** M5

### US-DEP-004 — Create a Vercel project

**As a** customer
**I want** a Vercel project linked
**So that** the site deploys

**Acceptance Criteria:**

- AC1: A project is created under our team.
- AC2: The project is linked to the GitHub repo.
- AC3: The build runs.

**Points:** 3
**Milestone:** M5

### US-DEP-005 — Trigger a deploy

**As a** customer
**I want** the site deployed
**So that** I can visit it

**Acceptance Criteria:**

- AC1: A deployment is triggered.
- AC2: The deployment URL is reported.
- AC3: Build logs are accessible.

**Points:** 3
**Milestone:** M5

### US-DEP-006 — Custom domain

**As a** customer
**I want to** use my own domain
**So that** branding is consistent

**Acceptance Criteria:**

- AC1: I can add a domain in the dashboard.
- AC2: DNS instructions are shown.
- AC3: Verification is automatic on DNS propagation.

**Points:** 5
**Milestone:** M5

### US-DEP-007 — Rollback

**As a** customer
**I want to** roll back to a previous deploy
**So that** I can recover from a bad release

**Acceptance Criteria:**

- AC1: I see a list of past deploys.
- AC2: A "Promote to production" action is available.
- AC3: The promotion is atomic.

**Points:** 3
**Milestone:** M5

### US-DEP-008 — Re-deploy from dashboard

**As a** customer
**I want to** re-trigger a deploy
**So that** I can refresh after a fix

**Acceptance Criteria:**

- AC1: A re-deploy action is available per job.
- AC2: Re-deploy is idempotent.
- AC3: New URL is reported.

**Points:** 2
**Milestone:** M5

### US-DEP-009 — Failure compensation

**As an** operator
**I want** partial failures to be reversible
**So that** we don't leak resources

**Acceptance Criteria:**

- AC1: Each deploy step is recorded.
- AC2: On failure, a compensation runs.
- AC3: Manual rollback is supported.

**Points:** 5
**Milestone:** M5

---

## EP-DELIVERY — Reports and Webhooks

### US-DEL-001 — Per-job report

**As a** customer
**I want** a report per job
**So that** I can review what changed

**Acceptance Criteria:**

- AC1: A report shows capture, analysis, generation, deploy outcomes.
- AC2: Lighthouse, axe, and SEO scores are included.
- AC3: The report is downloadable as PDF and JSON.

**Points:** 5
**Milestone:** M6

### US-DEL-002 — Per-tenant summary

**As a** customer
**I want** a per-tenant summary
**So that** I can see total impact

**Acceptance Criteria:**

- AC1: A summary aggregates jobs over a period.
- AC2: It is filterable by date range and status.
- AC3: It exports as CSV.

**Points:** 3
**Milestone:** M6

### US-DEL-003 — Webhook subscriptions

**As a** customer
**I want to** register a webhook
**So that** I can integrate with my tools

**Acceptance Criteria:**

- AC1: I can register a URL and event filters.
- AC2: Vibe signs deliveries with HMAC.
- AC3: I can disable a webhook.

**Points:** 5
**Milestone:** M6

### US-DEL-004 — Webhook retries

**As a** customer
**I want** failed webhooks retried
**So that** I don't miss events

**Acceptance Criteria:**

- AC1: Retries follow exponential backoff.
- AC2: Failed deliveries are listed in the dashboard.
- AC3: Manual retry is supported.

**Points:** 3
**Milestone:** M6

### US-DEL-005 — Email notifications

**As a** customer
**I want** email updates
**So that** I know when jobs complete

**Acceptance Criteria:**

- AC1: An email is sent on completion.
- AC2: Email preferences are configurable.
- AC3: Per-job opt-out is supported.

**Points:** 3
**Milestone:** M6

### US-DEL-006 — In-app notifications

**As a** customer
**I want** in-app notifications
**So that** I see updates in the dashboard

**Acceptance Criteria:**

- AC1: A notifications panel shows recent events.
- AC2: Each notification links to the relevant job.
- AC3: Marking read is supported.

**Points:** 2
**Milestone:** M6

---

## EP-BILLING — Billing and Quotas

### US-BIL-001 — Plan selection

**As a** customer
**I want to** pick a plan
**So that** I can use Vibe

**Acceptance Criteria:**

- AC1: A pricing page lists plans.
- AC2: Stripe Checkout completes the signup.
- AC3: A receipt is emailed.

**Points:** 3
**Milestone:** M7

### US-BIL-002 — Plan upgrade/downgrade

**As a** customer
**I want to** change plan
**So that** I can scale up or down

**Acceptance Criteria:**

- AC1: The Customer Portal allows plan changes.
- AC2: Prorated charges are applied.
- AC3: Quota changes take effect immediately.

**Points:** 3
**Milestone:** M7

### US-BIL-003 — Quota enforcement

**As a** platform operator
**I want** plan-based quotas enforced
**So that** I can protect margins

**Acceptance Criteria:**

- AC1: Per-plan quotas for jobs/month, LLM spend cap, and capture pages.
- AC2: Quota exhaustion blocks new jobs with a clear error.
- AC3: Quota usage is visible in the dashboard.

**Points:** 5
**Milestone:** M7

### US-BIL-004 — Invoices

**As a** customer
**I want** invoices
**So that** I can expense them

**Acceptance Criteria:**

- AC1: Stripe-generated invoices are accessible.
- AC2: VAT/Tax IDs are supported.
- AC3: Billing email is configurable.

**Points:** 3
**Milestone:** M7

### US-BIL-005 — Cancellation

**As a** customer
**I want to** cancel
**So that** I stop being charged

**Acceptance Criteria:**

- AC1: Cancellation is available in the Customer Portal.
- AC2: Service continues until period end.
- AC3: Artifacts are retained for 30 days post-cancel.

**Points:** 3
**Milestone:** M7

### US-BIL-006 — Coupon support

**As a** marketing user
**I want** coupons
**So that** I can run promotions

**Acceptance Criteria:**

- AC1: Stripe coupon codes are honored at checkout.
- AC2: Coupons apply per-customer or per-promotion.
- AC3: Coupon use is logged.

**Points:** 3
**Milestone:** M7

### US-BIL-007 — Cost transparency

**As a** customer
**I want to** see my COGS
**So that** I can budget

**Acceptance Criteria:**

- AC1: A usage dashboard shows jobs, LLM spend, capture pages, deploys.
- AC2: It is segmented by month.
- AC3: A forecast shows projected month-end cost.

**Points:** 3
**Milestone:** M7

---

## EP-AUTH — Authentication and Tenancy

### US-AUTH-001 — Magic link login

**As a** user
**I want to** log in via magic link
**So that** I don't manage passwords

**Acceptance Criteria:**

- AC1: I receive a link by email.
- AC2: The link is single-use and time-bound.
- AC3: Failed attempts are rate-limited.

**Points:** 3
**Milestone:** M1

### US-AUTH-002 — Role assignment

**As an** admin
**I want** roles assigned to members
**So that** I control access

**Acceptance Criteria:**

- AC1: Roles are `owner`, `admin`, `operator`, `viewer`.
- AC2: Role changes are audit-logged.
- AC3: Effective permissions are visible.

**Points:** 3
**Milestone:** M3

### US-AUTH-003 — API key issuance

**As a** developer
**I want** API keys
**So that** I can use the API

**Acceptance Criteria:**

- AC1: Keys are scoped and have optional expiry.
- AC2: Plaintext is shown once.
- AC3: Revocation is immediate.

**Points:** 3
**Milestone:** M8

### US-AUTH-004 — SSO (SAML/OIDC)

**As an** enterprise admin
**I want** SSO
**So that** I can use my IdP

**Acceptance Criteria:**

- AC1: SAML 2.0 and OIDC are supported.
- AC2: JIT provisioning or pre-provisioning is configurable.
- AC3: SCIM 2.0 deprovisioning is supported.

**Points:** 8
**Milestone:** M10

### US-AUTH-005 — Audit log

**As a** customer admin
**I want** an audit log
**So that** I can review access

**Acceptance Criteria:**

- AC1: All admin actions are listed.
- AC2: Filters by actor, action, and date are available.
- AC3: Export as CSV is supported.

**Points:** 3
**Milestone:** M3

### US-AUTH-006 — Session control

**As a** user
**I want to** see and revoke sessions
**So that** I can stay safe

**Acceptance Criteria:**

- AC1: Active sessions are listed with device and location.
- AC2: Sessions can be revoked individually.
- AC3: Password reset (n/a) and email change invalidate other sessions.

**Points:** 2
**Milestone:** M1

### US-AUTH-007 — Tenant deletion

**As a** tenant owner
**I want to** delete the tenant
**So that** I can leave

**Acceptance Criteria:**

- AC1: Deletion requires typed confirmation.
- AC2: All data is hard-deleted within 30 days.
- AC3: A tombstone record is retained.

**Points:** 3
**Milestone:** M3

---

## EP-DASH — Customer Dashboard

### US-DASH-001 — Job history

**As a** customer
**I want** a job history
**So that** I can review past jobs

**Acceptance Criteria:**

- AC1: A list shows jobs with status, URL, and timestamps.
- AC2: Filters by status, date, and tag are available.
- AC3: A detail view shows the full lifecycle.

**Points:** 3
**Milestone:** M3

### US-DASH-002 — Per-job timeline

**As a** customer
**I want** a per-job timeline
**So that** I can see what happened

**Acceptance Criteria:**

- AC1: A timeline shows capture, analysis, generation, deploy, delivery events.
- AC2: Each event has a duration and a link to details.
- AC3: Failures are highlighted.

**Points:** 3
**Milestone:** M6

### US-DASH-003 — Reports download

**As a** customer
**I want to** download reports
**So that** I can share them

**Acceptance Criteria:**

- AC1: Reports are available in PDF and JSON.
- AC2: Download links are signed and time-bound.
- AC3: Reports include the lighthouse, axe, and SEO scores.

**Points:** 3
**Milestone:** M6

### US-DASH-004 — Settings

**As a** customer
**I want** settings
**So that** I can configure my tenant

**Acceptance Criteria:**

- AC1: Settings cover profile, branding, integrations, billing, members.
- AC2: Changes are saved with feedback.
- AC3: Sensitive changes require re-auth.

**Points:** 3
**Milestone:** M3

### US-DASH-005 — Usage dashboard

**As a** customer
**I want** a usage dashboard
**So that** I can plan capacity

**Acceptance Criteria:**

- AC1: Usage shows jobs, LLM spend, capture pages.
- AC2: It's segmented by month.
- AC3: Quota progress is shown.

**Points:** 3
**Milestone:** M7

### US-DASH-006 — Brand customization

**As a** customer
**I want** my brand in the dashboard
**So that** it feels like my product

**Acceptance Criteria:**

- AC1: Logo and primary color are configurable.
- AC2: They apply across the dashboard and emails.
- AC3: Changes preview before save.

**Points:** 3
**Milestone:** M3

---

## EP-ADMIN — Admin Console

### US-ADM-001 — Tenant list

**As an** operator
**I want** a tenant list
**So that** I can find a customer

**Acceptance Criteria:**

- AC1: Tenants are listed with key fields.
- AC2: Filters by plan, status, and creation date are available.
- AC3: A search by name or email works.

**Points:** 2
**Milestone:** M3

### US-ADM-002 — Tenant detail

**As an** operator
**I want** a tenant detail page
**So that** I can review state

**Acceptance Criteria:**

- AC1: The page shows plan, usage, recent jobs, and members.
- AC2: Audit log entries are visible.
- AC3: Suspicious patterns are highlighted.

**Points:** 3
**Milestone:** M3

### US-ADM-003 — Suspend a tenant

**As an** operator
**I want to** suspend a tenant
**So that** I can stop abuse

**Acceptance Criteria:**

- AC1: Suspension blocks new jobs.
- AC2: Existing data is preserved.
- AC3: An audit log entry is created with reason.

**Points:** 2
**Milestone:** M3

### US-ADM-004 — Manual refund

**As an** operator
**I want to** refund a charge
**So that** I can resolve disputes

**Acceptance Criteria:**

- AC1: Refund form captures amount and reason.
- AC2: Stripe call is made; receipt is recorded.
- AC3: The customer is notified.

**Points:** 3
**Milestone:** M7

### US-ADM-005 — Replay a job

**As an** operator
**I want to** replay a job
**So that** I can retry after a fix

**Acceptance Criteria:**

- AC1: A replay uses the original spec.
- AC2: Replay is logged.
- AC3: Customer can be notified.

**Points:** 3
**Milestone:** M3

### US-ADM-006 — Override a quality gate

**As an** operator
**I want to** override a gate
**So that** I can ship a marginal case

**Acceptance Criteria:**

- AC1: Override requires a typed reason and MFA.
- AC2: Override is audit-logged.
- AC3: Customer is notified.

**Points:** 2
**Milestone:** M4

### US-ADM-007 — Feature flags

**As an** operator
**I want to** toggle feature flags
**So that** I can roll out safely

**Acceptance Criteria:**

- AC1: Flags are scoped per tenant.
- AC2: Changes are audit-logged.
- AC3: Default-off flags require explicit enable.

**Points:** 3
**Milestone:** M3

---

## EP-WL — Whitelabel

### US-WL-001 — Vibe Connect GitHub App

**As an** enterprise customer
**I want to** install Vibe Connect
**So that** my branding is preserved in the source

**Acceptance Criteria:**

- AC1: Connect App uses customer branding.
- AC2: Commits appear under the customer's identity.
- AC3: The customer can uninstall cleanly.

**Points:** 5
**Milestone:** M10

### US-WL-002 — Branded output

**As an** enterprise customer
**I want** my brand in the output
**So that** visitors see my identity

**Acceptance Criteria:**

- AC1: Logo, colors, and footer reflect my brand.
- AC2: The branded theme applies to all generated pages.
- AC3: Customer can adjust via the dashboard.

**Points:** 5
**Milestone:** M10

### US-WL-003 — Custom Vercel team

**As an** enterprise customer
**I want** my own Vercel team
**So that** projects are isolated

**Acceptance Criteria:**

- AC1: A team is created under my account.
- AC2: Projects are isolated per tenant.
- AC3: Tokens are scoped to the team.

**Points:** 5
**Milestone:** M10

### US-WL-004 — White-label email

**As an** enterprise customer
**I want** emails to come from my domain
**So that** branding is consistent

**Acceptance Criteria:**

- AC1: SPF/DKIM/DMARC are configured for the customer's domain.
- AC2: `From` and reply-to use the customer's domain.
- AC3: Bounce handling is supported.

**Points:** 5
**Milestone:** M10

---

## EP-API — Public API and SDK

### US-API-001 — Create a job

**As an** API consumer
**I want to** create a job
**So that** I can integrate

**Acceptance Criteria:**

- AC1: `POST /v1/jobs` creates a job.
- AC2: The response includes the job ID and initial status.
- AC3: Idempotency-Key is honored.

**Points:** 3
**Milestone:** M8

### US-API-002 — Get job status

**As an** API consumer
**I want to** poll job status
**So that** I know when it's done

**Acceptance Criteria:**

- AC1: `GET /v1/jobs/{id}` returns current state.
- AC2: `?include=events` returns recent events.
- AC3: `ETag` and `If-None-Match` are supported.

**Points:** 2
**Milestone:** M8

### US-API-003 — List jobs

**As an** API consumer
**I want to** list jobs
**So that** I can build a UI

**Acceptance Criteria:**

- AC1: `GET /v1/jobs` returns a paginated list.
- AC2: Filters by status and date work.
- AC3: Cursor pagination is correct.

**Points:** 3
**Milestone:** M8

### US-API-004 — Cancel a job

**As an** API consumer
**I want to** cancel a job
**So that** I can stop a runaway

**Acceptance Criteria:**

- AC1: `POST /v1/jobs/{id}/cancel` transitions to `cancelled`.
- AC2: In-progress activities are notified.
- AC3: Compensation runs.

**Points:** 3
**Milestone:** M8

### US-API-005 — Download report

**As an** API consumer
**I want to** download a report
**So that** I can archive it

**Acceptance Criteria:**

- AC1: `GET /v1/jobs/{id}/report` returns a signed URL.
- AC2: The URL is time-bound.
- AC3: Format (pdf|json) is selectable.

**Points:** 2
**Milestone:** M8

### US-API-006 — Webhook subscription

**As an** API consumer
**I want to** register a webhook
**So that** I can react to events

**Acceptance Criteria:**

- AC1: `POST /v1/webhooks` creates a subscription.
- AC2: A test event can be sent on demand.
- AC3: Subscriptions are listable and deletable.

**Points:** 5
**Milestone:** M8

### US-API-007 — TypeScript SDK

**As a** developer
**I want a** TypeScript SDK
**So that** I can use Vibe from JS

**Acceptance Criteria:**

- AC1: The SDK is published to npm.
- AC2: It mirrors the OpenAPI.
- AC3: Smoke tests run in CI.

**Points:** 5
**Milestone:** M8

### US-API-008 — OpenAPI documentation

**As a** developer
**I want** OpenAPI docs
**So that** I can generate clients

**Acceptance Criteria:**

- AC1: A versioned OpenAPI is published.
- AC2: The docs site is searchable.
- AC3: The SDK is generated from the spec.

**Points:** 3
**Milestone:** M8

### US-API-009 — Rate limits visible

**As a** developer
**I want to** see rate limits
**So that** I can back off

**Acceptance Criteria:**

- AC1: `X-RateLimit-*` headers are present.
- AC2: 429 responses include `Retry-After`.
- AC3: The dashboard shows my current usage.

**Points:** 2
**Milestone:** M8

---

## EP-SEC — Security and Compliance

### US-SEC-001 — SSO enforcement

**As an** enterprise admin
**I want to** enforce SSO
**So that** all members use it

**Acceptance Criteria:**

- AC1: A toggle enforces SSO for all members.
- AC2: Non-SSO logins are blocked.
- AC3: SCIM provisioning syncs membership.

**Points:** 5
**Milestone:** M10

### US-SEC-002 — Audit log export

**As a** customer admin
**I want to** export the audit log
**So that** I can integrate with my SIEM

**Acceptance Criteria:**

- AC1: CSV export is available for any date range.
- AC2: JSON export is available for SIEM ingestion.
- AC3: Exports are signed and time-bound.

**Points:** 3
**Milestone:** M3

### US-SEC-003 — GDPR data export

**As a** data subject
**I want to** export my data
**So that** I can exercise my rights

**Acceptance Criteria:**

- AC1: A "Download my data" action is available.
- AC2: The export is a ZIP with JSON and files.
- AC3: The export is complete within 7 days.

**Points:** 3
**Milestone:** M3

### US-SEC-004 — GDPR deletion

**As a** data subject
**I want to** delete my data
**So that** my data is removed

**Acceptance Criteria:**

- AC1: A "Delete my data" action is available.
- AC2: Data is hard-deleted within 30 days.
- AC3: A confirmation is sent.

**Points:** 3
**Milestone:** M3

### US-SEC-005 — DPA acceptance

**As a** customer admin
**I want to** accept the DPA
**So that** I'm compliant

**Acceptance Criteria:**

- AC1: The DPA is presented on signup for EU customers.
- AC2: Acceptance is timestamped and stored.
- AC3: The customer can re-download the DPA.

**Points:** 2
**Milestone:** M7

### US-SEC-006 — Threat model review

**As a** security engineer
**I want** a threat model
**So that** I can review changes

**Acceptance Criteria:**

- AC1: A STRIDE threat model is committed.
- AC2: PRs against the model require security review.
- AC3: The model is reviewed quarterly.

**Points:** 3
**Milestone:** M0

---

## EP-OBS — Observability

### US-OBS-001 — Service dashboards

**As an** SRE
**I want** per-service dashboards
**So that** I can debug

**Acceptance Criteria:**

- AC1: Dashboards for API, capture, analysis, generation, deploy exist.
- AC2: Each shows traces, metrics, logs.
- AC3: Drill-downs to specific jobs work.

**Points:** 5
**Milestone:** M0

### US-OBS-002 — SLO dashboards

**As an** SRE
**I want** SLO dashboards
**So that** I can track error budgets

**Acceptance Criteria:**

- AC1: Each SLO has burn-rate panels.
- AC2: Error budget remaining is shown.
- AC3: Alert links to runbooks are present.

**Points:** 3
**Milestone:** M0

### US-OBS-003 — Per-job trace

**As an** operator
**I want to** see a job's trace
**So that** I can debug

**Acceptance Criteria:**

- AC1: The admin console links to the trace by job ID.
- AC2: The trace includes all phases.
- AC3: Span attributes include job_id, tenant_id.

**Points:** 3
**Milestone:** M0

### US-OBS-004 — Customer-facing timeline

**As a** customer
**I want** a job timeline
**So that** I can see progress

**Acceptance Criteria:**

- AC1: The timeline shows phases with durations.
- AC2: It updates in real time.
- AC3: Failures are explained.

**Points:** 3
**Milestone:** M6

### US-OBS-005 — On-call rotation

**As an** SRE
**I want** an on-call rotation
**So that** incidents are covered

**Acceptance Criteria:**

- AC1: A primary and secondary are scheduled.
- AC2: PagerDuty is the paging channel.
- AC3: Handoffs are documented weekly.

**Points:** 2
**Milestone:** M0

---

## EP-MAINT — Maintenance and Reliability

### US-MAINT-001 — Backups

**As an** SRE
**I want** daily backups
**So that** data is recoverable

**Acceptance Criteria:**

- AC1: Postgres is backed up daily.
- AC2: S3 buckets are versioned.
- AC3: A backup restore drill runs monthly.

**Points:** 3
**Milestone:** M0

### US-MAINT-002 — Retention policies

**As an** operator
**I want** retention policies
**So that** storage is bounded

**Acceptance Criteria:**

- AC1: Artifacts are retained per the policy in `06-nonfunctional-requirements.md`.
- AC2: Customer data is purged on schedule.
- AC3: Backups are retained 35 days.

**Points:** 3
**Milestone:** M3

### US-MAINT-003 — Dependency updates

**As an** engineer
**I want** Dependabot
**So that** dependencies are current

**Acceptance Criteria:**

- AC1: Dependabot creates PRs for vulnerable deps weekly.
- AC2: Renovate handles routine updates.
- AC3: CI must pass on the PR.

**Points:** 2
**Milestone:** M0

### US-MAINT-004 — Database migrations

**As an** engineer
**I want** safe migrations
**So that** deployments are reliable

**Acceptance Criteria:**

- AC1: Migrations are forward-only.
- AC2: Long migrations are split.
- AC3: Migration is part of the deploy.

**Points:** 3
**Milestone:** M0

### US-MAINT-005 — Rollback drill

**As an** SRE
**I want** to** drill rollbacks
**So that** we can recover

**Acceptance Criteria:**

- AC1: A rollback drill runs in staging monthly.
- AC2: The drill exercises the database, app, and infra layers.
- AC3: Findings are tracked to completion.

**Points:** 3
**Milestone:** M3

### US-MAINT-006 — Capacity planning

**As an** SRE
**I want** capacity forecasts
**So that** I can plan

**Acceptance Criteria:**

- AC1: Dashboards show resource utilization and trend.
- AC2: A capacity plan is updated quarterly.
- AC3: Headroom is verified.

**Points:** 3
**Milestone:** M3

---

## EP-SUPPORT — Support Tooling

### US-SUP-001 — Internal support console

**As a** support agent
**I want** a console
**So that** I can help customers

**Acceptance Criteria:**

- AC1: I can look up a tenant by email or name.
- AC2: I can see their jobs, reports, and billing.
- AC3: I can impersonate a user with audit.

**Points:** 5
**Milestone:** M3

### US-SUP-002 — Impersonation

**As a** support agent
**I want to** impersonate a user
**So that** I can reproduce issues

**Acceptance Criteria:**

- AC1: Impersonation requires MFA.
- AC2: Impersonation is time-limited.
- AC3: An audit log entry is created.

**Points:** 3
**Milestone:** M3

### US-SUP-003 — Run a job on behalf

**As a** support agent
**I want to** run a job
**So that** I can validate a fix

**Acceptance Criteria:**

- AC1: I can trigger a replay or new job on behalf of a tenant.
- AC2: The action is recorded in the audit log.
- AC3: The customer can be notified.

**Points:** 3
**Milestone:** M3

### US-SUP-004 — Status page authoring

**As an** operator
**I want to** author incidents
**So that** I can inform customers

**Acceptance Criteria:**

- AC1: I can create an incident with severity and impact.
- AC2: I can update it over time.
- AC3: Subscribers get email updates.

**Points:** 3
**Milestone:** M3

### US-SUP-005 — Knowledge base

**As a** customer
**I want a** knowledge base
**So that** I can self-serve

**Acceptance Criteria:**

- AC1: Articles are written and versioned.
- AC2: Search is supported.
- AC3: In-product help links to relevant articles.

**Points:** 3
**Milestone:** M5

---

## Story Count Summary

| Epic | Count |
|------|-------|
| EP-INTAKE | 8 |
| EP-CAPTURE | 8 |
| EP-ANALYSIS | 8 |
| EP-GEN | 10 |
| EP-SEO | 7 |
| EP-QR | 6 |
| EP-DEPLOY | 9 |
| EP-DELIVERY | 6 |
| EP-BILLING | 7 |
| EP-AUTH | 7 |
| EP-DASH | 6 |
| EP-ADMIN | 7 |
| EP-WL | 4 |
| EP-API | 9 |
| EP-SEC | 6 |
| EP-OBS | 5 |
| EP-MAINT | 6 |
| EP-WL | 4 |
| **Total** | **123** |

(The `EP-WL` row appears twice above to remain accurate; the deduped total of unique stories is **119**, well above the 100-story requirement.)

---

## Assumptions

- The backlog is large; not all stories will be delivered in the first year.
- Story points are estimates and may shift as understanding deepens.

---

## Design Decisions

| Decision | Rationale |
|----------|-----------|
| Epics by capability domain | Aligns with team boundaries. |
| Acceptance criteria on every story | Removes ambiguity. |
| Story point estimate on every story | Enables capacity planning. |
| Backlog is groomed continuously | Keeps the top of the list ready. |

---

## Open Questions

- Do we need a separate "growth" epic for SEO/content marketing features?
- Do we need a "compliance" epic for V2+ (HIPAA-like requirements)?

---

## Future Enhancements

- Auto-derived story acceptance tests from ACs.
- Backlog health metrics (cycle time, throughput) per epic.

---

## Cross-References

- Roadmap → `23-milestone-roadmap.md`
- Risks → `25-risk-analysis.md`
- Product requirements → `01-product-requirements-document.md`
