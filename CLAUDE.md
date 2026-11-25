# CLAUDE.md — FitSphere Coach Web App

Instructions for Claude Code in this repo. Read this first in every session.

## Project summary

**FitSphere** is a fitness platform for gyms and coaches in India. It has two apps:
- **FitSphere client app** (Android, already live): clients see and log their own workouts, nutrition and lifestyle targets.
- **FitSphere Coach web app** (this repo): gym admins and coaches plan everything for their clients and track how well clients stick to it.

Whatever a coach plans here shows up in the client app, and whatever a client logs shows up here, because **both apps use the same Node.js API and Supabase database**.

**Who uses it**
- **Admin** (gym owner/manager): manages the gym's coaches and roles, and sees all clients.
- **Coach**: manages their own clients, calendars and programs.

**What the MVP does**

| Section | Summary |
|---|---|
| **Dashboard** | "Needs attention" tab listing clients with pending items. Tags are created automatically from the targets and activities the coach set (missed workout, not logged, lifestyle target missed, etc.). |
| **Clients** | Client list with streaks, search, bulk select and CSV export (coach picks the fields). Each client has a summary dashboard, coaching services setup, a client-autonomy toggle (on: client can add and log own workouts; off: only coach-assigned workouts are recorded) and an audit trail of the last coach. |
| **Client calendar** (inside each client) | Week/month view of the client's workouts, meals and responses. Coach can add, edit, move, copy, delete, multi-select and reorder activities (like the Teams calendar), react and comment, attach images/videos, and add a training for a set period (e.g. 1 or 3 months). Each workout has its own message thread. A **Lifestyle view** holds non-exercise targets (e.g. 8 glasses of water) with reminder time and days sent to the client app. |
| **Workout builder** | Workout → exercises → sets. Pick exercises from a searchable database (filter by movement pattern), add conditions (sets, reps, weight, rest…), and link exercises into **supersets**. |
| **Programs** | Build exercises → workouts → programs → trainings. Each coach can create many programs; prebuilt programs come with the app. Assign a program to many clients at once. |
| **Coaches** | All coaches in the gym; admin adds coaches and assigns roles. |
| **Notifications** | Alerts for client activity, updates and messages; shown in a pop-up panel, each expandable for details. |

**Not in the MVP** (don't build unless asked): client/task priorities, gym branding/white-label, invite links, slot booking, UPI payments, wearables, chat, check-in forms, challenges, multi-language.

**References**

- UI/UX design: https://claude.ai/artifact/S1BBcDkWgLnqPnNPLjfqsH?sk=8sowdQAFUHY3z5iDAv7WuA#page-dcb81aee8d83. Match its layout, spacing and components.

## Best practices (follow these on every task)

Follow the best-practice docs in `docs/best-practices/`. Read the relevant one before starting work:

- `01-coding-best-practices.md` — security (OWASP), multi-tenancy, TypeScript, API, data, observability
- `02-testing-and-unit-testing.md` — test pyramid, unit/integration/E2E rules, CI quality gates
- `03-design-guide-and-ui-design.md` — WCAG 2.2 AA, Core Web Vitals, design system, UI states
- `04-avoiding-deprecated-apis-and-libraries.md` — current docs, no deprecated APIs, dependency hygiene
- `05-claude-code-skills.md` — skills, hooks and workflow conventions

If this file and a best-practice doc conflict, follow this file and flag the conflict.

**Testing standard:** `docs/STANDARD_IT_TESTING_RULES.md` is the project's testing and release standard. Its rules have IDs (e.g. `GOV-004`, `REQ-009`, `PLAN-012`, `GOV-016`); cite the rule ID when you apply one. It is long (~2,000 lines), so don't load it all — use the table of contents and read only the sections relevant to the task. Risk tier for this app: high (stores personal data and cloud tokens). Never delete, skip or weaken a test to make it pass; release sign-off is a business decision on recorded evidence, not on Claude's assurance.

**Decisions:** follow every ✅ Accepted decision in `docs/DECISIONS.md`. Treat 🟡 Proposed and ❓ Open decisions (e.g. D-20 roles, D-21 calendar placement, D-22 auth provider) as unconfirmed — ask before building on them. A change that conflicts with an Accepted decision must cite its ID and be approved first. Never mark your own proposals Accepted; record new or changed decisions as 🟡 Proposed using the template in that file.

## ADR template

Each decision has a file `docs/adr/ADR-NNN-short-name.md` (ADR-NNN matches D-NN in `docs/DECISIONS.md`). For a new or changed decision, copy the structure of an existing ADR (e.g. `docs/adr/ADR-001-shared-nodejs-api.md`), write it as 🟡 Proposed, add a row to the Summary table in `docs/DECISIONS.md`, and never mark it Accepted yourself. Don't edit an Accepted ADR: add a new one and mark the old one ⛔ Superseded.

```markdown
# ADR-NNN · Title

| | |
|---|---|
| **Status** | 🟡 Proposed | ✅ Accepted | ❓ Open | ⛔ Superseded by ADR-NNN |
| **Log entry** | D-NN in `docs/DECISIONS.md` |
| **Date** | |
| **Owner / approver** | Business owner (name and approval date) |
| **Scope** | |
| **Risk tier** | Low | Medium | High |

## Context
Problem, requirements and constraints.

## Options considered
Table scoring each option 1–5 on: Security (20%), Maintainability (20%), Reliability (15%),
Performance (15%), Scalability (10%), Cost (10%), Integration (10%), then the weighted score.
Security, privacy and legal are pass/fail first; a failed option is excluded, not scored.
Scores are a proposal for the owner to review.

## Decision
## Why
## Rejected
## Consequences
## Related decisions
## Open points
```

Default candidate is a modular monolith; microservices or serverless need a proven business reason. If the score disagrees with the owner's decision, record the decision and add an "Architect note" instead of changing it.

## Tech stack (decided — do not swap libraries without asking)

| Area | Use |
|---|---|
| Foundation | React + Vite + TypeScript (strict), static build |
| Styling | Tailwind CSS **v4** (CSS-first `@theme` in CSS; **no** `tailwind.config.js`) |
| Components | shadcn/ui (Radix) in `src/components/ui/` |
| Icons | lucide-react |
| Charts | shadcn Charts (Recharts) |
| Calendar | FullCalendar **Standard only** (`@fullcalendar/react`, `daygrid`, `timegrid`, `interaction`) — no Premium plugins |
| Drag and drop in workouts | dnd-kit (`@dnd-kit/react`) |
| Routing | TanStack Router (file-based) |
| Server state | TanStack Query |
| Tables | TanStack Table (server-side sort/filter/pagination) |
| Forms | React Hook Form + Zod |
| API client | Generated from the backend OpenAPI spec; never hand-write API types |
| Tests | Vitest + React Testing Library + MSW; Playwright for E2E; axe for accessibility |
| Lint/format | ESLint (typescript-eslint, `@typescript-eslint/no-deprecated`) + Prettier |

Backend (separate repo): Node.js + TypeScript API on Google Cloud Run, Zod + OpenAPI, Supabase Postgres with RLS. Web hosting: Hostinger (static files).

## Commands

```bash
npm install          # install
npm run dev          # local dev server
npm run build        # typecheck + production build (dist/)
npm run typecheck    # tsc --noEmit
npm run lint         # eslint
npm run test         # vitest
npm run test:e2e     # playwright
npm run gen:api      # regenerate typed API client from OpenAPI
```

Before saying a task is done, run `typecheck`, `lint` and `test`, and fix every failure.

## Folder structure

```
src/
  routes/                 # TanStack Router file routes (dashboard, clients, programs, coaches)
  features/
    dashboard/            # Needs attention
    clients/              # list, profile, summary, autonomy, export
    calendar/             # FullCalendar views, lifestyle view, activity actions
    workouts/             # workout builder, exercises, supersets (dnd-kit)
    programs/             # programs, trainings, bulk assign
    coaches/              # coach list, add coach, roles
    notifications/        # bell, pop-up, expandable items
  components/ui/          # shadcn components (owned by us, edit freely)
  components/             # shared app components
  lib/api/                # generated API client + query hooks
  lib/                    # utils, auth, formatting
  styles/                 # Tailwind entry + @theme tokens
```

Each feature folder holds its components, hooks, Zod schemas and tests together.

## Architecture rules

1. **The web app talks only to the Node.js API.** Never call Supabase directly from the browser and never put a Supabase secret/service key in this repo.
2. **Every request is scoped to the user's gym.** Assume the API enforces `gym_id` and role; never filter tenancy on the client only.
3. **Roles:** `admin` (whole gym, manages coaches) and `coach` (own clients). Hide UI the role can't use, but treat the API as the real check.
4. **Server state lives in TanStack Query.** Don't copy API data into React state or a global store. Use query keys per feature (`['clients', params]`, `['calendar', clientId, range]`).
5. **Paginate everything.** Client list via server-side TanStack Table; calendar loads one week/month range at a time.
6. **CSV export:** selected rows/current page in the browser; "export all" calls the API export endpoint.
7. **Client autonomy:** respect the flag from the API — when off, don't offer client-logged workouts.
8. **Media:** images go to upload endpoints; exercise videos are external links (YouTube/Vimeo). Show thumbnails, not raw files.
9. Optimistic updates for calendar move/copy/delete, with rollback and an undo toast.

## Code style

- TypeScript strict. No `any`; use `unknown` and narrow. No non-null `!` unless justified in a comment.
- Function components and hooks only. Named exports.
- Validate every form with a Zod schema; infer types with `z.infer`.
- Style with Tailwind classes and theme tokens; no inline hex colours, no CSS-in-JS. Use `cn()` for conditional classes.
- Reuse shadcn components before writing new ones. Add them with `npx shadcn@latest add <name>`.
- Keep components small; move logic into hooks (`useClientList`, `useMoveActivity`).
- Every screen handles **loading** (Skeleton), **empty** and **error** states.
- Dates: store/send ISO strings in UTC; display in the gym's timezone (default `Asia/Kolkata`).

## Accessibility (WCAG 2.2 AA)

- All interactive elements reachable and usable by keyboard, with visible focus.
- Every drag action has a non-drag alternative: "Move to… / Copy to…" menu on calendar activities; buttons to reorder exercises and link supersets.
- Labels on every input; icon-only buttons need `aria-label`.
- Colour is never the only signal (e.g. Needs-attention tags have text too).

## Testing

- Unit/component tests next to the code: `*.test.tsx`.
- Mock the API with MSW, not by mocking fetch or hooks.
- Test behaviour from the user's view (React Testing Library queries by role/label).
- Add a Playwright test for each key flow: login, add activity to calendar, move activity, build superset, assign program to many clients, export CSV.
- Run axe in component and E2E tests.

## Avoid deprecated or outdated APIs

- Before using a library API you're not sure is current, check the docs with **Context7** (`use context7`) for the installed version in `package.json`.
- Don't use: Tailwind v3 config/`@tailwind` directives, React class components, `defaultProps` on function components, `ReactDOM.render`, Create React App, `moment.js`, React Router (we use TanStack Router).
- Fix any `no-deprecated` lint warning instead of disabling it.
- Don't add a new dependency without saying why; prefer what's already in the stack.

## Security

- No secrets in code or committed `.env` files. Only `VITE_` public config (e.g. `VITE_API_BASE_URL`) goes in the frontend.
- Never log tokens or client personal data (DOB, phone, health info).
- Don't render user HTML; no `dangerouslySetInnerHTML`.

## Git and workflow

- Branch per feature: `feature/<area>-<short-name>`. Small, focused commits with clear messages.
- Plan first for anything touching more than a couple of files; list the files and steps before editing.
- If a requirement is unclear or conflicts with the design, check `docs/FITSPHERE_COACH_WEBAPP.md` → "Open questions", then ask. Don't guess business rules (Needs-attention tags, exercise conditions, export fields).
- Never edit generated files (`src/lib/api/generated/`, `src/routeTree.gen.ts`) by hand.
