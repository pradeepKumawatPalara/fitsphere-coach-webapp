# ADR-005 · Multi-tenancy by `gym_id`, enforced in the API and by row-level security

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-05 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Data isolation between gyms, for the API and the database |
| **Risk tier** | High (a failure exposes one gym's client data to another) |

## Context

Many gyms share one API and one database (ADR-001, ADR-003). A coach in gym A must never read or change gym B's data. Broken access control (OWASP A01) is the top risk for this app.

**Requirements and constraints**
- Isolation must hold even if one layer has a bug.
- `gym_id` must never come from the request; it comes from the authenticated user.
- Existing Android data is already in the database and needs to be mapped to gyms.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. `gym_id` on every table, enforced in the API and by RLS | B. `gym_id` enforced in the API only | C. Separate schema or database per gym |
|---|---|---|---|
| Security / privacy gate | Pass | Pass, but one bug leaks data (single line of defence) | Pass |
| Security (20%) | 5 | 3 | 5 |
| Maintainability (20%) | 4 | 4 | 2 |
| Reliability (15%) | 4 | 3 | 3 |
| Performance (15%) | 4 | 5 | 3 |
| Scalability (10%) | 4 | 4 | 2 |
| Cost (10%) | 5 | 5 | 2 |
| Integration (10%) | 4 | 4 | 2 |
| **Weighted score** | **4.30** | **3.90** | **2.90** |

## Decision

**Option A.** Every table that holds gym data has a `gym_id`. The API takes `gym_id` from the logged-in user (never from the request), checks permissions in one policy module (role × action × resource), and Supabase row-level security (RLS) acts as a second line of defence.

## Why

- Two independent layers: a bug in one does not automatically expose data.
- One policy module keeps permission rules in one place and testable.
- Cheaper and simpler to run than a schema or database per gym.

## Rejected

- **B. API only:** a single missed filter leaks data; no safety net.
- **C. Per gym schema or database:** heavy to migrate, monitor and back up for many small gyms.

## Consequences

- Every API endpoint needs a test proving a coach from another gym gets 403/404.
- Indexes on `gym_id`, `client_id`, `coach_id` and date columns.
- Repository and query helpers require `gym_id`; no query without it.
- Client-autonomy and role checks run on the server, not only in the UI.
- Audit log for admin actions (see `docs/best-practices/01-coding-best-practices.md`).

## Related decisions

ADR-001, ADR-003, ADR-004 (API-only access), ADR-014 (backend standards), ADR-020 (roles), ADR-022 (auth).

## Open points

- **RLS only helps if the API's database connection is subject to it.** A connection using the Supabase service key bypasses RLS. Decide how the API connects (for example, a restricted database role with the user's claims set per request) so RLS is a real second line of defence.
- How existing Android users and data map to a gym (backfill migration), and what `gym_id` a user with no gym gets.
- Whether internal FitSphere support staff need cross-gym access, and how that is controlled and audited.
