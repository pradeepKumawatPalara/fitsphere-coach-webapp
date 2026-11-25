# ADR-020 · Roles: Admin and Coach

| | |
|---|---|
| **Status** | 🟡 Proposed (needs confirmation; not yet accepted by the business owner) |
| **Log entry** | D-20 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(approval pending)* |
| **Scope** | Who can see and do what in the Coach web app |
| **Risk tier** | High (permissions protect personal data) |

## Context

The Dashboard question from review: "Do we have any roles?" The requirements say at minimum **Admin** (gym owner or manager, sees the whole gym, manages coaches and roles) and **Coach** (manages their own clients, calendars and programs). The Dashboard shows an admin the whole gym and a coach only their own clients. Permissions must be enforced on the server through one policy module (ADR-005).

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. Two roles: Admin, Coach | B. Three roles: Admin, Coach, Assistant coach | C. Fine-grained custom permissions per user |
|---|---|---|---|
| Security / privacy gate | Pass | Pass | Pass, with higher misconfiguration risk |
| Security (20%) | 4 | 4 | 3 |
| Maintainability (20%) | 5 | 3 | 2 |
| Reliability (15%) | 4 | 4 | 3 |
| Performance (15%) | 4 | 4 | 3 |
| Scalability (10%) | 3 | 4 | 5 |
| Cost (10%) | 5 | 4 | 2 |
| Integration (10%) | 4 | 3 | 3 |
| **Weighted score** | **4.20** | **3.70** | **2.90** |

## Proposed decision

**Option A** for the MVP: two roles. **Admin** sees the whole gym and manages coaches; **Coach** sees only their own clients. Dashboard content follows the role.

Design so a third role can be added later as a policy change, not a rewrite: store the role as data, and keep all checks in one policy module (role × action × resource).

## Why

- Smallest model that covers the stated needs; easy to test (every action × both roles, cross-gym always denied).
- The assistant-coach role is already out of MVP scope (ADR-019).

## Rejected

- **B. Assistant coach:** more rules and tests with no confirmed need.
- **C. Custom permissions:** flexible but complex and easy to misconfigure.

## Consequences

- Hide UI the role cannot use, but the API is the real check (ADR-004).
- Every role × action × resource combination needs a unit test, and every endpoint a cross-gym test.
- Admin-only actions (add coach, change roles, export all) are written to the audit log.

## Related decisions

ADR-004, ADR-005, ADR-012, ADR-019, ADR-022.

## Open points (need a product owner answer)

- **Confirm** whether an Assistant-coach role is needed in the MVP.
- Can an admin also coach clients? Who assigns clients to coaches, and what happens to a coach's clients when the coach is removed?
- Can a coach see other coaches' clients in any case (for example, cover for leave)?
- Do FitSphere support staff need a platform-level role, and how is cross-gym access controlled and audited?
