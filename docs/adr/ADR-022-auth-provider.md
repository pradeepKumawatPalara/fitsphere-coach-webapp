# ADR-022 · Authentication provider: Supabase Auth

| | |
|---|---|
| **Status** | 🟡 Proposed (provider confirmed as Supabase Auth; awaiting business-owner acceptance) |
| **Log entry** | D-22 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 (was Open; fact confirmed Nov 2026) |
| **Owner / approver** | Business owner *(approval pending)* |
| **Scope** | How coaches and admins sign in, and how the API learns who they are, their gym and their role |
| **Risk tier** | High (authentication and session security) |

## Context

The Android app already authenticates its users, and the API (ADR-001) is shared. It was confirmed that **Supabase Auth is the auth provider in use**. The web app needs login for admins and coaches. The API must verify the token on every request and load the user's `gym_id` and role (ADR-005, ADR-020).

Because Android clients and web users share one Supabase Auth user pool, a valid sign-in does **not** mean permission to use the web app: the API must check the user's role and gym.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. Supabase Auth (same as Android) | B. Firebase Auth via Supabase third-party auth | C. Another managed identity provider | D. Custom-built authentication |
|---|---|---|---|---|
| Security / privacy gate | Pass | Pass | Pass | **Fail:** the coding standard requires a proven provider or library |
| Security (20%) | 4 | 4 | 4 | n/a |
| Maintainability (20%) | 5 | 2 | 3 | n/a |
| Reliability (15%) | 4 | 3 | 4 | n/a |
| Performance (15%) | 4 | 4 | 4 | n/a |
| Scalability (10%) | 4 | 4 | 4 | n/a |
| Cost (10%) | 5 | 3 | 2 | n/a |
| Integration (10%) | 5 | 2 | 2 | n/a |
| **Weighted score** | **4.40** | **3.15** | **3.40** | excluded (failed gate) |

## Proposed decision

**Option A.** Use Supabase Auth for the web app, the same provider as the Android app. The API verifies the Supabase JWT on every request and loads the user's `gym_id` and role.

## Why

- One identity system for both apps: no second set of accounts to migrate, link or secure.
- Already in production with the Android app; works with the same database and RLS (ADR-003, ADR-005).

## Rejected

- **B. Firebase Auth:** a second identity system next to live Supabase accounts.
- **C. Another provider:** extra vendor and cost with no requirement that A cannot meet.
- **D. Custom:** not acceptable under the coding standard.

## Requirements any option must meet (pass/fail)

- API verifies the JWT on every request and loads `gym_id` and role; `gym_id` is never taken from the request.
- MFA for admins; login rate limiting; secure password reset.
- Short-lived tokens with rotation; logout invalidates the session server-side.
- Accessible authentication (WCAG 3.3.8): password managers and paste allowed, no cognitive puzzles.
- Coach accounts are created by an admin (invite links are out of MVP scope, ADR-019), so a "set your password" flow is needed.
- No secrets in the web app; only public configuration.

## Consequences

- The web app uses the Supabase client **for sign-in and session handling only**, never for data queries (ADR-004). Enforce with a lint rule or review check that blocks `supabase.from(...)` and similar data calls in the web repo.
- The browser holds a user JWT, so Supabase row-level security must itself be correct: a signed-in user must not be able to read other gyms' data by calling Supabase directly, regardless of what the web app does (ADR-005).
- The API rejects valid tokens from users who are not an active coach or admin with a gym (for example, client-only accounts): see AC-AUTH-03.
- Creating coach accounts uses Supabase admin functions on the **server only**; the secret key never reaches the web app.
- Web `.env` holds only the project URL and the publishable key (`.env.example`).
- Needs auth test cases per the testing standard: expired token, wrong gym, wrong role, client-only account, brute-force limits.

## Related decisions

ADR-001, ADR-003, ADR-004, ADR-005, ADR-019, ADR-020.

## Open points

1. How do users sign in: email and password, phone OTP, or both? Do coaches use the same accounts as clients?
2. Where do `gym_id` and role come from: token claims (for example a custom access-token hook) or a database lookup per request?
3. Can an existing Android user also be a coach (one account, several roles)?
4. Token lifetime, refresh and session policy; MFA method for admins (confirm what Supabase Auth supports on the chosen plan).
5. Confirm that existing RLS policies (used by the Android app's direct Supabase access, if any) do not expose cross-gym data once coaches and admins can sign in.
