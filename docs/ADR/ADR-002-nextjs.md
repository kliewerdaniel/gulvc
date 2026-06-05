# ADR-002 — Next.js 14 as the Modernization Output

- **Status:** Accepted
- **Date:** 2026-06-01
- **Deciders:** Generation team, Eng leads
- **Supersedes:** —
- **Superseded by:** —

---

## Context

The platform modernizes customer websites into a maintained codebase. The output must be:

- A modern, accessible, performant web application
- Easy to deploy to standard hosts (Vercel preferred)
- Friendly to non-engineer customers (the source code is delivered to them)
- Composable and themeable, so the platform can produce varied output
- Self-hostable, so customers can leave Vercel if they want

The major alternatives considered for the output framework: Next.js 14, Astro 4, SvelteKit, Nuxt 3, and a custom static-site generator.

---

## Decision

Generate **Next.js 14** apps (App Router) as the default modernization output.

- **React Server Components** by default; client components only when necessary.
- **TypeScript** for all generated code.
- **Tailwind CSS 3.4** for styling with design tokens.
- **MDX** for blog-like content where applicable.
- **next/image** for image optimization.
- **next/font** for fonts.

The dashboard is also Next.js 14. Two reasons: shared component library, shared Vercel expertise.

---

## Rationale

### Why Next.js 14

- **Vercel is our deploy target.** Native fit, zero-config deploys, serverless functions, edge support, image optimization, ISR, on-demand revalidation.
- **RSC and Server Actions** reduce client bundle size and improve perceived performance.
- **File-based routing** is straightforward to generate from a navigation graph.
- **Mature ecosystem.** Customers can find engineers and answers.
- **Strong a11y defaults.** Vercel and the Next.js team invest in accessibility.
- **Image and font optimization** are first-class.
- **TypeScript** is first-class.

### Why not Astro 4

- **Pro:** outstanding performance for content-heavy sites; partial hydration.
- **Con:** smaller ecosystem; less familiar to mid-market engineering teams; no native Vercel/ISR parity that we need; harder to deliver a code-base a non-engineer can edit (mixing frameworks in islands complicates tooling).
- **Verdict:** Astro is an excellent content site framework, but the platform's output needs to support rich interactivity (forms, custom theming, blog comments in V3) with a single mental model. Next.js wins for our use case.

### Why not SvelteKit

- **Pro:** excellent DX, great performance.
- **Con:** smaller market share; harder to find engineers to maintain a customer's site; ecosystem immaturity in some areas (CMS integrations, analytics).
- **Verdict:** SvelteKit is a delight, but the customer's downstream is the deciding factor. React/Next wins on long-term maintainability for a non-Vibe audience.

### Why not Nuxt 3

- **Pro:** strong Vue ecosystem, especially in Europe.
- **Con:** smaller pool of mid-market engineers in North America; less synergy with our internal stack.
- **Verdict:** Vue has its fans, but our team and the broader SMB market default to React.

### Why not a custom SSG

- **Con:** we'd be reimplementing routing, SSR, image optimization, edge, ISR. Not a viable path.
- **Verdict:** not even considered seriously.

### Why App Router over Pages Router

- **RSC** is the future; choosing Pages would be a regressive decision.
- **Server Actions** simplify form handling for the customer.
- **Streaming** improves perceived performance on slower devices.
- **Caveat:** RSC brings complexity. We mitigate by keeping most routes static and using client components only when necessary (forms, theme pickers).

### Why Tailwind 3.4 over CSS Modules or styled-components

- **Tokens map cleanly to themes.** A theme is a small set of CSS variables, and Tailwind utilities consume them.
- **No runtime cost.** Tailwind is a compile-time tool.
- **Familiar to mid-market engineers.** Hire-able.
- **Lighthouse-friendly.** Generated CSS is small.

### Why not Tailwind 4

- Released in early 2025 but ecosystem compatibility is still settling at our build timeline. Tailwind 3.4 is stable and well-supported. We will revisit at V2.

---

## Consequences

### Positive

- Generated code is idiomatic, modern, and maintained by a large community.
- Vercel deploys are one-command and preview-by-default.
- Customers can edit the code with confidence that their changes will work.
- The dashboard shares a component library with the generated output, halving the surface area.

### Negative

- **React complexity.** RSC, Server Actions, and the metadata API are not trivial. Mitigated by a curated starter (`@vibe/starter-next`) and exhaustive scaffolding templates.
- **Tailwind purists may object.** We accept that; tokens + utilities are the right call for theme-driven generation.
- **Vendor alignment with Vercel.** Mitigated by supporting standard Next.js outputs that work on any Node host.

### Neutral

- The platform's output is "one stack", not a generator that emits many stacks. The trade-off is intentional: focus over flexibility.

---

## Alternatives Considered

| Option | Verdict |
|--------|---------|
| Next.js 14 | **Chosen** |
| Astro 4 | Rejected — better for content-only, worse for rich interactivity |
| SvelteKit | Rejected — smaller SMB market, lower hire-ability |
| Nuxt 3 | Rejected — non-default for our team and market |
| Custom SSG | Rejected — would reimplement the wheel |
| Hugo / Jekyll (static) | Rejected — modern sites need interactivity and forms |

---

## Notes

- We will revisit the choice at V5 in light of the multi-modal modernization vision.
- Generated code follows Next.js best practices, with documented opinionated defaults.
- A `@vibe/starter-next` package owns the canonical starter; generators do not write boilerplate by hand.
- The platform will keep dependencies pinned; Renovate handles updates; we test on every dependency bump.
