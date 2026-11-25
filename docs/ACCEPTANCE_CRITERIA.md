# FitSphere Coach Web App — Acceptance Criteria (first features)

| | |
|---|---|
| **Status** | 🟡 Draft — must be approved by the business owner before each feature is ready for development (REQ-009) |
| **Version** | 0.1 (Nov 2026) |
| **Rules** | `docs/STANDARD_IT_TESTING_RULES.md`: REQ-009 to REQ-013 |
| **Roles** | Admin, Coach (Client has no web access) |

**How to read this file**
- Each criterion has an ID, is written as given / when / then, and can be checked independently (REQ-010).
- **TBC** marks a business rule not yet decided. Claude must not guess these; it asks. The number refers to the open questions in `docs/FITSPHERE_COACH_WEBAPP.md` section 10, or to the ADR concerned.
- Once testing starts, criteria change only by a new version (REQ-013).
- Each ID is linked to at least one test (REQ-014); untested criteria are reported as gaps before release (REQ-016).

---

## F-01 Sign in and sign out

*Depends on ADR-022 (auth provider, still open). Criteria are written to hold for any provider.*

| ID | Criterion |
|---|---|
| AC-AUTH-01 | **Given** an active admin or coach account with valid credentials, **when** they sign in, **then** they reach the Dashboard and see only data from their own gym (admin: whole gym; coach: own clients). |
| AC-AUTH-02 | **Given** wrong credentials, **when** they sign in, **then** they see one generic error that does not reveal whether the account exists, no session is created, and the entered email is kept. |
| AC-AUTH-03 | **Given** a deactivated coach, a user without a gym or role, or a client-only (Android) account, **when** they sign in, **then** access is refused with a clear message and no data is returned. |
| AC-AUTH-04 | **Given** repeated failed sign-ins, **when** the limit is reached, **then** further attempts are blocked for a period and the message says to try later. Limit and period: **TBC (ADR-022)**. |
| AC-AUTH-05 | **Given** a signed-in user, **when** they sign out, **then** the session ends on the server, and reusing the old token or pressing Back returns no data (401). |
| AC-AUTH-06 | **Given** an expired session, **when** the user acts, **then** they are sent to sign in and, after signing in, returned to the page they were on. |
| AC-AUTH-07 | **Given** the sign-in screen, **then** password managers and paste work, there is no puzzle or CAPTCHA that tests memory or recognition, every field has a visible label, and the whole flow works with the keyboard only (WCAG 3.3.8). |
| AC-AUTH-08 | **Given** an admin account, **when** they sign in, **then** a second factor is required. Method: **TBC (ADR-022, point 5)**. |
| AC-AUTH-09 | **Given** a password reset request, **then** the response is the same whether or not the email exists, and the reset link is single-use and short-lived. Lifetime: **TBC**. |

**Who can act:** active Admin and Coach accounts can sign in. Deactivated users, users without a gym or role, and clients cannot.

---

## F-02 Gym isolation and role access (applies to every feature)

| ID | Criterion |
|---|---|
| AC-ACC-01 | **Given** a coach or admin from gym A, **when** they request any gym B record (client, calendar activity, program, coach, notification, file) by ID or by search, **then** the API returns 403 or 404, and the response does not confirm the record exists. |
| AC-ACC-02 | **Given** a request that includes a `gym_id`, **then** the server ignores it and uses the signed-in user's gym. |
| AC-ACC-03 | **Given** a coach, **when** they request a client of the same gym who is not assigned to them, **then** access is denied. **TBC (ADR-020): whether any coach cover is allowed.** |
| AC-ACC-04 | **Given** a coach, **when** they call an admin-only action (add coach, assign role, gym settings) directly through the API, **then** the API refuses it even if the screen hides the button. |
| AC-ACC-05 | **Given** any admin action (add coach, role change, bulk program assignment, export), **then** an audit record stores who, what, when and for which records. |
| AC-ACC-06 | **Given** a signed-in coach, **then** admin-only menu items are not shown. |
| AC-ACC-07 | **Given** a failure while checking permission, **then** access is denied (fail closed). |

**Role matrix for the first features**

| Action | Admin | Coach |
|---|---|---|
| Sign in | Yes | Yes |
| See all clients in the gym | Yes | No (own clients only) |
| See own clients | Yes | Yes |
| Export clients (within scope) | Yes | Yes |
| Add coach, assign role | Yes | No |
| Anything in another gym | No | No |

---

## F-03 Client list

| ID | Criterion |
|---|---|
| AC-CLI-01 | **Given** an admin, **then** the list shows all clients of the gym; **given** a coach, **then** only their assigned clients. Each row shows name and streak plus other key info: **TBC (fields)**. |
| AC-CLI-02 | **Given** more clients than one page, **then** the list shows 25 per page (**TBC: default page size**) with the total count and page controls, and the server returns only that page. |
| AC-CLI-03 | **Given** a search term, **when** the coach searches, **then** results contain only matching clients within their allowed scope. **Given** no match, **then** an empty state says so and offers "Clear search". |
| AC-CLI-04 | **Given** sort or filter changes, **then** the server applies them, the page resets to 1, the order is stable between pages, and the URL holds the state so a refresh or shared link restores it. |
| AC-CLI-05 | **Given** the list is loading, failing or empty, **then** it shows a skeleton, an error with a Retry button, or an empty state with a next action, respectively. |
| AC-CLI-06 | **Given** the streak, **then** it follows the streak rules: consecutive days, rest days, missed-day reset and time zone (gym time zone, default Asia/Kolkata). **TBC: the exact streak rules are not yet defined; this blocks F-03 and F-05.** |
| AC-CLI-07 | **Given** bulk select, **when** rows are selected, **then** a visible count is shown and actions apply only to those rows. Selecting across pages: **TBC (ADR-011)**. |
| AC-CLI-08 | **Given** the list, **then** it works with the keyboard and a screen reader, buttons and checkboxes are at least 24×24 px, colour is never the only signal, and the layout works at tablet width. |
| AC-CLI-09 | **Given** a typical gym list, **then** interaction latency (INP) stays at or below 200 ms at the 75th percentile. Largest gym size to test: **TBC**. |

**Who can act:** Admin sees the whole gym; Coach sees only their own clients; no one sees another gym.

---

## F-04 CSV export

| ID | Criterion |
|---|---|
| AC-CSV-01 | **Given** the export dialog, **when** the coach picks fields, **then** the file contains exactly those columns in the chosen order with a header row. Selectable fields: **TBC (open question 7)**. |
| AC-CSV-02 | **Given** selected rows or the current page, **when** exported, **then** the browser generates a file with only those rows and only permitted fields. |
| AC-CSV-03 | **Given** "export all clients", **when** requested, **then** the server generates the file for the user's scope only (coach: own clients; admin: whole gym), shows progress for large exports, and offers the download when ready. |
| AC-CSV-04 | **Given** a failed or timed-out export, **then** the user sees a clear message and can retry, and no partial file is presented as complete. |
| AC-CSV-05 | **Given** too many export requests, **then** the request is refused with a message, and no file is produced. Limit: **TBC**. |
| AC-CSV-06 | **Given** names or values containing commas, quotes, line breaks, non-English characters (for example Hindi) or text starting with `=`, `+`, `-` or `@`, **then** the file is correctly escaped, opens with the right characters, and no value runs as a formula. |
| AC-CSV-07 | **Given** any export (browser or server), **then** an audit record stores who exported, when, which fields and how many rows (ADR-012). |
| AC-CSV-08 | **Given** sensitive fields (for example date of birth, phone), **then** the roles allowed to export them are enforced by the API. **TBC: which fields are admin-only.** |

**Who can act:** Admin and Coach may export within their scope; no one can export another gym's data.

---

## Still to write (each before its feature starts)

F-05 Dashboard "Needs attention" (tag list and rules: open question 4) · F-06 Client profile, summary, autonomy toggle and audit trail (open question 8) · F-07 Client calendar and Lifestyle view (open questions 1, 6) · F-08 Workout builder and supersets (open question 5) · F-09 Programs and bulk assign · F-10 Coaches · F-11 Notifications.

## Open questions that block criteria

| Question | Blocks |
|---|---|
| Streak rules (rest days, time zone, reset) | AC-CLI-06, F-05 |
| Client list fields and default page size | AC-CLI-01, AC-CLI-02 |
| Selectable CSV fields; admin-only fields | AC-CSV-01, AC-CSV-08 |
| Rate limits (sign-in, export) | AC-AUTH-04, AC-CSV-05 |
| Auth provider, admin MFA method | F-01 (ADR-022) |
| Can a coach see other coaches' clients? | AC-ACC-03 (ADR-020) |
