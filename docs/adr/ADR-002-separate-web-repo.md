# ADR-002 · Separate repository for the Coach web app

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-02 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Source-code organisation, CI/CD and access for the Coach web app |
| **Risk tier** | Medium (organisational; no direct data impact) |

## Context

ADR-001 keeps one shared Node.js API for both apps. We now need to decide where the Coach web app's code lives. Today there is one repository for the Android app, and the API's deploy pipeline is currently tied to it.

**Requirements and constraints**
- The web app uses a different stack (React, Vite, TypeScript) and release cycle from the Android app.
- It needs its own `CLAUDE.md`, CI pipeline and test setup (ADR-015, ADR-016).
- Clear ownership and access control; secrets must not be shared between unrelated codebases.
- The web app depends on the API only through its published contract (OpenAPI), not through shared source code (ADR-004).

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. New repo for the web app | B. Put the web app inside the Android app repo | C. One repo for web app + API |
|---|---|---|---|
| Security / privacy gate | Pass | Pass, with weaker access separation | Pass |
| Security (20%) | 4 | 2 | 3 |
| Maintainability (20%) | 4 | 2 | 3 |
| Reliability (15%) | 4 | 3 | 3 |
| Performance (15%) | 4 | 3 | 3 |
| Scalability (10%) | 4 | 3 | 3 |
| Cost (10%) | 4 | 4 | 3 |
| Integration (10%) | 3 | 4 | 5 |
| **Weighted score** | **3.90** | **2.80** | **3.20** |

Notes on the scores:
- **A** scores 3 on integration because the web app and API are in different repos; the contract must be kept in sync (see Consequences).
- **B** mixes two unrelated stacks, pipelines and instruction files; access to web code would also expose Android code.
- **C** scores best on integration but conflicts with ADR-001, which plans to move the API into its own repo (`fitsphere-api`), and ties two release cycles together.

## Decision

**Option A.** Create a new repository for the Coach web app (suggested name `fitsphere-coach-web`). The API stays in its own repository.

## Why

- Different stack, release cycle and `CLAUDE.md` from the Android app.
- Clear ownership, and access can be granted per repo.
- Independent CI/CD: a web release never waits on an Android build, and the reverse.
- Keeps the web app limited to calling the API through its contract.

## Rejected

- **B. Inside the Android app repo:** mixes unrelated stacks and pipelines, slows both teams and widens access.
- **C. One repo for web app + API:** couples release cycles and contradicts the planned move of the API to its own repo (ADR-001).

## Consequences

- The web app's typed API client is generated from the API's OpenAPI spec (`npm run gen:api`); it is never hand-written.
- API changes that affect the web app need a coordination step across two repos; contract tests (see `docs/best-practices/02-testing-and-unit-testing.md`) should catch breaking changes.
- The repo needs its own branch protection, CI (lint, typecheck, test, build) and secrets handling.
- No Supabase secret or service key is ever stored in this repo (ADR-004).

## Related decisions

ADR-001 (shared API), ADR-004 (API-only access), ADR-006 (frontend stack), ADR-007 (hosting and CI/CD), ADR-015 (testing).

## Open points

- Confirm the final repository name and the GitHub organisation that owns it.
- Confirm who gets access (business owner, tech lead, QA lead, security reviewer).
