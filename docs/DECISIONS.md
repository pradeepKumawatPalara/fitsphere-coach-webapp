# decision.md — FitSphere Coach Web App

A log of the architecture and technology decisions for the FitSphere Coach web app: what was decided, why, and what was rejected. Add a new entry whenever a decision is made or changed; don't delete old ones — mark them **Superseded** and link the replacement.

**Status values:** ✅ Accepted · 🟡 Proposed (needs confirmation) · ❓ Open · ⛔ Superseded

Related docs: `CLAUDE.md` (rules for Claude Code), `docs/FITSPHERE_COACH_WEBAPP.md` (requirements), [UI/UX design](https://claude.ai/artifact/S1BBcDkWgLnqPnNPLjfqsH?sk=8sowdQAFUHY3z5iDAv7WuA#page-dcb81aee8d83).

---

## Summary

| # | Decision | Status | Date |
|---|---|---|---|
| [D-01](adr/ADR-001-shared-nodejs-api.md) | One shared Node.js API for the web app and the Android app | ✅ Accepted | Nov 2026 |
| [D-02](adr/ADR-002-separate-web-repo.md) | Separate repo for the coach web app | ✅ Accepted | Nov 2026 |
| [D-03](adr/ADR-003-supabase-postgres-database.md) | Supabase Postgres as the database; reuse existing tables | ✅ Accepted | Nov 2026 |
| [D-04](adr/ADR-004-web-app-talks-only-to-api.md) | Web app talks only to the API, never directly to Supabase | ✅ Accepted | Nov 2026 |
| [D-05](adr/ADR-005-multi-tenancy-gym-id.md) | Multi-tenancy by `gym_id`, enforced in the API and by RLS | ✅ Accepted | Nov 2026 |
| [D-06](adr/ADR-006-frontend-react-vite-typescript.md) | Frontend: React + Vite + TypeScript (static) — "Stack 1" | ✅ Accepted | Nov 2026 |
| [D-07](adr/ADR-007-hosting-and-cicd.md) | Hosting: Hostinger (web) + Google Cloud Run (API) | ✅ Accepted | Nov 2026 |
| [D-08](adr/ADR-008-ui-tailwind-shadcn.md) | UI: Tailwind CSS v4 + shadcn/ui + Lucide + shadcn Charts | ✅ Accepted | Nov 2026 |
| [D-09](adr/ADR-009-calendar-fullcalendar-dndkit.md) | Calendar: FullCalendar Standard + dnd-kit | ✅ Accepted | Nov 2026 |
| [D-10](adr/ADR-010-frontend-data-layer.md) | Data layer: TanStack Router / Query / Table + React Hook Form + Zod | ✅ Accepted | Nov 2026 |
| [D-11](adr/ADR-011-server-side-tables.md) | Tables: server-side sort/filter/paging in Supabase, shown with TanStack Table | ✅ Accepted | Nov 2026 |
| [D-12](adr/ADR-012-csv-export.md) | CSV export: browser for selected rows, server for full export | ✅ Accepted | Nov 2026 |
| [D-13](adr/ADR-013-media-storage.md) | Media: images on Cloudflare R2 (+ Cloudflare Images for resizing), exercise videos as external links | ✅ Accepted (updated) | Nov 2026 |
| [D-14](adr/ADR-014-backend-standards.md) | Backend standards: TypeScript, Zod, OpenAPI, Supabase CLI migrations | ✅ Accepted | Nov 2026 |
| [D-15](adr/ADR-015-testing-and-quality.md) | Testing and quality tooling | ✅ Accepted | Nov 2026 |
| [D-16](adr/ADR-016-ai-assisted-development-claude-code.md) | AI-assisted development with Claude Code | ✅ Accepted | Nov 2026 |
| [D-17](adr/ADR-017-cost-plan.md) | Cost plan: free tiers while building, Supabase Pro at go-live | ✅ Accepted | Nov 2026 |
| [D-18](adr/ADR-018-designs-as-claude-artifacts.md) | Designs delivered as Claude artifacts (not Canva) | ✅ Accepted | Nov 2026 |
| [D-19](adr/ADR-019-mvp-scope.md) | MVP scope and what's left out | ✅ Accepted | Nov 2026 |
| [D-20](adr/ADR-020-roles-admin-and-coach.md) | Roles: Admin and Coach | 🟡 Proposed | Nov 2026 |
| [D-21](adr/ADR-021-calendar-inside-each-client.md) | Calendar lives inside each client (not a top-level tab) | 🟡 Proposed | Nov 2026 |
| [D-22](adr/ADR-022-auth-provider.md) | Auth provider: Supabase Auth, same as the Android app | 🟡 Proposed | Nov 2026 |

---

## D-01 · One shared Node.js API for both apps
**Status:** ✅ Accepted

**Context:** The Android client app already has a Node.js API hosted on Google Cloud. The coach web app works on the same data: a coach assigns a workout, the client logs it; autonomy, streaks and "Needs attention" depend on both sides.

**Decision:** Extend the existing Node.js API with coach modules and use it for both apps.

**Why:** One place for business rules and security; both apps always behave the same; no duplicate auth, hosting or testing.

**Rejected:**
- *Separate coach API on the same database* — rules written twice and drifting apart, two services writing the same tables.
- *Rewrite the API from scratch* — it's already live; only worth it if the existing backend is seriously flawed.

**Consequences:** Changes must stay backward-compatible for installed Android builds. Plan to move the backend out of the Android repo into its own repo (`fitsphere-api`) with care, since its deploy pipeline is tied to the current repo.

---

## D-02 · Separate repo for the coach web app
**Status:** ✅ Accepted

**Decision:** New repo for the web app (e.g. `fitsphere-coach-web`). The API stays in its own repo.

**Why:** Different stack, release cycle and `CLAUDE.md` from the Android app; clear ownership.

**Consequences:** The web app gets its typed API client generated from the API's OpenAPI spec.

---

## D-03 · Supabase Postgres as the database
**Status:** ✅ Accepted

**Decision:** Keep using Supabase Postgres. Reuse the existing tables (`workout`, `nutrients`, `exercise`); create new tables (gyms, coaches, programs, calendar activities, notifications, etc.) in the same Supabase account.

**Why:** Already in use by the Android app; one source of truth.

**Consequences:** All schema changes go through versioned migrations (see D-14).

---

## D-04 · Web app talks only to the API
**Status:** ✅ Accepted

**Decision:** The browser calls the Node.js API only. It never queries Supabase directly. The Supabase secret key lives only in the API.

**Why:** One place for tenant isolation, roles and business rules (OWASP A01 Broken Access Control is the top risk for this app).

**Rejected:** Calling Supabase from the browser with RLS alone — splits business logic between frontend and database.

---

## D-05 · Multi-tenancy by `gym_id`
**Status:** ✅ Accepted

**Decision:** Every table holding gym data has a `gym_id`. The API takes `gym_id` from the logged-in user (never from the request), checks permissions in one policy module (role × action × resource), and Supabase **Row Level Security** acts as a second line of defence.

**Consequences:** Every API endpoint needs a test proving a coach from another gym gets 403/404. Indexes on `gym_id`, `client_id`, `coach_id` and date columns.

---

## D-06 · Frontend: React + Vite + TypeScript (Stack 1)
**Status:** ✅ Accepted

**Options compared:**

| | Stack 1: React + Vite (static) on Hostinger | Stack 2: Next.js on Hostinger Node.js | Stack 3: Angular on Firebase |
|---|---|---|---|
| Traffic handling | ★★★ | ★★☆ | ★★★ |
| Hosting cost | $0 (existing plan) | $0 (Business/Cloud plan only) | $0 (free tier) |
| Time to MVP | Fastest | Medium | Medium |
| Complexity | Low | Medium (runs a server) | Medium |

**Decision:** Stack 1.

**Why:** The app sits behind a login, so server rendering/SEO adds nothing; static files are the cheapest and easiest to scale; uses the Hostinger plan already paid for; fastest to build with Claude Code.

**Revisit if:** we need a backend-for-frontend layer or a public marketing site in the same app (then consider Next.js).

---

## D-07 · Hosting
**Status:** ✅ Accepted

**Decision:**
- Web app → **Hostinger** (existing plan, on a subdomain), static files with long cache headers.
- API → **Google Cloud Run** (auto-scales, scales to zero, free tier covers ~2M requests/month).
- Secrets → **Google Secret Manager**.
- CI/CD → **GitHub Actions** (lint → typecheck → test → deploy).

**Why:** Minimum or zero hosting cost while handling the expected traffic.

---

## D-08 · UI: Tailwind CSS + shadcn/ui
**Status:** ✅ Accepted

**Decision:** Tailwind CSS v4 (CSS-first `@theme` tokens) for styling; shadcn/ui (on Radix) for components; Lucide icons; shadcn Charts (Recharts) for the dashboard.

**Why:** Components are copied into our repo, so we own and edit them; Radix gives accessibility (keyboard, focus, screen readers) for WCAG 2.2; Claude Code works well with it; small CSS bundle.

**Trade-offs:** We maintain the component code ourselves; long class lists should be pulled into components.

---

## D-09 · Calendar: FullCalendar Standard + dnd-kit
**Status:** ✅ Accepted

**Decision:**
- **FullCalendar Standard** (MIT, free) for week/month views, moving activities between days and clicking a day to add.
- **dnd-kit** for reordering exercises, building supersets and dragging from the exercise library.
- shadcn menus/dialogs for copy, delete, multi-select and the Lifestyle view.

**Why:** shadcn's Calendar is only a date picker. FullCalendar gives ready calendar views; dnd-kit gives accessible drag-and-drop with a keyboard sensor.

**Constraints:** Don't use FullCalendar Premium (paid commercial licence). Every drag action needs a keyboard "Move to… / Copy to…" alternative (WCAG 2.5.7). Theme FullCalendar with Tailwind variables.

---

## D-10 · Frontend data layer
**Status:** ✅ Accepted

**Decision:** TanStack Router (routing), TanStack Query (server data and caching), TanStack Table (tables), React Hook Form + Zod (forms), typed API client generated from OpenAPI.

**Why:** Type-safe end to end; Query caching cuts API calls (lower Supabase egress cost).

---

## D-11 · Tables: Supabase does the work, TanStack Table shows it
**Status:** ✅ Accepted

**Decision:** Sorting, filtering and pagination run in Supabase via the API (e.g. 25 rows per page). TanStack Table runs in server-side mode (`manualSorting`, `manualFiltering`, `manualPagination`) and handles display and bulk select.

**Rejected:** Using Supabase's Table Editor — it's a developer/admin tool, not something coaches use.

---

## D-12 · CSV export
**Status:** ✅ Accepted

**Decision:** Coach chooses the fields. Selected rows or the current page are exported in the browser; "export all clients" is generated by the API and streamed back (background job for big gyms, rate-limited).

---

## D-13 · Media storage
**Status:** ✅ Accepted

**Decision (updated Nov 2026):**
- **Images** (exercise thumbnails, coach/client uploads in calendar threads, profile photos) are stored on **Cloudflare R2**.
- **Cloudflare Images** (transformations) resizes and converts them on the fly (thumbnail sizes, WebP/AVIF), reading from R2.
- **Exercise demo videos** stay as external links (YouTube unlisted / Vimeo).
- **Supabase** stores only the file **key and metadata** (owner, `gym_id`, client, activity, size, type) — not the file itself.

**How uploads work:**
1. Web app asks the Node.js API for an upload URL.
2. API checks the user's gym and permission, then returns a short-lived **presigned R2 upload URL** (R2 is S3-compatible, so the AWS S3 SDK works).
3. Browser uploads the file straight to R2 (the file never passes through Cloud Run).
4. Web app tells the API the upload is done; API saves the key and metadata in Supabase.

**Buckets:**
| Bucket | Contents | Access |
|---|---|---|
| `fitsphere-public` | Exercise library thumbnails, prebuilt program images | Public, via a custom domain with Cloudflare cache |
| `fitsphere-private` | Client and coach uploads (thread images, progress photos) | Private; API returns short-lived **signed URLs** after a permission check |

Object keys are prefixed by gym: `gyms/{gym_id}/clients/{client_id}/…` so files are easy to find, audit and delete per gym.

**Why:**
- R2 charges **no egress fees**, so serving images to coaches and clients doesn't grow the bill with traffic (egress was the main Supabase cost risk).
- Free tier: **10 GB storage, 1M writes and 10M reads per month**; after that $0.015/GB-month.
- Cloudflare Images Free plan includes **5,000 unique transformations a month**, then $0.50 per 1,000.
- Keeps Supabase egress for database traffic only, which delays the need for a bigger Supabase plan.

**Rejected:**
- *Supabase Storage* (previous decision) — 1 GB storage and 5 GB egress on the free plan; image traffic would eat the same egress quota as the database.
- *Storing images in Cloudflare Images itself* — paid plan only ($5 per 100,000 stored + $1 per 100,000 delivered); R2 + transformations is cheaper at our size.
- *Hosting videos on R2* — possible later, but large video files would use up the free storage fast and need streaming; external links are enough for the MVP.

**Consequences:**
- New secrets in Google Secret Manager: R2 account ID, access key, secret key, bucket names. Never in the web app.
- Keep the number of image sizes small (e.g. 3 widths) — each width counts as a unique transformation.
- Set CORS on the R2 bucket to allow uploads only from the web app's domain.
- Deleting a client or gym must also delete their R2 objects.

**Open:** max upload size for images in calendar threads (see `docs/FITSPHERE_COACH_WEBAPP.md` → Open questions).

---

## D-14 · Backend standards
**Status:** ✅ Accepted

**Decision:**
- Node.js LTS (pinned in `.nvmrc` and `engines`); TypeScript strict, converting existing JS file by file.
- Keep the existing framework; add modules under `src/modules/{dashboard, clients, calendar, programs, coaches, notifications}` (routes → service → repository).
- Zod on every request; OpenAPI generated from Zod.
- Database access through the Supabase **connection pooler**.
- Migrations with **Supabase CLI** in the backend repo; `supabase gen types typescript` for shared types.
- One error format; structured JSON logs to Cloud Logging.
- Rate limits on login, bulk assign and CSV export; paginate every list.

---

## D-15 · Testing and quality
**Status:** ✅ Accepted

**Decision:** Vitest + React Testing Library + MSW (frontend), Vitest + Supertest against local Supabase (API), Playwright (E2E), axe (accessibility, WCAG 2.2 AA), ESLint with `no-deprecated` + Prettier, all run in GitHub Actions.

---

## D-16 · AI-assisted development with Claude Code
**Status:** ✅ Accepted

**Decision:** Build with Claude Code. One `CLAUDE.md` per repo. Context7 MCP for current library docs (to avoid deprecated APIs). Stop hook runs typecheck + lint + tests. AI-written code is reviewed like any human change.

---

## D-17 · Cost plan
**Status:** ✅ Accepted

| Service | While building | At go-live |
|---|---|---|
| Hostinger | Existing plan | Existing plan |
| Cloud Run | Free tier | Free tier / a few dollars |
| Supabase | Free | **Pro (~$25/month)** — more egress, projects not paused |
| Cloudflare R2 + Images | Free (10 GB, no egress fees, 5,000 transformations) | Free tier; then $0.015/GB-month and $0.50 per 1,000 transformations |
| GitHub + Actions | Free | Free |

Google Cloud budget alerts are set up from day one.

---

## D-18 · Designs as Claude artifacts
**Status:** ✅ Accepted

**Decision:** UI/UX designs are shared as Claude artifact links, not Canva. Current design: [FitSphere Coach — Web](https://claude.ai/artifact/S1BBcDkWgLnqPnNPLjfqsH?sk=8sowdQAFUHY3z5iDAv7WuA#page-dcb81aee8d83).

---

## D-19 · MVP scope
**Status:** ✅ Accepted

**In:** Dashboard (Needs attention), Clients (list, streaks, summary, services, autonomy, CSV export, audit trail), client calendar (actions, supersets, threads, media, Lifestyle view, trainings), Programs (incl. prebuilt, bulk assign), Coaches, Notifications.

**Out (future):** client/task priorities, white-label branding and invite links, slot booking, UPI payments, wearables, chat and broadcast, check-in forms, challenges/badges, Indian food database, %1RM and injury modifiers, assistant-coach role, waivers, multi-language.

---

## D-20 · Roles: Admin and Coach
**Status:** 🟡 Proposed

**Proposal:** Two roles for MVP. **Admin** sees the whole gym and manages coaches; **Coach** sees only their own clients. Dashboard content follows the role.

**To confirm:** whether an Assistant-coach role is needed in MVP.

---

## D-21 · Calendar inside each client
**Status:** 🟡 Proposed

**Context:** An early scope note listed tabs as Dashboard, Clients, Calendar, Programs. The requirements doc lists Dashboard, Clients, Coaches, Programs, with the calendar inside each client.

**Proposal:** Follow the requirements doc — calendar opens from a client's profile; Coaches is a top-level tab.

**To confirm** with the product owner.

---

## D-22 · Auth provider
**Status:** 🟡 Proposed (Supabase Auth confirmed as the Android app's provider; see [ADR-022](adr/ADR-022-auth-provider.md))

**Proposal:** Supabase Auth, the same provider as the Android app. Options considered: Firebase Auth via Supabase third-party auth, another managed provider (custom auth rejected).

**Rule regardless of choice:** the API verifies the JWT on every request and loads the user's `gym_id` and role.

---

## How to add a decision

```markdown
## D-XX · Title
**Status:** 🟡 Proposed | ✅ Accepted | ❓ Open | ⛔ Superseded by D-YY
**Context:** what problem or question led to this
**Decision:** what we chose
**Why:** the main reasons
**Rejected:** other options and why not
**Consequences:** what this means for the code, cost or team
```

Also add a row to the Summary table.
