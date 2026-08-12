---
name: loops-react-native
description: "React Native / Expo implementation loop — for implementation work only. Use when the user asks to implement, build, create, add, modify, fix, or refactor any React Native or Expo surface (screen, component, route, hook, state layer, API client, native module, configuration, test, or library), or on explicit /loops-react-native."
license: MIT
compatibility: "React Native / Expo repositories with a package.json. Requires an agent with file, shell, search, skill-invocation, and MCP tool support. Stop-hook behavior is agent-specific."
metadata:
  author: cyril
  version: "1.0.0"
  collection: cyril-agent-skill
triggers:
  - user
  - model
---

# loops-react-native — Autonomous React Native / Expo validation loop

Run a tight red/green loop for a single React Native or Expo change. Do not stop until the change is implemented and the relevant validation is green, or you hit a genuine blocker that requires a human decision.

## Activation

This skill is the orchestration layer for implementation tasks in React Native / Expo repositories.

Auto-invoke when **all** of these are true:

1. The repository is a React Native or Expo project (see **1. Detect the project shape**).
2. The user asks for a concrete code change using verbs such as **implement, build, create, add, modify, fix, refactor, integrate, extend, wire up, or set up**.
3. The requested change targets a React Native / Expo surface: screen, component, route, navigation, hook, state-management layer, API client, utility, asset, native module, Expo configuration, EAS configuration, test, or library API.

Also invoke on explicit `/loops-react-native` regardless of the task wording.

Do not auto-invoke for:

- Conceptual questions ("Explain how Expo Router works.")
- Reading or summarizing files ("What does this file do?")
- Brainstorming or architecture-only discussion ("What do you think about this architecture?")
- Trivial copy/text changes unless validation is genuinely useful.

If in doubt, prefer invoking the skill whenever the user wants code that must be written, validated, and made production-ready in a React Native / Expo repo.

## Loop marker

Create a loop marker file at the start and remove it at the end. This lets agent stop-hooks prevent premature termination while the loop is running.

The default marker path is `<project-root>/.agents/state/loops-react-native.active`. This is the universal loop-state convention used across all `loops-*` skills. An agent-specific integration may enforce this state through its native lifecycle mechanism (for example, Devin's Stop hook); such mechanisms are documented under `integrations/<agent>/` and are not required for the skill to function.

1. At start: resolve the project root, create `.agents/state/` if it does not exist, then write the marker file containing the current ISO timestamp and the task summary.
2. At successful end: delete the marker file.
3. If blocked and stopping: leave the file in place, include the blocker in your final report, and tell the user to delete the marker manually if they want to bypass any active stop enforcement.

## 1. Detect the project shape

Before writing code, confirm this is a React Native or Expo project and determine which kind:

- Look for `package.json` at the project root.
- React Native project: `package.json` declares `react-native` in `dependencies`, and the project contains `android/`, `ios/`, `src/` or `app/` (Expo Router), `index.js`, `App.js`/`App.tsx`, or `metro.config.js`.
- Expo managed / Continuous Native Generation project: `package.json` declares `expo` in `dependencies`, plus `app.json` / `app.config.js` / `app.config.ts`, and commonly `app/` (Expo Router) or `App.js`.
- Expo development build / dev-client project: `expo-dev-client` in `dependencies`, `ios/` and `android/` directories may be present but should be regenerated through prebuild/CNG rather than hand-edited.
- Expo Router application: `package.json` declares `expo-router` and the project has an `app/` directory with file-based routes.
- Bare React Native project without Expo: `react-native` in `dependencies`, no `expo`.
- React Native library/package: `package.json` without an `app.json`, with `src/` or `lib/`, and typically `react-native` in `peerDependencies`.

Read `package.json` to note the package name, dependencies, scripts, and package manager (presence of `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, or `bun.lockb`/`bun.lock`).

Also read project instructions if they exist: `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `README.md`, `docs/plan.md`, `docs/handoff.md`, the target `docs/features/XX-feature.md`, `app.json` / `app.config.*`, and `eas.json`.

If the task is being executed through `project-flow`, respect its approved feature implementation plan. Do not bypass the human approval gate.

If the project is not React Native / Expo, stop and report that this skill is the wrong tool.

## 2. Understand the change

From the user's request and conversation context:

- Identify the affected surface (screen, component, route, hook, state layer, API client, utility, asset, native module, config, test).
- Identify files likely to change. Use `grep` / `find_file_by_name` to locate related code, existing tests, and architecture conventions.
- If the request is ambiguous, ask a focused clarifying question rather than guessing.

## 3. Plan and implement

1. Build a **tight** validation signal first if possible: a specific test, lint/typecheck command, or build output that will go red before the fix and green after.
2. Implement the change using the project's existing patterns.
3. Prefer existing skills when they fit, but invoke only the ones relevant to the task. Do not mechanically call every Expo skill.

   Official Expo / React Native skills to reach for when the task matches:
   - `/expo-router` for navigation and routing with Expo Router.
   - `/expo-data-fetching` for network requests, API calls, caching, and data-loading patterns.
   - `/expo-native-ui` for native-feeling screens and Apple HIG / platform-specific styling.
   - `/expo-ui` for cross-platform native UI with `@expo/ui`, SwiftUI / Jetpack Compose trees.
   - `/expo-project-structure` for laying out a new Expo Router project.
   - `/expo-dev-client` for development builds and TestFlight distribution.
   - `/expo-dom` for Expo DOM components and web-code-in-webview migration.
   - `/expo-examples` for canonical third-party library integrations and example-based scaffolding.
   - `/expo-module` for creating or modifying Expo native modules and views.
   - `/expo-migrate-module` for migrating an existing Swift Expo module to the v2 macro API.
   - `/expo-tailwind-setup` for Tailwind CSS v4 / NativeWind v5 setup.
   - `/expo-web-to-native` for end-to-end migration of a web app to native.
   - `/expo-upgrade` for Expo SDK upgrades.
   - `/expo-app-clip` for iOS App Clip targets.
   - `/expo-brownfield` for embedding React Native into an existing native app.
   - `/expo-skill-feedback` for submitting feedback or managing Expo skill telemetry.

   Official EAS skills to reach for when the task matches:
   - `/eas-workflows` for CI/CD workflow YAML files and EAS workflow syntax.
   - `/eas-app-stores` for store submissions, TestFlight, and app store metadata.
   - `/eas-hosting` for deploying Expo websites and API routes.
   - `/eas-simulator` for remote cloud simulators.
   - `/eas-update-insights` for EAS Update channels and usage.
   - `/eas-observe` for EAS Observe metrics and instrumentation.

   General engineering skills to reach for when the task matches:
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

When the task or failure involves Expo / React Native framework-level behavior, consult the **Expo MCP** before assuming:

- Use `mcp_list_tools` on the `expo` server to discover available tools.
- Use documentation tools (`read_documentation`, `search_documentation`, `learn`), dependency guidance (`add_library`), and EAS/project tools as appropriate.

Do not call the MCP for trivial CSS, copy changes, static assets, or unrelated business logic.

## 4. Validate the change

Run the relevant project commands in this order, stopping at the first red signal and jumping to **5. Diagnose and fix**:

1. **Format** — the project's format script if defined (commonly `npm run format` / `yarn format` / `pnpm format` / `bun format`), or `prettier --check .` / `eslint --fix` as appropriate. Run this first because it is fast and deterministic.
2. **Lint** — the script in `package.json` (commonly `npm run lint`, `pnpm lint`, `yarn lint`, `bun lint`).
3. **Typecheck** — the script in `package.json` (`typecheck`, `type-check`, `tsc --noEmit`), or run `tsc --noEmit` directly if no script exists.
4. **Relevant tests** — run tests closest to the change first, then broader suites:
   - Unit tests for changed files: `jest <path>` or the project's test runner.
   - Component tests with React Native Testing Library: `npm test <component_test_path>`.
   - Integration / E2E tests when the change touches an end-to-end flow (Maestro, Detox, Jest integration tests, etc.).
   - Full test command when the change is broad.
5. **Project health** — when available and appropriate:
   - `npx expo install --check` to verify compatible package versions.
   - `npx expo doctor` if the project uses it.
   - Metro bundler health check (`npx expo start` no-op or bundle precheck).
6. **Build** — `npx expo export` (web), EAS build preview, or the project's build script when the change touches routing, build configuration, native setup, platform code, or release behavior. For trivial layout/copy changes, build is optional but still run if cheap.
7. **Runtime verification** — for UI/runtime changes where meaningful and where the environment supports it, launch or attach to the app and use hot reload, React Native DevTools, or the Expo MCP local runtime tools to verify the target screen/flow renders and behaves correctly. Skip this when it adds no meaningful value or when no simulator/emulator/device is available.

Choose the fastest command that still covers the change. Tighten the loop by running file-scoped checks before full-suite checks.

## 5. Diagnose and fix (red loop)

If any validation command goes red:

1. Capture the **exact** error message, file path, and line number.
2. Determine whether the failure is related to the requested change or pre-existing.
3. If framework-level behavior is involved (Expo APIs, Expo Router, React Native components, layout, navigation, Metro, EAS, data fetching, state management, platform channels, build configuration, dependency compatibility):
   - Consult the **Expo MCP** before assuming.
   - Use `mcp_list_tools` on the `expo` server, then use the relevant documentation, dependency, or project tools.
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

## Bypass and portability

- To disable any active stop enforcement for a session, delete the loop marker file (`.agents/state/loops-react-native.active` by default).
- To skip the loop entirely, do not invoke `/loops-react-native`; issue instructions normally.
- To adjust the loop, edit the installed skill file in your agent's skills directory.

### Portability note

This skill is agent-neutral. It assumes only that the agent can read files, run shell commands, search the codebase, invoke other skills, and use MCP tools if configured. Agent-specific lifecycle enforcement (such as a Stop hook) belongs in `integrations/<agent>/loops-react-native/` and is optional.
