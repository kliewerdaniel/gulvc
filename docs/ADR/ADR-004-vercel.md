# ADR-004 — Vercel as the Deploy and Frontend Host

- **Status:** Accepted
- **Date:** 2026-06-01
- **Deciders:** Deployment team, Eng leads
- **Supersedes:** —
- **Superseded by:** —

---

## Context

Vibe deploys two distinct types of web application:

1. **The Vibe dashboard** (the customer-facing product UI we operate).
2. **Customer modernized sites** (the output of every job, owned by the customer).

Both must be:

- Fast at the edge.
- Operationally simple.
- Affordable at MVP scale.
- Friendly to the customer's GitHub repo (which is the source of truth).
- Optionally white-labelable for enterprise customers.

The deployment platform choice must satisfy both use cases, with the option to host the dashboard and customer sites on separate Vercel teams for isolation.

Alternatives: Vercel, Netlify, Cloudflare Pages + Workers, AWS (S3 + CloudFront + Lambda), Render, Fly.io, self-hosted on Kubernetes.

---

## Decision

Adopt **Vercel** as the primary host for both the dashboard and the modernized customer sites.

- The **dashboard** runs on the Vibe Vercel team.
- **Customer sites** run on a Vibe-managed Vercel team by default, with an option (V2) to attach a customer-owned Vercel team.
- A **GitHub App** (Vibe Platform App) creates and pushes the customer's repo; a second app (Vibe Connect, V2) supports tenant-installed deployments for white-label.
- **Vercel tokens** are JIT-minted per job and stored encrypted in our database.

---

## Rationale

### Why Vercel for both

- **Best-in-class Next.js support.** It is the platform Next.js was designed for; deployments are essentially zero-config.
- **Preview URLs** are free and unique per deploy — the perfect mechanism for the "preview before approve" UX.
- **ISR, on-demand revalidation, and edge functions** are first-class.
- **Image optimization** is first-class.
- **Observability** is built in (deploy logs, runtime logs, Web Vitals).
- **Custom domains** are managed by Vercel; DNS verification is automatic.
- **Pricing** is acceptable for our model: free tier covers previews; Pro scales with us.

### Why not Netlify

- **Pro:** mature, multi-framework.
- **Con:** Next.js experience is weaker than Vercel's; some Next 14 features (RSC, edge) are less first-class.
- **Verdict:** Vercel is the better Next.js home; the difference matters at the edge.

### Why not Cloudflare Pages + Workers

- **Pro:** excellent edge performance, generous pricing.
- **Con:** Next.js support is improving but still requires more glue; image optimization requires a separate service (Cloudflare Images); custom domains have more friction; ecosystem is smaller for our SMB customers.
- **Verdict:** strong alternative, but the operational gap is not worth it at MVP.

### Why not pure AWS (S3 + CloudFront + Lambda)

- **Pro:** ultimate control, no vendor lock-in.
- **Con:** vastly more operational overhead; we would reinvent Vercel; image optimization, ISR, edge functions, custom domain management all need custom work.
- **Verdict:** rejected. We use AWS for the data plane, Vercel for the experience plane.

### Why not Render or Fly.io

- **Verdict:** good platforms for some workloads, but neither is a Vercel-equivalent for Next.js. Rejected.

### Why not self-hosted Kubernetes

- **Verdict:** rejected. The team size and operational discipline do not justify a Kubernetes platform for the edge tier.

### Why two GitHub Apps

- **Vibe Platform App** is installed by us; it creates repos on the customer's behalf under our org or theirs (depending on permissions). This is the default flow.
- **Vibe Connect App** (V2) is installed by the customer; it lives in their org and uses their identity. This supports white-label and stricter enterprise requirements.
- Two apps keep permissions narrowly scoped and the failure modes small.

### Why per-job Vercel tokens

- A long-lived Vercel token is a security risk. We mint a token per deployment, scope it narrowly, and revoke it on completion. This keeps the blast radius small.

### Why a separate Vercel team per tenant (V2)

- For enterprise customers, the customer may want their projects isolated under their own Vercel team. We support this in V2 via Vibe Connect.

---

## Consequences

### Positive

- Deployments are fast and predictable.
- Preview URLs are free and high-quality.
- Customer sites are easy to inspect, easy to roll back, easy to promote.
- The Next.js expertise on the team transfers directly.

### Negative

- **Vendor alignment with Vercel.** Mitigated by the fact that we can move the dashboard to AWS if needed; customer sites can move anywhere because they own the repo and the Vercel project can be transferred.
- **Vercel pricing** can climb with traffic. Mitigated by clear per-customer limits and observability of usage.
- **Per-job token minting** adds latency. Mitigated by a fast token cache for short-lived reuse within a job.

### Neutral

- We accept the Vercel/North-America-centric default; multi-region is on the V3 roadmap.

---

## Alternatives Considered

| Option | Verdict |
|--------|---------|
| Vercel | **Chosen** |
| Netlify | Rejected — weaker Next.js fit |
| Cloudflare Pages + Workers | Rejected — more glue, weaker Next.js fit |
| AWS (S3+CF+Lambda) | Rejected — high operational cost |
| Render / Fly.io | Rejected — not Next.js-first |
| Self-hosted Kubernetes | Rejected — operational mismatch |

---

## Notes

- We will revisit the choice at V3 if Vercel pricing or reliability changes materially.
- A multi-CDN fallback (V3) may be introduced for global delivery.
- The Vercel API surface area we depend on is small and well-documented; switching to a different host is bounded.
- Vibe Connect (white-label GitHub App) is V2; the underlying decision (GitHub as the source of truth) is final.
