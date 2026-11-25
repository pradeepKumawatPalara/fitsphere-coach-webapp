# ADR-021 · The calendar lives inside each client (not a top-level tab)

| | |
|---|---|
| **Status** | 🟡 Proposed (needs confirmation with the product owner; not yet accepted) |
| **Log entry** | D-21 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner / product owner *(approval pending)* |
| **Scope** | Navigation structure and where calendar data is loaded |
| **Risk tier** | Medium |

## Context

An early scope note listed the tabs as Dashboard, Clients, Calendar, Programs. The requirements document (`docs/FITSPHERE_COACH_WEBAPP.md`) lists Dashboard, Clients, Coaches, Programs, with the calendar **inside** each client. This affects routes, API design and permissions.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review. Scores do not capture product or user-research input, which decides this question.

| Criterion (weight) | A. Calendar inside each client | B. Top-level Calendar tab across clients | C. Both |
|---|---|---|---|
| Security / privacy gate | Pass | Pass, with wider data exposure per request | Pass |
| Security (20%) | 4 | 3 | 3 |
| Maintainability (20%) | 4 | 3 | 2 |
| Reliability (15%) | 4 | 3 | 3 |
| Performance (15%) | 4 | 2 | 3 |
| Scalability (10%) | 4 | 2 | 3 |
| Cost (10%) | 4 | 3 | 2 |
| Integration (10%) | 4 | 3 | 3 |
| **Weighted score** | **4.00** | **2.75** | **2.70** |

## Proposed decision

**Option A.** Follow the requirements document: the calendar opens from a client's profile, and Coaches is a top-level tab. Top-level tabs: Dashboard, Clients, Coaches, Programs.

## Why

- Matches the requirements document and the "per client" nature of plans and threads.
- One client and one date range per request: natural permission boundary, better performance (INP) and simpler API.

## Rejected

- **B. Top-level calendar:** multi-client queries are heavier and expose more data per request.
- **C. Both:** two calendar experiences to build, test and keep consistent.

## Consequences

- Routes such as `/clients/$clientId/calendar`; API such as `/clients/{id}/calendar/activities` loading one week or month range at a time.
- Bulk work across clients (assigning a program to many clients) lives in Programs, not in the calendar.
- A gym-wide schedule view would be a future feature (not in MVP).

## Related decisions

ADR-009, ADR-018 (design), ADR-019 (scope), ADR-020.

## Open points

- **Confirm with the product owner** that coaches do not need a cross-client calendar in the MVP.
- Check that the design artifact's navigation matches this decision.
