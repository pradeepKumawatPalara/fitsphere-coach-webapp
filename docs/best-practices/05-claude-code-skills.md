# FitSphere Coach — 5. Claude Code Skills: What to Use and How

Part 5 of 5 in the FitSphere Coach engineering guide. No skill is officially mandatory — skills are optional extensions. Below is the recommended setup for this project; items marked *(Recommendation)* are architect guidance, not Anthropic requirements.

## 1. What a skill is

- A skill is a folder containing at least a `SKILL.md` file: YAML frontmatter (`name`, `description`) plus instructions. It can also hold scripts, templates and reference docs.
- Claude Code skills follow the open Agent Skills standard (agentskills.io); Claude Code adds extras such as invocation control and subagent execution.
- Locations: `.claude/skills/<name>/SKILL.md` (this project, shared via git) or `~/.claude/skills/<name>/SKILL.md` (you, all projects).
- Loading: Claude sees every skill's name and description in each session, but reads the full file only when the task needs it — so skills cost less context than putting everything in `CLAUDE.md`.
- Custom slash commands have been merged into skills: `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both create `/deploy`.

## 2. Which tool for which job

| Mechanism | Use it for | Enforced? |
|---|---|---|
| `CLAUDE.md` | Commands (test, lint, typecheck), pinned versions, decisions not visible in code. Keep it under ~200 lines. | No — Claude decides |
| Skill | Team procedures and know-how loaded on demand (UI rules, endpoint checklist, release) | No — unless `disable-model-invocation: true` (only you can start it) |
| Hook | Rules with zero exceptions (typecheck before finishing, format after edit, block edits to migrations) | Yes — harness runs it |
| Permission deny rule | Files Claude must never read (`.env`, secrets) | Yes |
| MCP server | External systems and data (Context7 docs, issue tracker) | n/a |
| Subagent | Independent review, parallel work, read-only reviewer | Tool limits are enforced |

## 3. Built-in skills to use

- `/init` — generate a starting `CLAUDE.md`, then trim it.
- `/code-review` — reviews the diff in a fresh subagent that didn't see how the code was written.
- `/verify` — builds and runs the app and looks at the result; catches issues tests miss.
- `/doctor` and `/doctor prompt-audit` — find unused extensions and outdated or conflicting instructions.

## 4. Recommended custom skills for FitSphere Coach *(Recommendation)*

| Skill | Invoked by | Contents |
|---|---|---|
| `fitsphere-ui` | Claude (auto) | Design tokens, component rules, four data states, WCAG 2.2 checklist (drag alternatives, target size) |
| `new-api-endpoint` | Claude (auto) | Schema validation, `gym_id` tenant scope, permission policy check, mandatory cross-gym test, error format |
| `calendar-feature` | Claude (auto) | Calendar data model, move/copy/superset rules, keyboard alternatives, undo |
| `db-migration` | You only (`/db-migration`) | Migration steps, backfill, rollback plan, review checklist |
| `release` | You only (`/release`) | Version bump, changelog, tag; CI publishes |

## 5. How to create one

1. Create the folder `.claude/skills/new-api-endpoint/`.
2. Add `SKILL.md`:

   ```markdown
   ---
   name: new-api-endpoint
   description: Checklist for adding or changing a FitSphere Coach API endpoint.
     Use whenever creating or modifying an API route.
   ---
   1. Validate the request body with the shared Zod schema in src/api/schemas/.
   2. Scope every query by the caller's gym_id; never accept gym_id from the client.
   3. Check permissions through src/auth/policy.ts (role x action x resource).
   4. Add an integration test proving a coach from another gym gets 403/404.
   5. Return errors through src/lib/errors.ts.
   ```

3. Commit it so the whole team gets it; for side-effect workflows add `disable-model-invocation: true` to the frontmatter.
4. Write only what Claude can't find by reading the repo; keep `SKILL.md` short and move long reference material into separate files in the folder.

## 6. Recommended MCP servers and hooks *(Recommendation)*

- MCP: Context7 for current library docs; GitHub CLI (`gh`) for PRs and reviews.
- Hooks: Stop → typecheck + lint + unit tests; PostToolUse on Edit/Write → Prettier.
- Deny rules: `Read(./.env)`, `Read(./.env.local)`, `Read(./secrets/**)`.

## 7. Generic best practices for writing skills

- **Write a precise description:** it is what Claude reads in every session to decide when to use the skill; say what it does and when to use it.
- **Only what Claude can't find:** don't restate what the repo already shows; capture team decisions, procedures and gotchas.
- **Progressive disclosure:** keep `SKILL.md` short and put long reference material, templates and examples in separate files in the skill folder, linked from `SKILL.md`.
- **Scripts for deterministic steps:** if a step must be exact (generate a migration name, validate a file), ship a script and tell Claude to run it.
- **Concrete examples** of good output beat abstract rules.
- **One skill, one job;** split skills that cover unrelated tasks.
- **Side effects need you:** add `disable-model-invocation: true` to anything that deploys, releases, migrates or sends.
- **Test and iterate:** run the skill on real tasks, see where Claude goes wrong, refine the instructions.
- **Version in git** and review changes like code.
- **Security:** skills can contain instructions and executable scripts; install third-party skills only from trusted sources and read them before use.

## 8. Generic Claude Code workflow best practices (Anthropic docs)

**Context**
- Keep `CLAUDE.md` short (Anthropic suggests under ~200 lines): commands, misleading code, decisions not in the repo, things you've corrected twice.
- Personal preferences go in `CLAUDE.local.md` (gitignored).
- Rules that apply to one area go in `.claude/rules/` with a `paths:` filter.
- Use `/init` to start `CLAUDE.md`, then remove lines.

**Prompting**
- Be specific: name the file, the symptom and what "done" looks like.
- Hand over artefacts directly: `@file`, screenshots, piped logs.
- For unclear features, ask Claude to interview you and write a `SPEC.md`, then build from it in a fresh session.
- Say "be ambitious" when you want edge cases and states handled, not just the layout.

**Verification**
- Always give Claude a pass/fail check (tests, build, linter, screenshot).
- Ask for evidence: the command and its output.
- Use `/code-review` or a reviewer subagent for an independent look; tell it to skip style.

**Steering**
- Watch the first minute; press Esc to stop and redirect.
- `/rewind` to roll back code and conversation to an earlier point.
- After two failed corrections, `/clear` and write one better prompt.
- `/clear` between unrelated tasks; name sessions with `/rename` and resume with `claude --resume`.

**Cost and effort**
- Raise effort (`/effort high`) for tricky bugs and schema changes; lower it for quick iteration.
- Check `/usage` to see where tokens go; use cheaper models for simple subagents.

**Parallel and unattended work**
- Use git worktrees (`claude -w <name>`) so parallel sessions don't collide.
- Before any unattended run: define done, give it a check, limit what it can touch, cap turns/spend, and have a person approve the merge.

## 9. Team adoption *(Recommendation)*

- Commit `.claude/settings.json`, `.claude/skills/`, `.claude/agents/` and `.claude/rules/` so the whole team shares one setup.
- Use managed settings for organisation-wide security policy.
- AI-written code goes through the same PR review, CI gates and branch protection as human code.
- Review skills, hooks and `CLAUDE.md` quarterly; delete what is stale (`/doctor prompt-audit`).
- Keep enforced rules (hooks, deny rules) few; most guidance belongs in `CLAUDE.md` and skills.

## References

- Anthropic — Claude Code Skills: https://code.claude.com/docs/en/skills
- Anthropic — Best practices for Claude Code: https://code.claude.com/docs/en/best-practices
- Anthropic — Claude Code memory (CLAUDE.md, rules): https://code.claude.com/docs/en/memory
- Anthropic — Claude Code subagents: https://code.claude.com/docs/en/sub-agents
- Anthropic — Equipping agents for the real world with Agent Skills: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
- Agent Skills specification: https://agentskills.io/specification
- Anthropic — Claude Code hooks guide: https://code.claude.com/docs/en/hooks-guide
- Anthropic — Claude Code permissions: https://code.claude.com/docs/en/permissions
