# ADR-007 · Hosting and CI/CD: Hostinger (web), Google Cloud Run (API), GitHub Actions

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-07 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Where the web app and API run, where secrets live, how releases are built and deployed |
| **Risk tier** | High (production hosting of personal data; secrets) |

## Context

ADR-006 produces static files; ADR-001 keeps the existing Node.js API on Google Cloud. We need minimum or zero hosting cost while handling the expected traffic, with safe handling of secrets and a repeatable release process.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. Web on Hostinger; API on Cloud Run | B. Everything on Google Cloud (web on Firebase Hosting or Cloud Storage + CDN) | C. Web on Vercel/Netlify; API on Cloud Run |
|---|---|---|---|
| Security / privacy gate | Pass, if security headers can be set (see Open points) | Pass | Pass |
| Security (20%) | 3 | 4 | 4 |
| Maintainability (20%) | 4 | 4 | 4 |
| Reliability (15%) | 3 | 4 | 4 |
| Performance (15%) | 3 | 4 | 4 |
| Scalability (10%) | 4 | 5 | 5 |
| Cost (10%) | 5 | 4 | 3 |
| Integration (10%) | 4 | 3 | 3 |
| **Weighted score** | **3.60** | **4.00** | **3.90** |

**Architect note:** the score favours option B. The owner's decision for A rests mainly on using the Hostinger plan already paid for. The decision stands; the open points below say when to revisit it.

## Decision

- Web app: **Hostinger** (existing plan, on a subdomain), static files with long cache headers.
- API: **Google Cloud Run** (auto-scales, scales to zero; free tier covers roughly 2 million requests a month, to be re-verified).
- Secrets: **Google Secret Manager**.
- CI/CD: **GitHub Actions** (lint → typecheck → test → build → deploy).

## Why

- Minimum or zero hosting cost while handling the expected traffic.
- No new vendor for the web tier; Cloud Run suits an API with spiky load.

## Rejected

- **B. All on Google Cloud:** stronger on headers, rollback and CDN, but adds a service when a paid plan already exists. First alternative if Hostinger falls short.
- **C. Vercel/Netlify:** extra vendor and free-tier terms to check for commercial use.

## Consequences

- SPA fallback rule (all routes to `index.html`) must be configured on Hostinger.
- Deploy credentials (SFTP/SSH) live in GitHub encrypted secrets, never in the repo; separate staging and production targets.
- Keep the last few build artefacts so rollback is a re-deploy of a previous build (build, release and run stay separate stages).
- Google Cloud budget alerts from day one (ADR-017).
- Run an automated smoke test after each deployment.

## Related decisions

ADR-001, ADR-006, ADR-014, ADR-015, ADR-017.

## Open points

- **Can Hostinger send the required security headers** (CSP, HSTS, X-Content-Type-Options, frame-ancestors) and long-cache headers? If not, this decision should be revisited in favour of option B.
- Staging environment: a second subdomain on Hostinger, and a separate Cloud Run service and Supabase project.
- Confirm Hostinger's uptime commitment is acceptable for the business.
