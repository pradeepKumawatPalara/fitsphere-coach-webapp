# ADR-010 · Frontend data layer: TanStack Router / Query / Table + React Hook Form + Zod

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-10 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Routing, server state, tables and forms in the web app |
| **Risk tier** | Medium |

## Context

The app is data-heavy: lists, a calendar, forms and bulk actions, all backed by an API (ADR-004). We want type safety from the API to the UI, cached server data (fewer API calls, lower Supabase egress), and one pattern for forms and tables.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. TanStack Router + Query + Table, React Hook Form + Zod | B. React Router + Redux Toolkit (RTK Query) + Formik | C. React Router + SWR + native forms |
|---|---|---|---|
| Security / privacy gate | Pass | Pass | Pass |
| Security (20%) | 4 | 4 | 4 |
| Maintainability (20%) | 4 | 3 | 3 |
| Reliability (15%) | 4 | 4 | 3 |
| Performance (15%) | 4 | 3 | 4 |
| Scalability (10%) | 4 | 4 | 3 |
| Cost (10%) | 5 | 5 | 5 |
| Integration (10%) | 5 | 3 | 3 |
| **Weighted score** | **4.20** | **3.65** | **3.55** |

## Decision

**Option A.** TanStack Router (file-based routing), TanStack Query (server data and caching), TanStack Table (tables), React Hook Form + Zod (forms), and a typed API client generated from the backend's OpenAPI spec.

## Why

- Type-safe end to end; Zod schemas validate forms and infer types.
- Query caching cuts API calls, lowering Supabase egress cost.
- One consistent family of libraries that work well together.

## Rejected

- **B. Redux-based stack:** more boilerplate and a global store we do not need, since server state lives in Query.
- **C. SWR with native forms:** weaker form and table support; more custom code.

## Consequences

- Server state lives only in TanStack Query; do not copy API data into React state or a global store. Query keys per feature, for example `['clients', params]`.
- Never hand-write API types; run `npm run gen:api`. Never edit generated files (`src/lib/api/generated/`, `src/routeTree.gen.ts`).
- Every form has a Zod schema; infer types with `z.infer`.
- React Router is not used.

## Related decisions

ADR-004, ADR-006, ADR-011, ADR-014 (Zod and OpenAPI on the backend).

## Open points

- Choose the OpenAPI client generator (for example openapi-typescript, Orval or Hey API); this is a technology choice that needs its own decision entry.
- Confirm the current stable versions of TanStack Router and Query via Context7 before scaffolding, since both change often.
