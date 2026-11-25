# ADR-009 · Calendar: FullCalendar Standard + dnd-kit

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-09 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Client calendar views and drag-and-drop in the workout builder |
| **Risk tier** | Medium (accessibility and licensing are the main risks) |

## Context

The client calendar needs week and month views, moving activities between days, click-to-add, multi-select, copy and delete, and a Lifestyle view. The workout builder needs reordering exercises, building supersets and dragging from the exercise library. shadcn's Calendar is only a date picker. WCAG 2.5.7 requires a non-drag alternative for every drag action.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. FullCalendar Standard (MIT) + dnd-kit | B. FullCalendar Premium plugins | C. Other open-source calendar library + dnd-kit |
|---|---|---|---|
| Security / privacy / legal gate | Pass | **Fail:** paid commercial licence, not budgeted | Pass |
| Security (20%) | 4 | n/a | 4 |
| Maintainability (20%) | 4 | n/a | 3 |
| Reliability (15%) | 4 | n/a | 3 |
| Performance (15%) | 3 | n/a | 3 |
| Scalability (10%) | 3 | n/a | 3 |
| Cost (10%) | 5 | n/a | 5 |
| Integration (10%) | 4 | n/a | 3 |
| **Weighted score** | **3.85** | excluded (failed gate) | **3.40** |

## Decision

**Option A.**
- **FullCalendar Standard** (free, MIT) for week/month views, moving activities between days and clicking a day to add.
- **dnd-kit** for reordering exercises, building supersets and dragging from the exercise library.
- shadcn menus and dialogs for copy, delete, multi-select and the Lifestyle view.

## Why

- Ready-made calendar views; dnd-kit gives accessible drag-and-drop with a keyboard sensor.
- Free and open-source; no licence cost.

## Rejected

- **B. Premium plugins:** paid commercial licence.
- **C. Other library:** more custom work for similar results.

## Consequences

- Use only `@fullcalendar/react`, `daygrid`, `timegrid` and `interaction`; no Premium plugins.
- Every drag action needs a keyboard alternative: a "Move to… / Copy to…" menu on activities and buttons to reorder exercises and link supersets (WCAG 2.5.7).
- Theme FullCalendar with Tailwind variables.
- Load one week or month range at a time; optimistic updates with rollback and an undo toast for move, copy and delete.
- Click and touch targets at least 24×24 CSS px (WCAG 2.5.8).

## Related decisions

ADR-008, ADR-010, ADR-021 (calendar inside each client).

## Open points

- **Verify keyboard and screen-reader behaviour of FullCalendar** against our WCAG 2.2 AA target early; if it cannot meet it, plan a custom accessible layer or a decision change.
- `@dnd-kit/react` is a newer package line; confirm its current stable version and API via Context7 before use.
- Performance check (INP) with a month of dense activities.
