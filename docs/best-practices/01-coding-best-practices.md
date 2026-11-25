# FitSphere Coach — 1. Coding Best Practices

Part 1 of 5 in the FitSphere Coach engineering guide. Items marked *(Recommendation)* are architect guidance, not a published standard; everything else is backed by the references at the end.

## 1. Security baseline — OWASP Top 10:2025

The OWASP Top 10 is the industry awareness standard for the most critical web application security risks. The 2025 edition (finalised early 2026) is the current version.

| OWASP 2025 category | What it means for FitSphere Coach |
|---|---|
| A01 Broken Access Control | Highest risk for us. Every query must be scoped to the coach's gym (tenant). Admin vs coach permissions and "coach sees only assigned clients" must be checked on the server, never only hidden in the UI. |
| A02 Security Misconfiguration | No default credentials, strict CORS, security headers (CSP, HSTS), separate configs per environment, no debug output in production. |
| A03 Software Supply Chain Failures (new) | Commit lockfiles, run `npm audit` in CI, review every new dependency, pin CI actions, keep the dependency list small. |
| A04 Cryptographic Failures | TLS everywhere, hashed passwords via a proven library or managed auth provider, encrypt sensitive client data at rest. |
| A09 Security Logging & Alerting Failures | Audit log for admin actions (adding coaches, role changes, bulk program assignment, CSV export). |
| A10 Mishandling of Exceptional Conditions (new) | Handle every failure path explicitly: CSV export timeouts, partially failed bulk assignments, media upload failures. Fail closed on permission errors. |

## 2. Code review standard — Google Engineering Practices

- Approve a change once it clearly improves overall code health, even if it is not perfect; seek continuous improvement.
- Never approve a change that makes code health worse (emergencies excepted).
- Reviewers look at design, functionality, complexity, tests, naming, comments and documentation.
- Apply the same standard to code written by Claude — AI-generated code gets reviewed like any human PR.

## 3. Language and structure *(Recommendation)*

- TypeScript with `strict: true`; no `any` without a comment explaining why.
- One formatter (Prettier) and one linter (ESLint + typescript-eslint), enforced in CI.
- Feature-based folders mirroring the product: `dashboard/`, `clients/`, `coaches/`, `programs/`, `calendar/`, `exercise-library/`, `notifications/`.
- Validate every API input with a schema (e.g. Zod) at the boundary; derive TypeScript types from the schema.
- One shared error helper and error format; never throw raw strings.
- Business rules (streaks, "Needs attention" tagging, client autonomy) live in pure, framework-free modules so they are easy to unit test.

## 4. Multi-tenancy rules for FitSphere *(Recommendation)*

- Every table that holds gym data carries a `gym_id`; repository/query helpers require it.
- Consider database-level row security as a second line of defence.
- Permission checks centralised in one policy module (role × action × resource), not scattered across handlers.
- Client-autonomy toggle enforced on the server when the client app logs workouts.

## 5. Git and delivery hygiene *(Recommendation)*

- Small PRs, one concern each; branch protection on `main`.
- CI on every PR: typecheck, lint, unit tests, `npm audit`.
- No secrets in the repo; use environment variables and a secret manager. `.env` files are gitignored.
- Conventional commit messages and a CHANGELOG for releases.

## 6. Generic clean-code principles (industry convention)

- **Readability first:** code is read far more often than written. Clear names (`calculateStreak`, not `calc2`), no abbreviations, no magic numbers (use named constants).
- **Small, focused units:** a function does one thing; a module has one reason to change (Single Responsibility).
- **KISS and YAGNI:** the simplest design that works; don't build features or abstractions before they are needed.
- **DRY, within reason:** remove real duplication of knowledge, but don't merge code that only looks similar.
- **Prefer pure functions and immutability** for business logic; keep side effects (DB, network, time) at the edges.
- **Composition over inheritance;** depend on interfaces so parts can be swapped and tested (dependency injection).
- **Fail fast:** validate early, return early, never swallow errors silently.
- **Comments explain why, not what;** delete dead and commented-out code (git keeps history).
- **Consistency beats preference:** follow the existing pattern in the codebase unless the team agrees to change it.
- **Boy Scout rule:** leave touched code slightly cleaner than you found it, in the same PR only when small.

## 7. Secure coding checklist (OWASP ASVS 5.0)

OWASP ASVS is the detailed requirements standard behind the Top 10 (version 5.0.0, May 2025). It defines three verification levels; FitSphere stores personal and activity data of clients, so target **Level 2** *(Recommendation)*.

| Area | Practice |
|---|---|
| Input | Validate all input server-side against an allow-list schema; reject unknown fields |
| Output | Encode output by context to prevent XSS; never render user HTML without sanitising |
| Database | Parameterised queries / ORM only; no string-built SQL |
| Authentication | Proven auth provider or library; MFA for admins; rate-limit login; secure password reset |
| Sessions | Cookies `HttpOnly`, `Secure`, `SameSite`; short-lived tokens with rotation; logout invalidates server-side |
| Access control | Deny by default; check on every request; least privilege for roles and service accounts |
| CSRF | Tokens or SameSite cookies on state-changing requests |
| File uploads | Validate type and size, store outside the web root/in object storage, scan, serve via signed URLs (exercise images/videos) |
| Secrets | Never in code or logs; secret manager; rotate keys |
| Headers | CSP, HSTS, X-Content-Type-Options, frame-ancestors |
| Abuse | Rate limits on API, CSV export and bulk endpoints |
| Privacy | Collect only needed data; mask PII in logs; support export and deletion requests |

## 8. Architecture and operations (The Twelve-Factor App)

The Twelve-Factor App is a widely used methodology for building SaaS web apps:

1. **Codebase:** one repo per app, many deploys.
2. **Dependencies:** declared explicitly and isolated (`package.json` + lockfile).
3. **Config:** stored in environment variables, not in code.
4. **Backing services:** database, cache, storage treated as attached resources swappable by config.
5. **Build, release, run:** strictly separate stages.
6. **Processes:** stateless, share-nothing; state lives in backing services.
7. **Port binding:** the app is self-contained and exports HTTP via a port.
8. **Concurrency:** scale out by adding processes.
9. **Disposability:** fast startup, graceful shutdown.
10. **Dev/prod parity:** keep environments as similar as possible.
11. **Logs:** written as event streams to stdout; the platform collects them.
12. **Admin processes:** one-off tasks (migrations, backfills) run in the same environment as the app.

## 9. API design *(Recommendation, industry convention)*

- Resource-oriented URLs (`/clients/{id}/calendar/activities`), correct HTTP verbs and status codes.
- One error format for every endpoint (e.g. RFC 9457 Problem Details).
- Pagination, filtering and sorting on every list endpoint; never return unbounded lists.
- Idempotency keys on bulk and retryable operations (bulk program assignment, CSV export jobs).
- Version the API from day one (`/v1/`) because the client mobile app depends on it.
- Describe the API in an OpenAPI spec and generate typed clients from it.
- Long-running work (exports, bulk assigns, video processing) runs as background jobs with status polling or notifications.

## 10. Data and database *(Recommendation)*

- Schema changes only through versioned migrations in git; every migration reviewed and reversible.
- Transactions around multi-row operations (bulk assign, superset grouping).
- Indexes for every common filter (`gym_id`, `client_id`, date).
- Store timestamps in UTC; convert to the coach's time zone in the UI.
- Audit columns (`created_by`, `updated_at`) and soft delete where history matters.
- Automated backups with tested restores.

## 11. Observability *(Recommendation)*

- Structured JSON logs with a request/correlation ID; never log secrets or PII.
- Metrics for latency, error rate and throughput per endpoint; alert on SLO breaches.
- Distributed tracing (OpenTelemetry) across webapp, API and background jobs.
- Front-end error tracking and real-user performance monitoring.

## 12. Versioning, releases and documentation

- **Semantic Versioning** for released packages and the public API: MAJOR for breaking changes, MINOR for backward-compatible features, PATCH for backward-compatible fixes.
- Conventional Commits so changelogs can be generated *(Recommendation)*.
- Trunk-based development with short-lived branches and feature flags for unfinished work *(Recommendation)*.
- CI/CD gates: lint → typecheck → tests → security scans → build → deploy to staging → smoke test → production, with one-click rollback *(Recommendation)*.
- Docs in the repo: README (setup in under 10 minutes), Architecture Decision Records (ADRs) for major choices, API docs from OpenAPI *(Recommendation)*.

## References

- OWASP Top 10:2025 — https://owasp.org/Top10/2025/
- OWASP Top 10:2025 Introduction (what changed) — https://owasp.org/Top10/2025/0x00_2025-Introduction/
- OWASP Application Security Verification Standard (ASVS) 5.0 — https://owasp.org/www-project-application-security-verification-standard/ and https://github.com/OWASP/ASVS
- OWASP Cheat Sheet Series — https://cheatsheetseries.owasp.org/
- The Twelve-Factor App — https://12factor.net/
- Semantic Versioning 2.0.0 — https://semver.org/
- RFC 9457 Problem Details for HTTP APIs — https://www.rfc-editor.org/rfc/rfc9457
- Conventional Commits — https://www.conventionalcommits.org/
- Google Engineering Practices, The Standard of Code Review — https://google.github.io/eng-practices/review/reviewer/standard.html
- Google Code Review Developer Guide — https://google.github.io/eng-practices/review/
