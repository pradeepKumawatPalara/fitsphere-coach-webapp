# ADR-004 · The web app talks only to the API, never directly to Supabase

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-04 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Trust boundary between the browser, the API and the database |
| **Risk tier** | High (access control for personal data across gyms) |

## Context

ADR-001 gives both apps one Node.js API, and ADR-003 keeps Supabase Postgres as the database. Supabase can also be called directly from a browser using a public key, with Row Level Security (RLS) policies deciding what each user may see. We must decide whether the web app may do that.

The risk that matters most here is OWASP A01, Broken Access Control: a coach must only see their own clients, an admin only their own gym, and a client-autonomy flag, audit logs and export limits must be enforced every time.

**Requirements and constraints**
- Tenant isolation, roles and business rules (streaks, "Needs attention" tags, autonomy) live in one place so both apps behave the same.
- Audit logging, rate limits (login, bulk assign, CSV export) and input validation must be enforced on the server.
- The browser is untrusted: anything shipped to it can be read.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. API only | B. Browser calls Supabase directly (RLS only) | C. Hybrid: API for writes, direct Supabase reads under RLS |
|---|---|---|---|
| Security / privacy gate | Pass | At risk: rules such as audit, rate limits and autonomy are hard to enforce in RLS alone | Pass only with conditions (two enforcement points to keep identical) |
| Security (20%) | 5 | 2 | 3 |
| Maintainability (20%) | 4 | 2 | 3 |
| Reliability (15%) | 4 | 3 | 3 |
| Performance (15%) | 3 | 4 | 4 |
| Scalability (10%) | 4 | 4 | 4 |
| Cost (10%) | 4 | 5 | 4 |
| Integration (10%) | 5 | 2 | 3 |
| **Weighted score** | **4.15** | **2.95** | **3.35** |

Notes on the scores:
- **A** loses a point on performance because every request takes an extra hop through the API; caching with TanStack Query reduces this (ADR-010).
- **B** is cheapest (no API compute) but splits business logic between the frontend and the database, and duplicates rules that the Android app already gets from the API.
- **C** keeps two ways to read the same data, so permission rules would need to be kept in sync in both places.

## Decision

**Option A.** The browser calls the Node.js API only. It never queries Supabase directly. The Supabase secret (service) key lives only in the API.

## Why

- One place for tenant isolation, roles and business rules; the API remains the real permission check and the UI only hides what a role cannot use.
- Server-side audit logging, rate limits and validation are possible.
- A browser-visible database key and wide-open query surface are avoided.
- Both apps go through the same rules.

## Rejected

- **B. Direct Supabase calls with RLS alone:** splits business logic between frontend and database; weak fit for audit, rate limits and rules like client autonomy.
- **C. Hybrid:** two read paths to secure and keep consistent; no strong benefit for the MVP.

## Consequences

- Every feature needs an API endpoint first. Frontend work can proceed in parallel using the OpenAPI contract and MSW mocks (ADR-002, ADR-015).
- No Supabase secret or service key in the web repo. Only public `VITE_` config, such as `VITE_API_BASE_URL`, is allowed in the frontend.
- The API's CORS allows only the web app's domain; the web app's Content Security Policy limits connections to the API (plus the signed upload URLs from ADR-013).
- Realtime features (for example, notifications) cannot use Supabase Realtime from the browser; use polling or an API-provided mechanism.
- Image uploads go straight to object storage via short-lived signed URLs issued by the API (ADR-013). That is not a Supabase call and does not break this rule.
- RLS stays on as a second line of defence (ADR-005), not as the primary check.

## Related decisions

ADR-001 (shared API), ADR-003 (database), ADR-005 (multi-tenancy), ADR-010 (data layer), ADR-013 (media), ADR-022 (auth provider, still Open).

## Open points

- If Supabase Auth is chosen in ADR-022, confirm the browser may use the Supabase SDK for sign-in only and never for data queries, and add a lint or review rule that blocks `@supabase/supabase-js` data calls.
- Decide how notifications reach the browser (polling interval or a push mechanism) before the Notifications feature is built.
