# FitSphere Coach — 2. Testing & Unit Testing

Part 2 of 5 in the FitSphere Coach engineering guide. Items marked *(Recommendation)* are architect guidance; the rest is backed by the references at the end.

## 1. Core principle — test like a user

- Testing Library's guiding principle: the closer tests are to how the software is actually used, the more confidence they give.
- Query the UI the way a user finds things: by role, label and visible text. Use `data-testid` only as a last resort.
- Avoid testing implementation details (component internal state, private functions). Tests should break only when behaviour breaks, not when code is refactored.
- React Testing Library recommends Mock Service Worker (MSW) to mock API calls declaratively, instead of stubbing `window.fetch`.

## 2. Unit testing rules *(Recommendation)*

- Test runner: Vitest (or Jest) with React Testing Library for components.
- One behaviour per test; name tests as sentences (`marks client as needing attention when a target is missed for 3 days`).
- Arrange – Act – Assert structure; no logic (loops, ifs) inside tests.
- Tests are independent and deterministic: fake timers and fixed dates for anything date-based (streaks, calendar weeks, time zones).
- Cover edge cases explicitly: empty states, time-zone boundaries, leap days, client with no programme, autonomy switched mid-week.
- Coverage is a signal, not a goal — aim for full coverage of business rules, not of UI boilerplate.

## 3. FitSphere unit-test priorities *(Recommendation)*

| Module | What to test |
|---|---|
| Streak calculation | Consecutive days, missed day resets, time zones, rest days |
| "Needs attention" tagging | Each rule that derives a tag from targets and activities set by the coach |
| Client autonomy | ON: client-logged workouts accepted; OFF: only coach-assigned workouts recorded |
| Calendar operations | Move, copy, delete, bulk select, superset grouping/ungrouping |
| Program assignment | Assigning own and prebuilt programs to many clients; partial failure handling |
| CSV export | Column order, escaping of commas/quotes, special characters in names |
| Permissions policy | Admin vs coach for every action; cross-gym access always denied |

## 4. Integration / API tests *(Recommendation)*

- Run against a real test database (e.g. Testcontainers or a disposable DB), not mocks.
- **Mandatory tenant-isolation test for every endpoint:** a coach from gym A must never read or change gym B's data.
- Test the error paths too (OWASP A10): invalid input, timeouts, duplicate submissions.

## 5. End-to-end tests — Playwright

- Playwright's first best practice: test user-visible behaviour.
- Use built-in locators; they auto-wait and retry, so no manual sleeps.
- Prefer role, text and test-id locators (the order Playwright's own generator uses); avoid CSS/XPath chains.
- Keep each test isolated with its own data; run in CI on every PR.
- Keep E2E to a small set of critical journeys *(Recommendation)*:
  - Coach logs in → Dashboard → opens a "Needs attention" client
  - Assign a program to multiple clients
  - Add, move and copy an activity on the client calendar (including the keyboard alternative to drag)
  - Export the client list to CSV
  - Admin adds a coach and assigns a role

## 6. Test distribution *(Recommendation)*

Many fast unit tests for business rules → a solid layer of API/integration tests (especially permissions) → a few E2E journeys. Add automated accessibility checks (e.g. axe) to component and E2E tests.

## 7. Testing with Claude Code

- Put the exact test commands in `CLAUDE.md` (run all, run one file) so Claude can check its own work.
- Ask for test-first work: "write a failing test that reproduces it, then fix it".
- Ask Claude for evidence — the command it ran and its output — not just "tests pass".
- Enforce with a Stop hook so a turn cannot end while tests or typecheck fail.

## 8. The test pyramid (Martin Fowler)

- Group tests by granularity and keep a balanced portfolio: many fast, low-level unit tests; fewer integration/service tests; very few broad end-to-end tests through the UI.
- Broad UI tests are slower, more brittle and more expensive to maintain, so push each check to the lowest level that can prove it.
- Avoid the "ice-cream cone" anti-pattern: mostly manual and UI tests, few unit tests.

| Level | Scope | Speed | Share (rough guide) |
|---|---|---|---|
| Unit | One function/component, collaborators faked | Milliseconds | Most |
| Integration / API | Component + real DB or service | Seconds | Some |
| End-to-end | Full stack through the browser | Minutes | Few |

## 9. Generic unit-testing principles (industry convention)

- **F.I.R.S.T.:** Fast, Independent, Repeatable, Self-validating, Timely.
- Test behaviour through the public interface, not private internals.
- One logical assertion per test, so a failure points to one cause.
- Use test-data builders/factories instead of copy-pasted fixtures.
- Use test doubles (stubs, fakes, mocks) only at real boundaries (network, clock, randomness); over-mocking hides bugs.
- No shared mutable state between tests; tests run in any order and in parallel.
- Every bug fix starts with a failing test that reproduces it (regression test).
- Test-Driven Development where it fits: red (failing test) → green (simplest code) → refactor.
- Flaky tests are bugs: quarantine immediately, fix within days, never "retry until green".
- Treat test code as production code: readable, reviewed, refactored.

## 10. Other test types to include *(Recommendation)*

| Type | Purpose in FitSphere Coach |
|---|---|
| Contract tests | Guarantee the API contract between the Coach webapp, the API and the client mobile app |
| Component tests | UI components in isolation (tables, calendar cells, forms) |
| Visual regression | Catch unintended layout changes on key screens |
| Accessibility tests | Automated axe checks + manual keyboard and screen-reader pass |
| Performance / load | Load-test bulk assign, CSV export, dashboard queries before launch |
| Security tests | SAST and dependency scanning on every PR; DAST on staging; OWASP ASVS as the checklist |
| Smoke tests | A few checks run right after each deploy |
| Exploratory / UAT | Real coaches try new features on staging before release |

## 11. Test data and environments *(Recommendation)*

- Synthetic seed data only; never copy real client personal data into test or dev.
- Deterministic seeds so every run starts from the same state.
- Ephemeral preview environment per PR where possible.
- Separate test database per worker so parallel tests don't collide.

## 12. Quality gates in CI *(Recommendation)*

1. Lint and format check
2. Typecheck
3. Unit tests (with coverage report)
4. Integration/API tests
5. Dependency and security scans
6. Build
7. E2E tests on the built app
8. Merge only when all pass; branch protection enforces it

## References

- Testing Library — Guiding Principles: https://testing-library.com/docs/guiding-principles/
- React Testing Library intro: https://testing-library.com/docs/react-testing-library/intro/
- Playwright — Best Practices: https://playwright.dev/docs/best-practices
- Martin Fowler — Test Pyramid: https://martinfowler.com/bliki/TestPyramid.html
- Ham Vocke — The Practical Test Pyramid: https://martinfowler.com/articles/practical-test-pyramid.html
- Martin Fowler — Software Testing Guide: https://martinfowler.com/testing/
- OWASP ASVS (security test basis): https://github.com/OWASP/ASVS
- Anthropic — Best practices for Claude Code (Give Claude a check to run; Stop hook): https://code.claude.com/docs/en/best-practices
