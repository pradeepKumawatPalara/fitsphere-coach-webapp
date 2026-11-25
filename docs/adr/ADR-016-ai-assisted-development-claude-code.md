# ADR-016 · AI-assisted development with Claude Code

| | |
|---|---|
| **Status** | ✅ Accepted (decision made by the business owner; recorded retroactively as an ADR) |
| **Log entry** | D-16 in `docs/DECISIONS.md` |
| **Date** | Nov 2026 |
| **Owner / approver** | Business owner *(name and approval date to be filled in)* |
| **Scope** | How the team uses Claude Code to build the web app |
| **Risk tier** | Medium–High (AI can introduce insecure or deprecated code; tools can read the repo) |

## Context

The team builds the web app with Claude Code. Claude's knowledge has a cutoff and can suggest deprecated APIs; instructions in `CLAUDE.md` are followed most of the time but not guaranteed, while hooks, deny rules and CI are enforced. The framework criteria were designed for technical architecture, so they are applied loosely here.

## Options considered

Scores are 1–5 per criterion (5 = best) using the framework weights. **Security, privacy and legal are pass/fail first.** Scores are the architect's proposal for owner review.

| Criterion (weight) | A. Claude Code with rules, current-docs access, enforced checks and human review | B. No AI assistance | C. AI assistance with only `CLAUDE.md` guidance (no enforcement) |
|---|---|---|---|
| Security / privacy gate | Pass, with conditions (deny rules, trusted skills and MCP servers) | Pass | At risk: no enforcement of secrets or quality rules |
| Security (20%) | 3 | 4 | 2 |
| Maintainability (20%) | 4 | 3 | 3 |
| Reliability (15%) | 4 | 4 | 2 |
| Performance (15%) | 4 | 3 | 4 |
| Scalability (10%) | 4 | 3 | 3 |
| Cost (10%) | 4 | 2 | 4 |
| Integration (10%) | 4 | 3 | 4 |
| **Weighted score** | **3.80** | **3.25** | **3.00** |

## Decision

**Option A.** Build with Claude Code:
- One `CLAUDE.md` per repo.
- Context7 MCP for current library docs (to avoid deprecated APIs).
- A Stop hook that runs typecheck, lint and tests.
- AI-written code is reviewed like any human change.

## Why

- Faster delivery with guardrails that are enforced, not just requested.
- Matches `docs/best-practices/04-avoiding-deprecated-apis-and-libraries.md` and `05-claude-code-skills.md`.

## Rejected

- **B. No AI assistance:** slower and more expensive for the same output.
- **C. Guidance only:** `CLAUDE.md` alone is not enforced.

## Consequences

- Keep `CLAUDE.md` short (under about 200 lines); put procedures in skills and enforced rules in hooks and deny rules.
- Add deny rules for `.env`, `.env.local` and `secrets/**`, and a PostToolUse formatter hook, once the project is scaffolded.
- The Stop hook is added after the first scaffold; before then it would fail every turn.
- A person other than the author reviews code before merge; run `/code-review` for an independent check.
- Never paste real client data into prompts; synthetic data only.
- Install third-party skills and MCP servers only from trusted publishers, and read them first (Context7 is a third-party server by Upstash).
- Ask Claude for evidence (command and output), not just "tests pass".

## Related decisions

ADR-015, ADR-017 (usage cost).

## Open points

- The skills package from the startup guide (architecture-and-design, api-and-data, testing-and-quality, reliability-and-observability, security-and-delivery) is not yet in the repo.
- Recommended custom skills (`fitsphere-ui`, `new-api-endpoint`, `calendar-feature`, `db-migration`, `release`) are not yet created.
- Who approves plugin and MCP installs, and any spend or turn limits for unattended runs.
