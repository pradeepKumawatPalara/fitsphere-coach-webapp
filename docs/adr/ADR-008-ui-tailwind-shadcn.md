# ADR-008 · UI: Tailwind CSS v4 + shadcn/ui + Lucide + shadcn Charts

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-08 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Styling, component library, icons and charts for the Coach web app |
| **Risk tier** | Medium |

## Context

The app needs many consistent components (tables, dialogs, menus, forms) that meet WCAG 2.2 AA (keyboard, focus, screen readers), light and dark themes from one set of tokens, and a dashboard with charts. We want to build quickly with Claude Code and avoid one-off styled components.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. Tailwind v4 + shadcn/ui (Radix) + Lucide + shadcn Charts | B. Material UI (MUI) | C. Headless components (Radix) + our own CSS |
|---|---|---|---|
| Security / privacy gate | Pass | Pass | Pass |
| Security (20%) | 4 | 4 | 4 |
| Maintainability (20%) | 3 | 4 | 2 |
| Reliability (15%) | 4 | 4 | 3 |
| Performance (15%) | 5 | 3 | 5 |
| Scalability (10%) | 4 | 4 | 4 |
| Cost (10%) | 5 | 5 | 4 |
| Integration (10%) | 4 | 3 | 3 |
| **Weighted score** | **4.05** | **3.85** | **3.50** |

Notes: B scores lower on integration because it brings its own styling system, which clashes with Tailwind and the design tokens; some advanced MUI components are paid. A scores 3 on maintainability because component code is copied into our repo and we maintain it.

## Decision

**Option A.** Tailwind CSS v4 (CSS-first `@theme` tokens, no `tailwind.config.js`), shadcn/ui on Radix, Lucide icons, shadcn Charts (Recharts).

## Why

- Components are copied into our repo, so we own and edit them.
- Radix provides keyboard, focus and screen-reader behaviour for WCAG 2.2.
- Works well with Claude Code; small CSS bundle.

## Rejected

- **B. MUI:** second styling system, larger bundle, some paid components.
- **C. Fully custom:** slow to build and easy to get accessibility wrong.

## Consequences

- We maintain the component code in `src/components/ui/`; pull long class lists into components.
- Design tokens (colour, type, spacing, radius, elevation) live in `@theme`; document them in `design.md` (see `docs/best-practices/03-design-guide-and-ui-design.md`).
- No inline hex colours and no CSS-in-JS; `cn()` for conditional classes.
- Charts need a text or table alternative so data is not conveyed by colour or visuals alone.
- Every data view has loading, empty, error and populated states.

## Related decisions

ADR-006, ADR-009 (calendar theming), ADR-018 (designs).

## Open points

- Confirm Tailwind v4's minimum browser versions match our browser support policy (verify against the current Tailwind docs).
- Confirm the chart accessibility approach (table view or summary text next to each chart).
