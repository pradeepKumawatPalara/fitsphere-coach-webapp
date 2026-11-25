# ADR-022 · Authentication provider

| | |
|---|---|
| **Status** | ❓ Open (no decision yet; depends on facts to be confirmed) |
| **Log entry** | D-22 in `docs/DECISIONS.md` |
| **Date opened** | Nov 2026 |
| **Owner / approver** | Business owner *(approval pending)* |
| **Scope** | How coaches and admins sign in, and how the API learns who they are, their gym and their role |
| **Risk tier** | High (authentication and session security) |

## Context

The Android app already authenticates its users, and the API (ADR-001) is shared. The web app needs login for admins and coaches. The API must verify the token on every request and load the user's `gym_id` and role (ADR-005, ADR-020). This decision is Open: it cannot be scored fairly until we confirm what the Android app uses today, so this ADR has no weighted scores yet.

## Options

| Option | Notes |
|---|---|
| A. Supabase Auth | Same vendor as the database; JWT works with RLS |
| B. Firebase Auth (via Supabase third-party auth) | Fits if the Android app already uses Firebase |
| C. Another managed identity provider | Extra vendor and cost; only if A and B cannot meet the requirements |
| D. Custom-built authentication | **Rejected in advance:** the coding standard requires a proven provider or library (see `docs/best-practices/01-coding-best-practices.md`) |

## Architect's recommendation

**Use whichever provider the Android app already uses (A or B).** Running a second identity system means migrating or linking live user accounts and keeping two systems secure. Score A and B formally once the Android provider is confirmed.

## Requirements any option must meet (pass/fail)

- API verifies the JWT on every request and loads `gym_id` and role; `gym_id` is never taken from the request.
- MFA for admins; login rate limiting; secure password reset.
- Cookie or token handling per the secure coding checklist: short-lived tokens with rotation, logout invalidates server-side.
- Accessible authentication (WCAG 3.3.8): password managers and paste allowed, no cognitive puzzles.
- Coach accounts are created by an admin (invite links are out of MVP scope, ADR-019), so a "set your password" flow is needed.
- No secrets in the web app; only public configuration.

## Consequences (once decided)

- Defines the shape of the token claims and where roles live.
- If Supabase Auth is chosen, the browser may use the SDK for sign-in only, never for data queries (ADR-004).
- Needs auth test cases per the testing standard: expired token, wrong gym, wrong role, brute-force limits.

## Related decisions

ADR-001, ADR-004, ADR-005, ADR-019, ADR-020.

## Open points (needed to close this ADR)

1. Which provider does the Android app use today?
2. How do users sign in: email and password, phone OTP, or both? Do coaches use the same accounts as clients?
3. Where do `gym_id` and role come from: token claims or a database lookup?
4. Can an existing Android user also be a coach (one account, several roles)?
5. Token lifetime, refresh and session policy; MFA method for admins.
