# FitSphere Coach — 3. Design Guide & UI Design

Part 3 of 5 in the FitSphere Coach engineering guide. The app targets WCAG 2.2 level AA and "good" Core Web Vitals; items marked *(Recommendation)* are architect guidance.

## 1. Accessibility — WCAG 2.2, level AA

- WCAG 2.2 is the current W3C Recommendation (October 2023, updated December 2024) and the W3C advises using it over 2.0/2.1. It is backward compatible: content meeting 2.2 also meets 2.1 and 2.0.
- 2.2 adds 9 success criteria over 2.1 and drops 4.1.1 Parsing as obsolete.
- Four principles: Perceivable, Operable, Understandable, Robust (POUR).

New 2.2 criteria that directly affect FitSphere Coach screens:

| WCAG 2.2 criterion | Where it hits FitSphere Coach | What we do |
|---|---|---|
| 2.5.7 Dragging Movements (AA) | Calendar: move/copy activities, reorder exercises, build supersets | Every drag action has a single-pointer alternative ("Move to…", "Copy to…" menu, arrow buttons) |
| 2.5.8 Target Size (Minimum) (AA) | Calendar cells, icon buttons, table row actions | Targets at least 24×24 CSS px (or enough spacing) |
| 2.4.11 Focus Not Obscured (Minimum) (AA) | Sticky headers, toasts, notification drawer | Focused element is never fully hidden |
| 3.3.8 Accessible Authentication (Minimum) (AA) | Coach login | Allow password managers and paste; no cognitive puzzles |
| 3.3.7 Redundant Entry (A) | Multi-step program builder, client onboarding | Don't make coaches re-type data already entered |

Also required: keyboard access everywhere, visible focus, colour contrast at least 4.5:1 for text, labels on every form field, meaningful alt text for exercise images, captions for exercise videos.

## 2. Performance — Core Web Vitals

Measured at the 75th percentile of real users (field data).

| Metric | Measures | Good | Poor |
|---|---|---|---|
| LCP — Largest Contentful Paint | Loading | ≤ 2.5 s | > 4.0 s |
| INP — Interaction to Next Paint | Responsiveness | ≤ 200 ms | > 500 ms |
| CLS — Cumulative Layout Shift | Visual stability | ≤ 0.1 | > 0.25 |

FitSphere focus *(Recommendation)*: INP matters most — the calendar, client tables and bulk-select are interaction-heavy. Virtualise long client lists, paginate server-side, lazy-load exercise videos and thumbnails, reserve space for images to avoid layout shift, and track real-user vitals with the `web-vitals` library.

## 3. Design system *(Recommendation)*

- Design tokens for colour, typography, spacing, radius and elevation; light and dark themes from the same tokens.
- One component library (built on an accessible headless base) used everywhere; no one-off styled components.
- Every data view defines four states: **loading** (skeleton), **empty** (with next action), **error** (with retry), and **populated**.
- Consistent patterns: one table pattern (sort, filter, bulk select, CSV export), one form pattern (inline validation), one confirmation pattern for destructive actions (delete activity, remove client).
- Document the system in a `design.md` in the repo so Claude Code and developers produce consistent UI.

## 4. FitSphere screen guidelines *(Recommendation)*

- **Dashboard:** "Needs attention" first, each item showing why it was flagged and one direct action.
- **Clients:** streak and summary visible in the list; autonomy toggle clearly labelled with what ON and OFF mean.
- **Calendar:** week/month views, Lifestyle view visually distinct from training, multi-select with a visible selection count, undo for move/copy/delete.
- **Programs:** clear distinction between own and prebuilt programs; bulk-assign shows a preview of affected clients before confirming.
- **Notifications:** compact by default, expandable for details, mark read/unread.
- Responsive down to tablet width (coaches on the gym floor).

## 5. Design workflow with Claude *(Recommendation)*

- Give Claude the mockup or screenshot and ask for the full scope ("empty and error states, keyboard navigation, not just the layout").
- Use `/verify` or a browser screenshot next to the mockup to check the result visually.
- Run automated accessibility checks (axe) in component and E2E tests, plus a manual keyboard pass per release.

## 6. Nielsen's 10 usability heuristics (Nielsen Norman Group)

The most widely used checklist for evaluating any interface. Use it in every design review.

| # | Heuristic | FitSphere Coach example |
|---|---|---|
| 1 | Visibility of system status | Show saving/saved state, progress of bulk assign and CSV export |
| 2 | Match between system and the real world | Use coach language (sets, reps, supersets), not database terms |
| 3 | User control and freedom | Undo for move/copy/delete on the calendar; cancel any long action |
| 4 | Consistency and standards | Same table, filter and button patterns on every screen |
| 5 | Error prevention | Confirm before assigning a program to 50 clients; disable invalid choices |
| 6 | Recognition rather than recall | Recently used exercises, visible filters, labels on icons |
| 7 | Flexibility and efficiency of use | Keyboard shortcuts, bulk actions, copy-week for power users |
| 8 | Aesthetic and minimalist design | Dashboard shows only what needs action today |
| 9 | Help users recognise, diagnose and recover from errors | Plain-language messages that say what happened and how to fix it |
| 10 | Help and documentation | Contextual tips and an in-app help link for complex flows |

## 7. Visual design fundamentals (industry convention)

- **Visual hierarchy:** one primary action per screen; size, weight and colour signal importance.
- **Spacing system:** a fixed spacing scale (e.g. multiples of 4 or 8 px) and a layout grid.
- **Typography scale:** 1 font family, a small set of sizes and weights; body text at least 16 px.
- **Limited colour palette:** brand, neutral and semantic colours (success, warning, error, info) defined as tokens.
- **Colour is never the only signal (WCAG 1.4.1):** pair it with icons or text, e.g. "Needs attention" tags have a label, not just red.
- Icons with text labels for anything that isn't universally understood.
- Alignment and whitespace group related items and separate unrelated ones.

## 8. Interaction design patterns *(Recommendation)*

- Immediate feedback on every action; optimistic updates with rollback on failure.
- Destructive actions: confirmation, or better, undo.
- Inline form validation with clear messages; never clear the form on error.
- Sensible defaults (e.g. today's date, last-used program).
- Progressive disclosure: advanced options hidden until needed.
- Autosave for long editors (program builder) with a visible saved indicator.
- Skeleton loaders instead of spinners for content areas.

## 9. Forms, content and localisation *(Recommendation)*

- Visible labels (not placeholder-only); group related fields; mark optional fields rather than required ones when most are required.
- Error messages state the problem and the fix ("Enter reps between 1 and 100").
- Consistent date, time and number formats from the user's locale; units configurable (kg/lb, km/mi).
- Externalise UI strings from day one so the app can be translated later.
- Write microcopy in short, active, friendly sentences.

## 10. Semantic HTML and ARIA (W3C)

- Use native HTML elements first (`button`, `a`, `label`, `table`, `dialog`); they come with keyboard and screen-reader support.
- Add ARIA only when no native element fits, following the WAI-ARIA Authoring Practices patterns for complex widgets (grid for the calendar, menu, combobox for exercise search, dialog, tabs).
- Manage focus on route changes and when dialogs open or close.

## 11. Responsive and cross-browser *(Recommendation)*

- Design mobile/tablet-up with defined breakpoints; test calendar and tables at each.
- Support current and previous versions of evergreen browsers (Chrome, Edge, Firefox, Safari).
- Touch-friendly targets and no hover-only actions.

## 12. UX process *(Recommendation)*

1. **Research:** interview a few real coaches and gym admins; map their daily workflow.
2. **Define:** personas (coach, head coach/admin), user flows for each section.
3. **Design:** wireframes → high-fidelity mockups using the design tokens.
4. **Validate:** usability test prototypes with 3–5 coaches; fix the top issues.
5. **Handoff:** specs, tokens and states documented; developers and Claude build from the same `design.md`.
6. **Measure:** analytics on key tasks and real-user Core Web Vitals after release.

## References

- W3C — WCAG 2.2 Recommendation: https://www.w3.org/TR/WCAG22/
- W3C WAI — What's New in WCAG 2.2: https://www.w3.org/WAI/standards-guidelines/wcag/new-in-22/
- W3C WAI — ARIA Authoring Practices Guide: https://www.w3.org/WAI/ARIA/apg/
- Nielsen Norman Group — 10 Usability Heuristics for User Interface Design: https://www.nngroup.com/articles/ten-usability-heuristics/
- web.dev — Web Vitals: https://web.dev/articles/vitals
- web.dev — How the Core Web Vitals thresholds were defined: https://web.dev/articles/defining-core-web-vitals-thresholds
- Anthropic — Best practices for Claude Code (specific prompts, screenshots, /verify): https://code.claude.com/docs/en/best-practices
