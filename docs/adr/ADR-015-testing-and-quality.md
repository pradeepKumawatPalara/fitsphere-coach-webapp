# ADR-015 · Testing and quality tooling

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-15 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Test tools and automated quality gates for the web app and the API |
| **Risk tier** | High (the app stores personal data; see `docs/STANDARD_IT_TESTING_RULES.md`) |

## Context

The testing standard requires test strategy, risk-based coverage, tenant-isolation tests, accessibility checks and CI quality gates. We need tools that fit Vite and TypeScript and run in GitHub Actions.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. Vitest + React Testing Library + MSW; Vitest + Supertest (API); Playwright; axe; ESLint + Prettier | B. Jest + Cypress | C. Minimal automated testing, mostly manual |
|---|---|---|---|
| Security / privacy gate | Pass | Pass | **Fail:** high risk tier needs automated regression and tenant-isolation tests |
| Security (20%) | 4 | 4 | n/a |
| Maintainability (20%) | 4 | 3 | n/a |
| Reliability (15%) | 5 | 4 | n/a |
| Performance (15%) | 4 | 3 | n/a |
| Scalability (10%) | 4 | 4 | n/a |
| Cost (10%) | 5 | 5 | n/a |
| Integration (10%) | 5 | 3 | n/a |
| **Weighted score** | **4.35** | **3.65** | excluded (failed gate) |

## Decision

**Option A.**
- Frontend: Vitest + React Testing Library + MSW.
- API: Vitest + Supertest against local Supabase (a real test database, not mocks).
- End-to-end: Playwright.
- Accessibility: axe (WCAG 2.2 AA) in component and E2E tests.
- Lint and format: ESLint with `@typescript-eslint/no-deprecated`, plus Prettier.
- All run in GitHub Actions.

## Why

- Vitest shares the Vite config; fast feedback.
- Testing Library and Playwright test user-visible behaviour, which survives refactoring.
- Matches the test pyramid and quality gates in `docs/best-practices/02-testing-and-unit-testing.md`.

## Rejected

- **B. Jest + Cypress:** workable, but a second toolchain next to Vite, and Playwright has better multi-browser and auto-waiting.
- **C. Mostly manual:** fails the testing standard for a high-risk app.

## Consequences

- CI gates in order: lint and format, typecheck, unit tests, integration/API tests, dependency and security scans, build, E2E on the built app; merge only when all pass.
- Mandatory tenant-isolation test for every endpoint; synthetic seed data only.
- Key E2E journeys: login, add activity to the calendar, move activity, build a superset, assign a program to many clients, export CSV.
- No test is deleted, skipped or weakened to make it pass; flaky tests are fixed, not retried until green.
- Release sign-off follows the testing standard (REL rules) and is a business decision.

## Related decisions

ADR-005, ADR-010, ADR-014, ADR-016.

## Open points

- **Package manager:** `CLAUDE.md` uses `npm` commands while some best-practice examples use `pnpm`; confirm one and record it.
- Test environments: staging with seeded synthetic data and test accounts for E2E.
- Contract tests between the web app, the API and the Android app (see `docs/best-practices/02-testing-and-unit-testing.md`).
- A written test strategy per the testing standard (scope, entry and exit criteria) to be produced before the first feature.
