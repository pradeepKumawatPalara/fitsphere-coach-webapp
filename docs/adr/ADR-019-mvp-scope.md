# ADR-019 · MVP scope and what is left out

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-19 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | Features in and out of the first release |
| **Risk tier** | Medium |

## Context

A fixed MVP scope limits cost and risk and keeps the architecture simple. Scope is a business decision, so this ADR does not use the weighted technical scoring; it records what is in, what is out, and what the scope means for the architecture.

## Options considered

| Option | Description | Assessment |
|---|---|---|
| A. Defined MVP (below) | Dashboard, Clients, client calendar, Programs, Coaches, Notifications | Covers the coach's core loop: plan, assign, track, follow up |
| B. Larger MVP | Adds chat, check-in forms, challenges, payments, booking, wearables and so on | More cost and risk; needs realtime, payments and compliance work before launch |
| C. Smaller MVP | Dashboard, Clients and calendar only | Coaches could not build or bulk-assign programs, so the product would not replace their current tools |

## Decision

**Option A.**

**In:** Dashboard ("Needs attention"); Clients (list, streaks, summary, services, autonomy, CSV export, audit trail); client calendar (actions, supersets, threads, media, Lifestyle view, trainings); Programs (including prebuilt, bulk assign); Coaches; Notifications.

**Out (future):** client/task priorities, white-label branding and invite links, slot booking, UPI payments, wearables, chat and broadcast, check-in forms, challenges and badges, Indian food database, %1RM and injury modifiers, assistant-coach role, waivers, multi-language.

## Why

- Delivers the full coach workflow without payments, realtime chat or wearable integrations.
- Keeps compliance, hosting and testing effort within budget (ADR-017).

## Rejected

- **B. Larger MVP:** too much risk before the core is proven.
- **C. Smaller MVP:** not enough value for coaches.

## Consequences

- Claude does not build out-of-scope features unless asked (stated in `CLAUDE.md`).
- No chat means no realtime messaging infrastructure for now; workout message threads and notifications are in scope.
- Adding scope later needs a new entry in `docs/DECISIONS.md` and an ADR if it changes the architecture.
- The assistant-coach role is out of scope; the roles decision is ADR-020.

## Related decisions

ADR-016, ADR-017, ADR-018, ADR-020, ADR-021.

## Open points

- Clarify the boundary between "each workout has its own message thread" (in) and "chat" (out), so threads are not built as general chat.
- Multi-language is out, but externalising UI strings from day one is cheap and recommended (`docs/best-practices/03-design-guide-and-ui-design.md`); confirm.
- Acceptance criteria per feature (success path, failure path, role permissions) to be written before each feature starts.
