# FitSphere Coach — Web App (MVP)

> The coach-facing web app of the FitSphere fitness platform. Gym coaches and admins use it to manage clients, plan workouts, nutrition and lifestyle goals, and track how well clients stick to their plans. Whatever a coach plans here shows up in the FitSphere client app (Android).

| Item | Value |
|---|---|
| Product | FitSphere Coach (web) |
| Users | Gym admins and coaches (India first) |
| Status | MVP: design done, development starting (Nov 2026) |
| UI/UX design | [FitSphere Coach — Web (design)](https://claude.ai/artifact/S1BBcDkWgLnqPnNPLjfqsH?sk=8sowdQAFUHY3z5iDAv7WuA#page-dcb81aee8d83) |
| Repo | New, separate repo for the web app (backend API stays in its existing repo) |
| Source documents | `Fitsphere_Client__Coach.docx` (MVP requirements with screenshots), `Fitness_Platform_Features.docx` (full feature vision), manual review notes |

---

## 1. Navigation (MVP)

| # | Section | Purpose |
|---|---|---|
| 1 | **Dashboard** | "Needs attention" overview of clients with pending items |
| 2 | **Clients** | Client list, client profile, summary dashboard, **client calendar** |
| 3 | **Programs** | Exercise library, workouts, programs and trainings; assign to many clients at once |
| 4 | **Coaches** | All coaches of the gym; admin adds coaches and assigns roles |
| — | **Notifications** | Bell icon on every page; expandable pop-up panel |

> ⚠️ **To confirm:** an earlier scope note listed the tabs as *Dashboard, Clients, Calendar, Programs*. The requirements doc lists *Dashboard, Clients, Coaches, Programs*, with the calendar **inside** each client. This file follows the requirements doc. See [Open questions](#10-open-questions).

---

## 2. Roles and access

| Role | Can do |
|---|---|
| **Admin** (gym owner/manager) | Everything a coach can do, plus manage coaches (add, deactivate, assign role), see all clients in the gym, gym-level settings |
| **Coach** | Manage their assigned clients, their calendars, programs and messages; create their own programs |
| **Client** | Uses the Android client app only (no web access). Sees only their own data |

Rules:
- Every record belongs to a **gym** (`gym_id`). A coach can never see another gym's data.
- Permissions are checked in the API (policy module: role × action × resource) and again by **Row Level Security** in Supabase.
- **Dashboard question (from review):** *Do we have any roles?* → Yes, at minimum **Admin** and **Coach** (the Coaches tab assigns roles). The Dashboard should show an admin the whole gym and a coach only their own clients. Confirm whether an "Assistant coach" role is needed (see Open questions).

---

## 3. Dashboard

- **Needs attention** tab listing clients with pending items.
- Tags are **created automatically** from the targets and activities the coach has set (for example: missed workout, workout not logged, lifestyle target missed, unread message, program ending soon).
- Each item links straight to the client or to the calendar day in question.
- Admin view: whole gym. Coach view: own clients.
- **Not in MVP:** setting priorities on clients or tasks.

---

## 4. Clients

### 4.1 Client list
- Table of clients with key info, including the **streak**.
- Search, sort, filter and pagination (done on the server).
- Bulk select (for example, to assign a program).
- **Export to CSV**: the coach chooses which fields to include (name, age, DOB, email, phone, coach, program, streak, etc.).
  - Selected rows or current page: exported in the browser.
  - All clients: generated on the server and streamed as a file.

### 4.2 Client profile
- **Summary dashboard**: streak, adherence, recent activity, current program.
- **Coaching services**: configure which services apply to the client (workouts, nutrition, lifestyle, etc.).
- **Client autonomy** toggle:
  - **On**: the client can add and log their own workouts in the client app.
  - **Off**: only workouts the coach assigns can be recorded.
- **Audit trail**: keep a record of the last coach/consultant who worked with the client and who changed what and when, so history is kept when a client changes coach.

### 4.3 Client calendar
The coach opens a client's calendar to see and manage everything planned and logged.

- **Views**: week and month; a separate **Lifestyle view** for non-exercise targets.
- **What the coach sees**: the client's activity and responses on exercises and nutrition/meals (logged, skipped, notes).
- **Coach responses**: react, comment and upload **images and videos** on an activity.
- **Activity actions** (on hover/menu): **Add, Edit, Move, Copy, Delete, Select** (multi-select).
- **Reorder** workouts up and down within a day, and drag between days (similar to the Teams calendar).
- **Keyboard alternative** for drag: a "Move to… / Copy to…" menu on every activity (accessibility).
- **Add training** for a set period (for example, 1 month or 3 months), which fills the calendar for that range.

### 4.4 Workout builder
Structure: **Workout → Exercises → Sets**.

- Pick exercises from the **exercise database** by typing; filter by movement pattern and other details.
- Add exercises **with conditions** (sets, reps, weight, tempo, rest, duration, notes; exact fields to confirm).
- **Supersets**: link two or more exercises in a workout so they're done together (drag one onto another, or select and "Link as superset").
- Attach images and videos (from the exercise library or uploaded).

### 4.5 Message thread per workout
- Each workout on a single day has its own **thread**.
- Coach and client can comment and add images and videos in that thread.

### 4.6 Lifestyle targets
- Coach adds lifestyle targets per client (for example, *8 glasses of water a day*).
- Set the **unit**, the target amount, and **reminder time and days**.
- Reminders are delivered in the client app.
- Shown in the calendar's Lifestyle view.

---

## 5. Programs

Build-up: **Exercise → Workout → Program → Training**.

- **Exercises**: add and manage exercises; attach videos from the exercise library.
- **Workouts**: create reusable workouts from exercises (with supersets).
- **Programs**: each coach can create several programs; the app also ships **prebuilt programs**.
- **Trainings**: schedule a program over a period for a client (start date and duration).
- **Assign** a program (own or prebuilt) to **many clients at once**.

---

## 6. Coaches

- Lists every coach in the gym.
- Admin can **add a new coach** and **assign their role**.

---

## 7. Notifications

- Notifications for every client and coach activity: workouts logged, updates, messages, targets missed.
- Bell icon opens a **pop-up panel**; each notification can be **expanded** to show details.
- Unread count, mark as read.

---

## 8. Out of scope for MVP (future, from the platform feature list)

| Area | Feature |
|---|---|
| Branding | Gym logo and colour theme (white-label); invite link that applies the coach's theme |
| Mobile coach app | Quick edits on the gym floor (detailed planning stays on the web) |
| Booking | Slot booking with configurable free-cancellation window; waitlist |
| Payments | UPI payment link / QR; UPI AutoPay for monthly plans |
| Wearables | Health Connect first, other devices later |
| Training load | Post-workout difficulty (RPE 1–10) and duration |
| Engagement | Chat, broadcast messages, check-in forms, challenges, badges, IG-ready summaries |
| Nutrition | Indian food database, meal builder |
| Advanced training | %1RM programming, injury modifiers, exercise swaps |
| Admin | Assistant-coach role, digital waivers and PAR-Q, tiered plans |
| Language | Hindi, Marathi, Tamil, Telugu |
| Dashboard | Client/task priorities |

---

## 9. Tech stack (decided)

### 9.1 Frontend — coach web app

| Layer | Tool |
|---|---|
| Foundation | **React + Vite + TypeScript** (static build) |
| Styling | **Tailwind CSS** (v4, CSS-first `@theme` config) |
| Components | **shadcn/ui** (on Radix) |
| Icons | **Lucide** |
| Dashboard charts | **shadcn Charts** (on Recharts) |
| Client calendar | **FullCalendar Standard** (week/month views, move activities between days) |
| Workout editing | **dnd-kit** (reorder exercises, build supersets, drag from library) |
| Routing | **TanStack Router** |
| Server data and caching | **TanStack Query** |
| Tables | **TanStack Table** (server-side sorting, filtering, pagination; bulk select) |
| Forms and validation | **React Hook Form + Zod** |
| API client | Typed client **generated from OpenAPI** |
| Auth | Same login as the Android app |

### 9.2 Backend

| Layer | Tool |
|---|---|
| API | **Existing Node.js API** (TypeScript), extended with coach modules: `src/modules/{dashboard, clients, calendar, programs, coaches, notifications}` |
| Validation and docs | **Zod** + **OpenAPI** |
| Database access | `supabase-js` with secret key (server only) or `pg`/Drizzle through the **Supabase pooler** |
| Database | **Supabase Postgres**: reuse existing tables (`workout`, `nutrients`, `exercise`); new tables in the same project, with **RLS** on every table |
| Migrations and types | **Supabase CLI** (`supabase/migrations/`, `supabase gen types typescript`) |
| File storage | **Supabase Storage** for images and thumbnails; exercise videos as external links (YouTube unlisted / Vimeo) |
| Secrets | **Google Secret Manager** |

### 9.3 Hosting and DevOps

| Layer | Tool | Cost |
|---|---|---|
| Web app | **Hostinger** (existing plan, subdomain) | $0 extra |
| API | **Google Cloud Run** | Free tier |
| Database | **Supabase** | Free while building → Pro (~$25/month) at go-live |
| Code and CI/CD | **GitHub + GitHub Actions** (lint → typecheck → test → deploy) | $0 |
| Cost protection | Google Cloud **budget alerts** | $0 |

### 9.4 Quality and tooling

| Layer | Tool |
|---|---|
| Unit and component tests | **Vitest + React Testing Library + MSW** |
| API tests | **Vitest + Supertest** against local Supabase (includes a cross-gym access test per endpoint) |
| End-to-end tests | **Playwright** |
| Accessibility | **axe** (WCAG 2.2) |
| Lint and format | **ESLint** (typescript-eslint, `no-deprecated`) + **Prettier** |
| AI development | **Claude Code** + `CLAUDE.md` + **Context7 MCP** + Stop hook (typecheck + lint + tests) |

### 9.5 How a request flows

```
Coach web app (Hostinger) ──► Node.js API (Cloud Run)
                                1. Verify JWT → coach, gym_id, role
                                2. Validate input (Zod)
                                3. Check permission (policy)
                                4. Business rules (autonomy, streaks, Needs attention)
                                5. Read/write Supabase (pooled connection)
                              ◄── JSON
Android client app ─────────► same API, same rules
```

- The web app talks **only** to the Node.js API, never to Supabase directly.
- The Supabase secret key lives only in the API.

### 9.6 Traffic and cost rules

- Static files served with long cache headers.
- Cloud Run auto-scales and goes to zero when idle.
- Supabase through the pooler; indexes on `gym_id`, `client_id`, `coach_id` and date columns.
- Every list is paginated; the calendar loads one week/month at a time.
- TanStack Query caching to avoid refetching on tab switch.
- Rate limits on login, bulk assign and CSV export; large exports and bulk assigns run as background jobs.
- Videos hosted externally; only thumbnails stored.

---

## 10. Open questions

1. **Tabs**: is Calendar a top-level tab, or only inside each client (as in the requirements doc)? Is Coaches in the MVP?
2. **Roles**: only Admin and Coach for MVP, or also Assistant coach?
3. **Dashboard by role**: confirm admin sees the whole gym and coach sees only their clients.
4. **"Needs attention" rules**: the exact list of tags and when each fires.
5. **Exercise conditions**: which fields (sets, reps, weight, tempo, rest, RPE, duration)?
6. **Training duration**: fixed options (1/3 months) or any date range?
7. **CSV export**: the full list of selectable fields.
8. **Audit trail**: what to keep (last coach only, or full change history)?
9. **Auth**: Supabase Auth or Firebase Auth — whichever the Android app uses today.
10. **Media limits**: maximum image/video size for uploads in threads and calendar.
