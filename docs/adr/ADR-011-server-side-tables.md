# ADR-011 · Tables: the server sorts, filters and pages; TanStack Table displays

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-11 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Client list and other data tables |
| **Risk tier** | High (large lists of personal data; interaction performance) |

## Context

The Clients list (and later Coaches, Programs, exercise library) can hold many rows per gym, with search, sort, filter and bulk select. INP is the most important Core Web Vital for this app, and bulk-select and CSV export touch personal data.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. Server-side sort/filter/paging via the API; TanStack Table in manual mode | B. Load all rows, sort/filter in the browser | C. Supabase Table Editor or another admin tool |
|---|---|---|---|
| Security / privacy gate | Pass | At risk: moves a whole gym's client data into the browser | **Fail:** a developer tool, bypasses app roles |
| Security (20%) | 5 | 2 | n/a |
| Maintainability (20%) | 3 | 4 | n/a |
| Reliability (15%) | 4 | 3 | n/a |
| Performance (15%) | 5 | 2 | n/a |
| Scalability (10%) | 5 | 1 | n/a |
| Cost (10%) | 4 | 5 | n/a |
| Integration (10%) | 4 | 3 | n/a |
| **Weighted score** | **4.25** | **2.85** | excluded (failed gate) |

## Decision

**Option A.** Sorting, filtering and pagination run in Supabase through the API (for example 25 rows per page). TanStack Table runs in server-side mode (`manualSorting`, `manualFiltering`, `manualPagination`) and handles display and bulk select.

## Why

- Scales to large gyms; only one page of data is sent to the browser (data minimisation).
- Fast interactions; consistent with "paginate everything".

## Rejected

- **B. Client-side:** fine for tiny lists, but slow and over-exposes data at scale.
- **C. Supabase Table Editor:** a developer and admin tool; not something coaches use.

## Consequences

- Every list endpoint supports pagination, filtering and sorting, with a maximum page size (never an unbounded list).
- Stable sort order (add a unique tie-breaker such as `id`) so pages do not repeat or skip rows.
- Indexes for each common filter (`gym_id`, `client_id`, date).
- Table state (page, sort, filters) is kept in the URL so views can be shared and restored.
- One table pattern everywhere (sort, filter, bulk select, CSV export), keyboard accessible.

## Related decisions

ADR-004, ADR-005, ADR-010, ADR-012 (CSV export), ADR-014.

## Open points

- Define "select all" across pages: for example, "select all N matching clients" resolved on the server, with a clear on-screen count.
- Default and maximum page sizes, and whether an exact total count is needed (it can be slow on large tables).
