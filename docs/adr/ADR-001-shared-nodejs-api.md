# ADR-001 · One shared Node.js API for the web app and the Android app

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-01 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Backend architecture; affects both the Coach web app and the Android client app |
| **Risk tier** | High (personal data, multi-tenant) |

## Context

The FitSphere Android client app is live and already uses a Node.js API hosted on Google Cloud, backed by Supabase Postgres. The Coach web app works on the same data: a coach assigns a workout, the client logs it; client autonomy, streaks and "Needs attention" tags depend on both sides. We must decide where the Coach web app's server-side logic lives.

**Requirements and constraints**
- Both apps must apply identical business rules (streaks, tagging, autonomy).
- Tenant isolation (`gym_id`) and role checks must be enforced in one place (OWASP A01).
- Installed Android builds must keep working (backward compatibility).
- Budget and team size favour the smallest number of services.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first**; a high score cannot outweigh a fail. Scores are the architect's proposal for owner review.

| Criterion (weight) | A. Extend the existing API | B. Separate coach API, same DB | C. Rewrite the API |
|---|---|---|---|
| Security / privacy gate | Pass | Pass, with duplicated rule risk | Pass |
| Security (20%) | 4 | 2 | 3 |
| Maintainability (20%) | 4 | 2 | 3 |
| Reliability (15%) | 4 | 3 | 2 |
| Performance (15%) | 4 | 4 | 4 |
| Scalability (10%) | 4 | 4 | 4 |
| Cost (10%) | 5 | 3 | 1 |
| Integration (10%) | 5 | 2 | 2 |
| **Weighted score** | **4.20** | **2.75** | **2.80** |

## Decision

**Option A.** Extend the existing Node.js API with coach modules and use it for both apps.

## Why

- One place for business rules and security, so both apps always behave the same.
- No duplicate auth, hosting, deployment or testing setup.
- Lowest cost and fastest path: the API is already live.

## Rejected

- **B. Separate coach API on the same database:** rules written twice and drifting apart; two services writing the same tables.
- **C. Rewrite the API from scratch:** it is already live; only worth doing if the existing backend is seriously flawed (no evidence of that so far).

## Consequences

- Changes to the API must stay backward-compatible for installed Android builds.
- Plan to move the backend out of the Android repo into its own repo (`fitsphere-api`) with care, because its deploy pipeline is tied to the current repo.
- The web app's typed client is generated from the API's OpenAPI spec (see ADR-002).
- Every endpoint needs a cross-gym test (see ADR-005).

## Related decisions

ADR-002 (separate web repo), ADR-004 (API-only access), ADR-005 (multi-tenancy), ADR-014 (backend standards).

## Open points

- Confirm with the tech lead that the existing backend is sound enough to extend (this is the assumption behind rejecting option C).
- Confirm whether the Android app's current auth provider constrains ADR-022.
