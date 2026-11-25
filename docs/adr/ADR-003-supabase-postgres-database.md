# ADR-003 · Supabase Postgres as the database; reuse existing tables

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-03 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Data storage for both the Coach web app and the Android client app |
| **Risk tier** | High (personal and health-related data, multi-tenant) |

## Context

The Android app already stores its data in Supabase Postgres (existing tables include `workout`, `nutrients` and `exercise`). With ADR-001, the Coach web app uses the same API and therefore needs the same data: what a coach plans appears in the client app, and what a client logs appears in the web app. The Coach app also needs new data: gyms, coaches, programs, calendar activities, notifications and so on.

**Requirements and constraints**
- One source of truth for both apps; no data copying between systems.
- Tenant isolation by `gym_id` (ADR-005) with database-level protection as a second line of defence.
- Changes must not break installed Android builds.
- Low cost while building; predictable cost at go-live (ADR-017).

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. Keep Supabase Postgres; reuse tables; add new ones | B. Separate database for coach data | C. Migrate everything to another database (e.g. Cloud SQL, Firestore) |
|---|---|---|---|
| Security / privacy gate | Pass (RLS available) | Pass | Pass, but needs a full re-audit |
| Security (20%) | 4 | 4 | 4 |
| Maintainability (20%) | 4 | 2 | 3 |
| Reliability (15%) | 4 | 2 | 3 |
| Performance (15%) | 4 | 3 | 4 |
| Scalability (10%) | 4 | 3 | 4 |
| Cost (10%) | 4 | 3 | 2 |
| Integration (10%) | 5 | 1 | 2 |
| **Weighted score** | **4.10** | **2.65** | **3.25** |

Notes on the scores:
- **A** loses a point on cost and reliability only because the free plan pauses inactive projects and limits egress; the plan is to move to Supabase Pro at go-live (ADR-017).
- **B** needs the shared data (clients, workouts) kept in sync between two databases, which is the main source of drift and bugs.
- **C** means migrating live data and rewriting parts of the live Android API, with no business benefit identified.

## Decision

**Option A.** Keep Supabase Postgres as the single database. Reuse the existing tables (`workout`, `nutrients`, `exercise`) and create the new tables for the Coach app in the same Supabase account.

## Why

- Already in use by the Android app; one source of truth.
- Postgres supports row-level security, transactions, indexes and migrations, which the coding standards require.
- No data migration, no sync jobs, lowest cost and risk.

## Rejected

- **B. Separate database:** duplicates and syncs shared data; two places to secure, back up and restore.
- **C. Migrate to another database:** high effort and risk for no identified benefit.

## Consequences

- All schema changes go through versioned, reviewed, reversible migrations (ADR-014).
- New and existing tables holding gym data need a `gym_id`, with indexes on `gym_id`, `client_id`, `coach_id` and date columns (ADR-005).
- Existing tables are shared with the live Android app, so changes must be backward-compatible (add columns and tables; avoid renames and drops without a migration plan).
- Only synthetic data in dev and test; never copy real client data (see `docs/STANDARD_IT_TESTING_RULES.md`, TDATA rules).
- Backups must be automated and restores tested before go-live.
- Supabase Pro is needed at go-live (ADR-017).

## Related decisions

ADR-001 (shared API), ADR-004 (API-only access), ADR-005 (multi-tenancy), ADR-013 (media storage), ADR-014 (backend standards), ADR-017 (cost plan).

## Open points

- Review the existing tables for `gym_id` readiness: which already have it, which need a backfill migration.
- Confirm the separate Supabase projects for development, staging and production (dev/prod parity).
- Confirm the Supabase region and any data-protection requirements for the personal and health data of users in India.
