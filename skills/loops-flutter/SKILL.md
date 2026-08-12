---
name: loops-flutter
description: "Flutter/Dart implementation loop — for implementation work only. Use when the user asks to implement, build, create, add, modify, fix, or refactor any Flutter/Dart surface (screen, widget, route, state layer, service, model, localization, test, or package), or on explicit /loops-flutter."
license: MIT
compatibility: "Flutter/Dart repositories with a pubspec.yaml. Requires an agent with file, shell, search, skill-invocation, and MCP tool support. Stop-hook behavior is agent-specific."
metadata:
  author: cyril
  version: "1.0.0"
  collection: cyril-agent-skill
triggers:
  - user
  - model
---

# loops-flutter — Autonomous Flutter/Dart validation loop

Run a tight red/green loop for a single Flutter/Dart change. Do not stop until the change is implemented and the relevant validation is green, or you hit a genuine blocker that requires a human decision.

## Activation

This skill is the orchestration layer for implementation tasks in Flutter/Dart repositories.

Auto-invoke when **all** of these are true:

1. The repository is a Flutter or Dart project (see **1. Detect the project shape**).
2. The user asks for a concrete code change using verbs such as **implement, build, create, add, modify, fix, refactor, integrate, extend, wire up, or set up**.
3. The requested change targets a Flutter/Dart surface: screen, widget, route, layout, animation, state-management layer, service, model, JSON mapping, localization, test, CLI entrypoint, or package API.

Also invoke on explicit `/loops-flutter` regardless of the task wording.

Do not auto-invoke for:

- Conceptual questions ("Explain how Riverpod works.")
- Reading or summarizing files ("What does this file do?")
- Brainstorming or architecture-only discussion ("What do you think about this architecture?")
- Trivial copy/text changes unless validation is genuinely useful.

If in doubt, prefer invoking the skill whenever the user wants code that must be written, validated, and made production-ready in a Flutter/Dart repo.

## Loop marker

Create a loop marker file at the start and remove it at the end. This lets agent stop-hooks prevent premature termination while the loop is running.

The default marker path is `<project-root>/.agents/state/loops-flutter.active`. This is the universal loop-state convention used across all `loops-*` skills. An agent-specific integration may enforce this state through its native lifecycle mechanism (for example, Devin's Stop hook); such mechanisms are documented under `integrations/<agent>/` and are not required for the skill to function.

1. At start: resolve the project root, create `.agents/state/` if it does not exist, then write the marker file containing the current ISO timestamp and the task summary.
2. At successful end: delete the marker file.
3. If blocked and stopping: leave the file in place, include the blocker in your final report, and tell the user to delete the marker manually if they want to bypass any active stop enforcement.

## 1. Detect the project shape

Before writing code, confirm this is a Flutter or Dart project and determine which kind:

- Look for `pubspec.yaml` at the project root.
- Flutter application: `pubspec.yaml` declares `flutter` in `dependencies` plus `sdk: flutter`, and the project contains a `lib/` directory (often with `main.dart`).
- Flutter package/plugin: `pubspec.yaml` declares `flutter` in `dependencies` and/or `dev_dependencies`, with `sdk: flutter`, plus a `lib/` directory. A plugin also has platform directories (`android/`, `ios/`, `linux/`, `macos/`, `windows/`, `web/`).
- Pure Dart package/application: `pubspec.yaml` without a Flutter SDK dependency, with `lib/` and/or `bin/`, and Dart source files.
- Read `pubspec.yaml` to note the package name, SDK constraints, dependencies, and any `flutter:` or `flutter_lints:` configuration.

Also read project instructions if they exist: `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `README.md`, `docs/plan.md`, `docs/handoff.md`, and the target `docs/features/XX-feature.md`.

If the task is being executed through `project-flow`, respect its approved feature implementation plan. Do not bypass the human approval gate.

If the project is not Flutter/Dart, stop and report that this skill is the wrong tool.

## 2. Understand the change

From the user's request and conversation context:

- Identify the affected surface (screen, widget, route, layout, state layer, service, model, test, CLI, package API).
- Identify files likely to change. Use `grep` / `find_file_by_name` to locate related code, existing tests, and architecture conventions.
- If the request is ambiguous, ask a focused clarifying question rather than guessing.

## 3. Plan and implement

1. Build a **tight** validation signal first if possible: a specific test, analyzer command, or build output that will go red before the fix and green after.
2. Implement the change using the project's existing patterns.
3. Prefer existing skills when they fit, but invoke only the ones relevant to the task. Do not mechanically call every Flutter/Dart skill.

   Official Flutter skills to reach for when the task matches:
   - `/flutter-add-integration-test` for integration testing or converting MCP-driven UI exploration into permanent tests.
   - `/flutter-add-widget-preview` when adding or updating a widget and an interactive preview would help.
   - `/flutter-add-widget-test` for component-level widget tests.
   - `/flutter-apply-architecture-best-practices` when structuring a new app or refactoring for scalability.
   - `/flutter-build-responsive-layout` for cross-form-factor layouts.
   - `/flutter-fix-layout-issues` for overflow or unbounded-constraint errors.
   - `/flutter-implement-json-serialization` for manual `fromJson` / `toJson` models.
   - `/flutter-setup-declarative-routing` for `go_router` or similar URL-based routing.
   - `/flutter-setup-localization` for adding `flutter_localizations` / `intl` / `l10n.yaml`.
   - `/flutter-use-http-package` for REST API calls with `package:http`.

   Official Dart skills to reach for when the task matches:
   - `/dart-add-unit-test` for unit tests on functions, methods, and classes.
   - `/dart-build-cli-app` for command-line utilities and scripts.
   - `/dart-collect-coverage` for coverage reports.
   - `/dart-fix-runtime-errors` for runtime stack traces and hot-reload verification.
   - `/dart-generate-test-mocks` for `mockito` / `build_runner` mocks.
   - `/dart-migrate-to-checks-package` for migrating expectations to `package:checks`.
   - `/dart-resolve-package-conflicts` when `pub get` fails on version conflicts.
   - `/dart-run-static-analysis` for `dart analyze` / `flutter analyze` and `dart fix --apply`.
   - `/dart-setup-ffi-assets` for Native Assets / C interop.
   - `/dart-use-ffigen` for generating FFI bindings.
   - `/dart-use-pattern-matching` for switch expressions and pattern matching.
   - `/dart-use-primary-constructors` for modern constructor syntax.

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

When the task or failure involves Flutter/Dart framework-level behavior, consult the **Dart/Flutter MCP** (`dart mcp-server`) before assuming:

- Use `mcp_list_tools` on the `dart` server to discover available tools.
- Use analysis tools (`analyze_files`), LSP tools (`lsp.*`), pub.dev search (`pub_dev_search`), package/dependency tools (`pub`), formatting/fixes, test tools, runtime interaction, hot reload/restart, and widget/device inspection as appropriate.

Do not call the MCP for trivial copy changes, static assets, or unrelated business logic.

## 4. Validate the change

Run the relevant project commands in this order, stopping at the first red signal and jumping to **5. Diagnose and fix**:

1. **Format** — `dart format --set-exit-if-changed .` (or `flutter format` on older Flutter versions). Prefer `dart format`. Run this first because it is fast and deterministic.
2. **Static analysis** — `dart analyze` for pure Dart packages and CLI apps; `flutter analyze` for Flutter apps/packages. Use the narrowest scope first if the project supports it (e.g., `dart analyze <path>`).
3. **Relevant tests** — run tests closest to the change first, then broader suites:
   - Unit tests for changed files: `dart test <path>` or `flutter test <path>`.
   - Widget tests for changed UI: `flutter test <widget_test_path>`.
   - Integration tests when the change touches an end-to-end flow: `flutter test integration_test/<path>` (or the project's integration-test script).
   - Full test command when the change is broad.
4. **Build** — `flutter build apk`, `flutter build ios --no-codesign`, `flutter build web`, `dart compile exe <entrypoint>`, or the project's build script when the change touches routing, build configuration, native setup, platform channels, or release behavior. For trivial layout/copy changes, build is optional but still run if cheap.
5. **Runtime verification** — for UI/runtime changes where meaningful, launch or attach to the app and use hot reload/restart, widget inspection, or the Dart/Flutter MCP runtime tools to verify the target screen/flow renders and behaves correctly. Skip this when it adds no meaningful value.

Choose the fastest command that still covers the change. Tighten the loop by running file-scoped checks before full-suite checks.

## 5. Diagnose and fix (red loop)

If any validation command goes red:

1. Capture the **exact** error message, file path, and line number.
2. Determine whether the failure is related to the requested change or pre-existing.
3. If framework-level behavior is involved (Flutter widgets, layout, routing, state management, async/gap/Isolate behavior, platform channels, build configuration, pub dependencies):
   - Consult the **Dart/Flutter MCP** before assuming.
   - Use `mcp_list_tools` on the `dart` server, then use the relevant analysis, LSP, or pub tools.
4. Form a falsifiable hypothesis and a minimal fix.
5. Apply the fix. Prefer fixing the root cause over suppressing errors.
6. Re-run the failing validation command (and any cheaper checks it depends on) before running the full suite again.
7. Do not weaken lint rules, analyzer checks, or tests merely to make the loop pass.

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

- To disable any active stop enforcement for a session, delete the loop marker file (`.agents/state/loops-flutter.active` by default).
- To skip the loop entirely, do not invoke `/loops-flutter`; issue instructions normally.
- To adjust the loop, edit the installed skill file in your agent's skills directory.

### Portability note

This skill is agent-neutral. It assumes only that the agent can read files, run shell commands, search the codebase, invoke other skills, and use MCP tools if configured. Agent-specific lifecycle enforcement (such as a Stop hook) belongs in `integrations/<agent>/loops-flutter/` and is optional.
