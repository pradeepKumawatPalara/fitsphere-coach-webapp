# ADR-006 · Frontend: React + Vite + TypeScript, static build ("Stack 1")

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-06 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Foundation of the Coach web app frontend |
| **Risk tier** | Medium |

## Context

The Coach web app sits behind a login (so SEO and server-side rendering add little), is interaction-heavy (calendar, tables, bulk select) and must be cheap to host and fast to build. The team already pays for a Hostinger plan.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | Stack 1: React + Vite, static, on Hostinger | Stack 2: Next.js on Hostinger Node.js | Stack 3: Angular on Firebase |
|---|---|---|---|
| Security / privacy gate | Pass | Pass | Pass |
| Security (20%) | 4 | 3 | 4 |
| Maintainability (20%) | 4 | 3 | 3 |
| Reliability (15%) | 4 | 3 | 4 |
| Performance (15%) | 3 | 4 | 3 |
| Scalability (10%) | 5 | 3 | 5 |
| Cost (10%) | 5 | 3 | 4 |
| Integration (10%) | 4 | 3 | 2 |
| **Weighted score** | **4.05** | **3.15** | **3.55** |

Notes: per the decision log, Stack 2 needs a Hostinger Business or Cloud plan to run Node.js; Stack 3 does not use the existing hosting plan. A single-page app loads its JavaScript before rendering, so Largest Contentful Paint needs care (score 3).

## Decision

**Stack 1.** React + Vite + TypeScript (strict), built to static files.

## Why

- Behind a login, server rendering and SEO add nothing; static files are the cheapest and easiest to scale.
- Uses the hosting already paid for; fastest to build with Claude Code.
- No server to patch for the web tier, which reduces attack surface.

## Rejected

- **Stack 2 (Next.js):** runs a server and needs a higher hosting plan, for features we do not need.
- **Stack 3 (Angular on Firebase):** new hosting, different tooling, no advantage for this app.

## Consequences

- Use route-level code splitting and lazy loading to protect LCP and INP (Core Web Vitals targets in `docs/best-practices/03-design-guide-and-ui-design.md`).
- Single-page routing needs a fallback rule to `index.html` on the host (ADR-007).
- Only public `VITE_` variables are baked into the build; no secrets (ADR-004).
- TypeScript `strict: true`; ESLint and Prettier enforced in CI (ADR-015).
- Pin the Node.js LTS version (`.nvmrc`, `engines`).

## Related decisions

ADR-004, ADR-007 (hosting), ADR-008 (UI), ADR-010 (data layer), ADR-015 (testing).

## Open points

- **Revisit if** we need a backend-for-frontend layer or a public marketing site in the same app (then consider Next.js).
- Confirm browser support policy (current and previous versions of evergreen browsers is the recommendation).
