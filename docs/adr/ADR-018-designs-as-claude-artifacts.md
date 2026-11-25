# ADR-018 · UI/UX designs delivered as Claude artifacts (not Canva)

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-18 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | How UI/UX designs are produced, shared and handed to development |
| **Risk tier** | Low–Medium |

## Context

The current design is a Claude artifact: [FitSphere Coach — Web](https://claude.ai/artifact/S1BBcDkWgLnqPnNPLjfqsH?sk=8sowdQAFUHY3z5iDAv7WuA#page-dcb81aee8d83). `CLAUDE.md` tells Claude to match its layout, spacing and components. The framework criteria were designed for technical architecture, so they are applied loosely here.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. Claude artifacts | B. Canva | C. Figma |
|---|---|---|---|
| Security / privacy gate | Pass (link-based access; no real client data in designs) | Pass | Pass |
| Security (20%) | 3 | 4 | 4 |
| Maintainability (20%) | 3 | 2 | 4 |
| Reliability (15%) | 3 | 4 | 4 |
| Performance (15%) | 4 | 4 | 4 |
| Scalability (10%) | 3 | 4 | 4 |
| Cost (10%) | 4 | 3 | 3 |
| Integration (10%) | 5 | 2 | 4 |
| **Weighted score** | **3.45** | **3.30** | **3.90** |

**Architect note:** the score favours Figma for long-term design management. The owner's decision for artifacts rests on speed and direct hand-off to Claude Code with no extra licence. The decision stands, with the consequences below to avoid lock-in.

## Decision

**Option A.** UI/UX designs are shared as Claude artifact links, not Canva.

## Why

- Fast to produce and iterate; Claude Code can use the design directly.
- No extra tool or licence.

## Rejected

- **B. Canva:** suited to marketing graphics, weaker as a product UI design system.
- **C. Figma:** strong design management and tokens, but extra tool and licence; remains the likely upgrade path.

## Consequences

- The artifact is a prototype, not the source of truth. Capture design tokens, component rules and the four data states (loading, empty, error, populated) in a `design.md` in the repo (see `docs/best-practices/03-design-guide-and-ui-design.md`).
- Take a snapshot of the design (export or screenshots) into the repo so the design survives if the artifact changes or is deleted.
- The design must show empty and error states, keyboard behaviour and tablet width, or Claude is asked to add them (WCAG 2.2 AA).
- Do not put real client data in designs; share links only with people who need them.

## Related decisions

ADR-008 (UI stack), ADR-019 (scope), ADR-021 (calendar placement).

## Open points

- Confirm the artifact's navigation matches ADR-021 (Dashboard, Clients, Coaches, Programs, with the calendar inside each client).
- Who owns the artifact and who may edit it.
- Revisit if the design grows beyond what an artifact manages comfortably (then consider Figma).
