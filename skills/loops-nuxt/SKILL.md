---
name: loops-nuxt
description: "Nuxt/Vue implementation loop — for implementation work only. Use when the user asks to implement, build, create, add, modify, fix, or refactor any Nuxt/Vue surface (page, component, composable, server route, layout, middleware, plugin, module, CMS feature, authentication, or service), or on explicit /loops-nuxt."
license: MIT
compatibility: "Nuxt/Vue repositories with a package manager (npm/pnpm/yarn/bun) and Node. Requires an agent with file, shell, search, skill-invocation, and MCP tool support. Stop-hook behavior is agent-specific."
metadata:
  author: cyril
  version: "1.0.0"
  collection: cyril-agent-skill
triggers:
  - user
  - model
---

# loops-nuxt — Autonomous Nuxt validation loop

Run a tight red/green loop for a single Nuxt/Vue change. Do not stop until the change is implemented and the relevant validation is green, or you hit a genuine blocker that requires a human decision.

## Activation

This skill is the orchestration layer for implementation tasks in Nuxt/Vue repositories.

Auto-invoke when **all** of these are true:

1. The repository is a Nuxt or Vue project (see **1. Detect the project shape**).
2. The user asks for a concrete code change using verbs such as **implement, build, create, add, modify, fix, refactor, integrate, extend, wire up, or set up**.
3. The requested change targets a Nuxt/Vue surface: page, component, composable, server route, layout, middleware, plugin, module, CMS feature, authentication flow, data-fetch layer, or service.

Also invoke on explicit `/loops-nuxt` regardless of the task wording.

Do not auto-invoke for:

- Conceptual questions ("Explain how Nuxt middleware works.")
- Reading or summarizing files ("What does this file do?")
- Brainstorming or architecture-only discussion ("What do you think about this architecture?")
- Trivial copy/text changes unless validation is genuinely useful.

If in doubt, prefer invoking the skill whenever the user wants code that must be written, validated, and made production-ready in a Nuxt/Vue repo.

## Loop marker

Create a loop marker file at the start and remove it at the end. This lets agent stop-hooks prevent premature termination while the loop is running.

The default marker path is `<project-root>/.devin/.loops-nuxt-active` to align with the Devin stop-hook integration. If your agent uses a different workspace directory or hook convention, the agent-specific integration should document the adjusted path.

1. At start: resolve the project root, then write the marker file containing the current ISO timestamp and the task summary.
2. At successful end: delete the marker file.
3. If blocked and stopping: leave the file in place, include the blocker in your final report, and tell the user to delete the marker manually if they want to bypass the stop-hook.

## 1. Detect the project shape

Before writing code, confirm this is a Nuxt/Vue project:

- Look for `nuxt.config.ts`, `nuxt.config.js`, or `nuxt.config.mjs`.
- Check `package.json` for `nuxt`, `vue`, `@nuxt/*`, `nitropack`, or `h3`.
- Note the package manager: `package-lock.json` → npm, `yarn.lock` → yarn, `pnpm-lock.yaml` → pnpm, `bun.lockb`/`bun.lock` → bun.
- Read `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, and `README.md` if they exist. Respect project conventions recorded there.

If the project is not Nuxt/Vue, stop and report that this skill is the wrong tool.

## 2. Understand the change

From the user's request and conversation context:

- Identify the affected surface (page, component, composable, server route, plugin, layout, middleware, store, config).
- Identify files likely to change. Use `grep`/`find_file_by_name` to locate related code.
- If the request is ambiguous, ask a focused clarifying question rather than guessing.

## 3. Plan and implement

1. Build a **tight** validation signal first if possible: a specific test, typecheck command, or build output that will go red before the fix and green after.
2. Implement the change using the project's existing patterns.
3. Prefer existing skills when they fit:
   - `/architect` for any load-bearing design choice that is not already decided.
   - `/develop` for the build step when a spec exists.
   - `/prototype` for a throwaway proof-of-concept.
   - `/research` for framework or API facts.
   - `/tdd` for test-first work.
   - `/test` for writing tests after implementation.
   - `/check verify` for final acceptance verification.
   - `/debug` for a clear, isolated bug.
   - `/diagnosing-bugs` for a hard or non-deterministic failure.
   - `/code-review` for a final review before declaring done.

Do not introduce unrelated refactors. Keep the change scoped.

When the task or failure involves Nuxt/Vue framework-level behavior, consult the **Nuxt MCP** before assuming:

- Use `mcp_list_tools` on the `nuxt` server.
- Use `mcp_call_tool` with `list-documentation-pages` or `get-documentation-page` for the relevant topic.

Do not call the MCP for trivial CSS, copy changes, static HTML, or unrelated business logic.

## 4. Validate the change

Run the relevant project commands in this order, stopping at the first red signal and jumping to **5. Diagnose and fix**:

1. **Lint** — the script in `package.json` (commonly `npm run lint`, `pnpm lint`, `yarn lint`).
2. **Typecheck** — the script in `package.json` (`typecheck`, `type-check`, `nuxt typecheck`, or `vue-tsc --noEmit` via package manager).
3. **Relevant tests** — unit tests for changed files first (`vitest run <path>`), then the affected test suite, then the full test command if the change is broad.
4. **Build** — `nuxt build` (via package manager script) when the change touches routing, layouts, middleware, composables, SSR, Nitro, server routes, runtimeConfig, rendering modes, or build-time configuration. For trivial CSS/copy/static HTML changes, build is optional but still run if cheap.

Choose the fastest command that still covers the change. Tighten the loop by running file-scoped checks before full-suite checks.

## 5. Diagnose and fix (red loop)

If any validation command goes red:

1. Capture the **exact** error message, file path, and line number.
2. Determine whether the failure is related to the requested change or pre-existing.
3. If framework-level behavior is involved (Nuxt APIs, Vue APIs, routing, layouts, middleware, composables, auto-imports, data fetching, SSR, hydration, Nitro, server routes, modules, plugins, runtimeConfig, rendering modes, build behavior, deployment behavior):
   - Consult the **Nuxt MCP** before assuming.
   - Use `mcp_list_tools` on the `nuxt` server, then `mcp_call_tool` with `list-documentation-pages` or `get-documentation-page` for the relevant topic.
4. Form a falsifiable hypothesis and a minimal fix.
5. Apply the fix. Prefer fixing the root cause over suppressing errors.
6. Re-run the failing validation command (and any cheaper checks it depends on) before running the full suite again.
7. Do not weaken lint rules, TypeScript checks, or tests merely to make the loop pass.

## 6. Iterate safely

- Maintain a `todo_write` list so the loop is visible and scoped.
- Keep a failure log mentally: if the same error repeats after a fix, your hypothesis was wrong. Re-diagnose.
- Limit: after **5 consecutive red iterations on the same root symptom**, escalate to the user with what was tried and what is blocking.
- Stop looping if continuing would require a product/business decision, an external dependency, or a destructive/irreversible action.

## 7. Stop and report

Only stop when one of these is true:

1. The change is implemented and all relevant validation passes.
2. A genuine external blocker exists that cannot be resolved from the repository, tools, MCPs, documentation, or environment.
3. A missing product/business decision blocks further progress.
4. The next step requires destructive or irreversible human approval.

Final report (concise):

- What changed: files touched and a one-line summary.
- What was validated: the commands/checks that passed.
- Result: green / blocked.
- Remaining non-blocking warnings, if any.

If blocked:

- What is blocking progress.
- What was already tried.
- The exact human decision or missing dependency required.

## Bypass instructions

- To disable the stop-hook for a session, delete the loop marker file (`.devin/.loops-nuxt-active` by default).
- To skip the loop entirely, do not invoke `/loops-nuxt`; issue instructions normally.
- To adjust the loop, edit the installed skill file in your agent's skills directory.
