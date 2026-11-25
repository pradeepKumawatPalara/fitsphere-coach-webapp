# FitSphere Coach Web App — Project Charter (Step 1: Prepare)

| | |
|---|---|
| **Status** | 🟡 Draft — needs business-owner sign-off before any feature code is written |
| **Date** | Nov 2026 |
| **Related** | `docs/FITSPHERE_COACH_WEBAPP.md` (requirements), `docs/DECISIONS.md` (decisions), `docs/STANDARD_IT_TESTING_RULES.md` (testing standard), `docs/ACCEPTANCE_CRITERIA.md` |

No code is written until the business owner has signed off on goals, owners and risk tier (section 5).

## 1. Business goal

Give gym admins and coaches in India one web app to plan workouts, nutrition and lifestyle targets for their clients and to track how well clients stick to the plan. Whatever a coach plans appears in the live FitSphere Android client app, and whatever a client logs appears in the web app, because both use the same API and database.

**Success looks like** *(measures to be set by the business owner, per REQ-002 and REQ-005)*:
- Coaches can complete the core loop in the web app: find a client, plan a calendar, assign a program, follow up on "Needs attention".
- No client data is ever visible to a user from another gym.
- Targets to confirm: number of pilot gyms, coaches and clients at launch, and a date. **TBC**

## 2. Target users

| Role | Who | Web access |
|---|---|---|
| Admin | Gym owner or manager | Yes: whole gym, manages coaches and roles |
| Coach | Trainer in a gym | Yes: own clients, calendars and programs |
| Client | Gym member | No: Android app only |

Open: whether an Assistant-coach role is needed (ADR-020, proposed).

## 3. First features (build order)

Build one feature at a time. Acceptance criteria for the first four are drafted in `docs/ACCEPTANCE_CRITERIA.md`; the rest get criteria approved before each one starts (REQ-009).

| # | Feature | Status of criteria |
|---|---|---|
| F-01 | Sign in and sign out | Drafted (blocked on ADR-022) |
| F-02 | Gym isolation and role access (applies to every feature) | Drafted |
| F-03 | Client list: search, sort, filter, paging, streak | Drafted (streak rules to confirm) |
| F-04 | CSV export of clients | Drafted (field list to confirm) |
| F-05 | Dashboard: "Needs attention" | To write (tag rules to confirm) |
| F-06 | Client profile, summary and autonomy toggle | To write |
| F-07 | Client calendar and Lifestyle view | To write |
| F-08 | Workout builder and supersets | To write |
| F-09 | Programs and bulk assign | To write |
| F-10 | Coaches: add coach, assign role | To write |
| F-11 | Notifications | To write |

Out of MVP scope: see ADR-019 and requirements section 8.

## 4. Owners and responsibilities (GOV-004, GOV-014)

Names are blank on purpose; the business owner fills them in.

| Role | Person | Accountable for |
|---|---|---|
| Business owner | *TBC* | Goals, scope, risk acceptance, accepting ADRs, release decision (GOV-016) |
| Product owner | *TBC* | Requirements, acceptance criteria, open questions |
| Tech lead | *TBC* | Architecture, answering Claude's clarifying questions, code review |
| QA lead | *TBC* | Test strategy, test results, defect triage |
| Security reviewer | *TBC* | Security review of login, tenant isolation, uploads, exports, before a feature is accepted |
| Owner of the testing standard (GOV-004) | *TBC* | Maintaining `STANDARD_IT_TESTING_RULES.md`, review at least yearly |

Responsibility matrix (GOV-014): **R** = does the work, **A** = accountable, **C** = consulted.

| Area | Business owner | Product owner | Tech lead | QA lead | Security reviewer |
|---|---|---|---|---|---|
| Test strategy | C | C | C | A/R | C |
| Test design | | C | C | A/R | C |
| Test execution | | | R | A | |
| Test automation | | | R | A | |
| Environments | | | A/R | C | C |
| Test data (synthetic only) | | | R | A | C |
| Defect triage | C | R | R | A | C |
| Security testing | | | R | C | A |
| Release recommendation | A (decides) | C | R | R | R |

## 5. Risk tier (Annex 2 of the testing standard)

**Proposed: Tier 2 — High.** The tier is the highest level triggered by any factor.

| Factor | Assessment | Tier |
|---|---|---|
| Data sensitivity | Personal data of clients (name, date of birth, phone, email) plus workout, nutrition and lifestyle data | 2 |
| User reach | Many external users across several gyms | 2 |
| Recoverability | A cross-gym data leak cannot be undone | 2 |
| Safety | Training and nutrition plans have indirect safety relevance | 2 |
| Legal / regulatory | Personal data of users in India; applicable obligations to be confirmed by the business owner | 2 (TBC) |
| Financial impact | Payments are out of MVP scope | 3 |
| Autonomy | Humans make all decisions; "Needs attention" tags are advisory | 3 |
| Integration footprint | Shares one API and database with the live Android app | 2 |

**Consequences of Tier 2** (testing standard, Annex 2): independent review recommended; bidirectional traceability from requirements to tests (REQ-015); threat model, static analysis, dependency scanning and dynamic testing; risk-selected non-functional testing with targets; formal evidence summaries.

## 6. Acceptance criteria

Agreed by the business owner before a feature is ready for development (REQ-009). Each covers the success path, failure paths and which roles can and cannot act (REQ-011, REQ-012). See `docs/ACCEPTANCE_CRITERIA.md`.

## 7. Open items to resolve before or during Step 2

1. Fill in the names in section 4.
2. Confirm the risk tier in section 5, including any legal obligations for personal data of users in India.
3. Close ADR-022 (auth provider), then review ADR-020 and ADR-021.
4. Answer the requirement open questions that block F-03 to F-05 (streak rules, "Needs attention" rules, CSV fields).
5. Requirements doc section 9.2 still lists Supabase Storage for images; ADR-013 changed this to Cloudflare R2. Update the requirements doc to match.

## 8. Sign-off

| Role | Name | Decision (approve / changes needed) | Date |
|---|---|---|---|
| Business owner | | | |
| Tech lead | | | |
| QA lead | | | |
| Security reviewer | | | |
