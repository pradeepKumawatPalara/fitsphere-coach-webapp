# ADR-014 · Backend standards: TypeScript, Zod + OpenAPI, Supabase CLI migrations

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-14 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | How the shared Node.js API is extended (backend lives in a separate repo; this records the standards the web app depends on) |
| **Risk tier** | High |

## Context

ADR-001 extends the existing Node.js API (currently JavaScript, in the Android repo). The web app needs a typed, documented API with one error format, safe database changes and logging.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. Keep the framework; TypeScript strict (file by file); Zod → OpenAPI; modules; Supabase CLI migrations | B. Stay in JavaScript and add routes | C. Rewrite on a new framework |
|---|---|---|---|
| Security / privacy gate | Pass | Pass, with weaker input typing | Pass |
| Security (20%) | 4 | 3 | 4 |
| Maintainability (20%) | 4 | 2 | 4 |
| Reliability (15%) | 4 | 3 | 2 |
| Performance (15%) | 4 | 4 | 4 |
| Scalability (10%) | 4 | 4 | 4 |
| Cost (10%) | 4 | 5 | 2 |
| Integration (10%) | 5 | 4 | 2 |
| **Weighted score** | **4.10** | **3.35** | **3.30** |

## Decision

**Option A.**
- Node.js LTS (pinned in `.nvmrc` and `engines`); TypeScript strict, converting existing JavaScript file by file.
- Keep the existing framework; add modules under `src/modules/{dashboard, clients, calendar, programs, coaches, notifications}` (routes → service → repository).
- Zod on every request; OpenAPI generated from Zod.
- Database access through the Supabase connection pooler.
- Migrations with Supabase CLI in the backend repo; `supabase gen types typescript` for shared types.
- One error format; structured JSON logs to Cloud Logging.
- Rate limits on login, bulk assign and CSV export; paginate every list.

## Why

- Incremental: no rewrite of a live system; the web app gets a typed contract.
- Matches the coding standard (validation at the boundary, reviewed and reversible migrations, versioned API).

## Rejected

- **B. Stay in JavaScript:** weaker type safety on a growing API.
- **C. Rewrite on a new framework:** high cost and risk for a live API.

## Consequences

- The OpenAPI spec is the contract; the web app generates its client from it (ADR-010).
- Version the API from day one (`/v1/`) because the Android app depends on it; keep changes backward-compatible.
- Use one error format for all endpoints (for example RFC 9457 Problem Details); idempotency keys on bulk and retryable operations; long jobs run in the background.
- Every endpoint has a cross-gym test (ADR-005).

## Related decisions

ADR-001, ADR-003, ADR-005, ADR-010, ADR-012, ADR-015.

## Open points

- **Confirm the existing framework** and Node version; the startup guide cites Fastify only as an example.
- Confirm whether an OpenAPI spec already exists, or must first be generated from Zod.
- When the backend moves to its own repo (ADR-001), apply these standards there and add its own `CLAUDE.md`.
