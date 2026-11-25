# FitSphere Coach — 4. Avoiding Deprecated APIs & Libraries with Claude Code

Part 4 of 5 in the FitSphere Coach engineering guide. Claude's training data has a cutoff, so it can suggest APIs that have since been deprecated; the fix is to give it current docs and an automatic check that fails on deprecated usage. Items marked *(Recommendation)* are architect guidance.

## 1. Why it happens

- A model's knowledge stops at its training cutoff; libraries such as React, Next.js and auth SDKs change faster than that.
- TypeScript shows deprecated APIs with a strikethrough in the editor but does not report them as errors on its own, so they slip through builds.
- An instruction in `CLAUDE.md` is followed most of the time but is not guaranteed; only hooks, deny rules and CI are enforced every time (Anthropic Claude Code docs).

## 2. Five layers of defence

1. **Give Claude current documentation — Context7 MCP.** Context7 (a third-party server by Upstash, not Anthropic) pulls version-specific documentation and code examples into the prompt.
   - Install: `claude mcp add context7 -- npx -y @upstash/context7-mcp@latest`
   - Use: add "use context7" to a prompt, or name the version ("Next.js 16 middleware").
   - Alternative without MCP: paste the official migration guide or point Claude at the library's docs URL.

2. **Pin versions and decisions in `CLAUDE.md`.** Claude can't infer team decisions from the code. Example lines *(Recommendation)*:

   ```
   Stack: React 19, Next.js <version>, App Router only — never pages/
   Check current docs via context7 before using any library API you are unsure of
   Run: pnpm lint && pnpm tsc --noEmit before finishing
   ```

3. **Fail lint on deprecated code — typescript-eslint `no-deprecated`.** Reports any use of code marked `@deprecated` (from your code, libraries, browser and Node typings). Needs typed linting. Config: `"@typescript-eslint/no-deprecated": "error"` (also enabled by `strict-type-checked`).

4. **Check dependencies — npm.** `npm outdated` lists installed packages behind the registry (Current / Wanted / Latest); `npm audit` lists known vulnerabilities. Run both in CI; review major upgrades with their changelog. Add Dependabot or Renovate for automated upgrade PRs *(Recommendation)*.

5. **Enforce with a Claude Code hook.** A Stop hook runs your script whenever Claude tries to finish and sends it back to work if the script exits with code 2. Example `.claude/settings.json`:

   ```json
   {
     "hooks": {
       "Stop": [
         { "hooks": [
           { "type": "command", "command": "pnpm tsc --noEmit 1>&2 && pnpm eslint . 1>&2 || exit 2" }
         ] }
       ]
     }
   }
   ```

## 3. Day-to-day workflow *(Recommendation)*

- Starting a feature that uses a library: "Check the current docs for `<library>` `<version>` with context7, then implement…".
- Before a dependency upgrade: run `npm outdated`, ask Claude to read the migration guide and list breaking changes before editing code.
- After Claude finishes: ask for evidence — the lint/typecheck command and its output — and run `/code-review` for an independent review.
- Monthly: dependency upgrade pass; quarterly: review `CLAUDE.md` for stale version notes (`/doctor prompt-audit` flags outdated instructions).

## 4. Generic dependency hygiene (industry convention)

- **Prefer the platform:** use built-in web and Node.js APIs (`fetch`, `URL`, `Intl`, `structuredClone`) before adding a library.
- **Keep dependencies few:** every package is code you must update and trust (OWASP 2025 A03 Supply Chain).
- **Evaluate before adding:** recent releases, active maintainers, open-issue response, TypeScript types, licence, bundle size, known vulnerabilities.
- **Lockfile always committed;** install with `npm ci` in CI for reproducible builds.
- **Read the changelog before every upgrade;** follow official migration guides and codemods.
- **Upgrade one major version at a time,** with the test suite green before and after.
- **Understand Semantic Versioning:** a MAJOR bump signals incompatible API changes, so expect breaking changes and deprecation removals there.
- **Remove unused dependencies regularly.**

## 5. Runtime and framework deprecation signals

- Node.js classifies deprecations as Documentation-only, Application, Runtime and End-of-Life, each with a DEP code in its Deprecated APIs list. Run CI with `--throw-deprecation` so a runtime deprecation becomes an error, and use `--pending-deprecation` to surface documentation-only ones early.
- React: run development builds in `<StrictMode>` and treat console deprecation warnings as failures in tests *(Recommendation)*.
- Browser: check the DevTools Console and Issues panel for deprecation warnings during QA *(Recommendation)*.
- Treat warnings as errors in CI: fail the test run on unexpected `console.warn` / `console.error` *(Recommendation)*.

## 6. Version policy *(Recommendation)*

- Use an Active or Maintenance LTS release of Node.js in production; pin it with `.nvmrc` and the `engines` field.
- Keep framework majors within one version of the latest; schedule upgrades rather than waiting for end-of-life.
- Record the exact versions in `CLAUDE.md` and update that file in the same PR as the upgrade.

## 7. Deprecating your own code *(Recommendation)*

- Mark old internal APIs with a JSDoc `@deprecated` tag that names the replacement; `no-deprecated` then flags every caller.
- Remove deprecated internal code after one release cycle.
- For the public API used by the client app, version endpoints and announce removals in the changelog.

## 8. Prompting Claude to stay current *(Recommendation)*

- State the versions in the prompt or `CLAUDE.md`; never let Claude guess.
- Ask Claude to cite the doc page it used for any library API it introduces.
- When unsure, ask: "Is this API current in `<library>` `<version>`? Check the docs before using it."
- Review AI-written code for deprecated patterns the same way as human code; lint and CI are the final gate.

## References

- Anthropic — Best practices for Claude Code (deterministic vs non-deterministic instructions, Stop hook, CLAUDE.md): https://code.claude.com/docs/en/best-practices
- Anthropic — Claude Code hooks guide: https://code.claude.com/docs/en/hooks-guide
- Anthropic — Claude Code MCP: https://code.claude.com/docs/en/mcp
- Context7 (Upstash) GitHub: https://github.com/upstash/context7
- typescript-eslint — no-deprecated: https://typescript-eslint.io/rules/no-deprecated/
- npm docs — npm outdated: https://docs.npmjs.com/cli/v11/commands/npm-outdated
- npm docs — npm audit: https://docs.npmjs.com/cli/v11/commands/npm-audit
- Node.js — Deprecated APIs: https://nodejs.org/api/deprecations.html
- Node.js — Release schedule (LTS): https://nodejs.org/en/about/previous-releases
- Semantic Versioning 2.0.0: https://semver.org/
- OWASP Top 10:2025 A03 Software Supply Chain Failures: https://owasp.org/Top10/2025/
