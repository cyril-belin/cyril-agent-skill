# loops-flutter

Autonomous red/green validation loop for Flutter/Dart implementation tasks.

## What it does

- Detects Flutter applications, Flutter packages/plugins, and pure Dart packages.
- Auto-activates on concrete implementation requests (screens, widgets, routes, state layers, services, models, localization, API integration, tests, refactors, bug fixes).
- Orchestrates existing engineering skills (`/architect`, `/develop`, `/tdd`, `/test`, `/debug`, `/diagnosing-bugs`, `/check`, `/code-review`, etc.).
- Consults the official Dart/Flutter MCP (`dart mcp-server`) for framework-level questions, static analysis, runtime errors, symbol resolution, pub.dev lookup, formatting/fixes, tests, hot reload/restart, and widget/device inspection when available.
- Implements the change.
- Validates with `dart format`, static analysis (`dart analyze` / `flutter analyze`), relevant unit/widget/integration tests, build when appropriate, and runtime verification when meaningful.
- Diagnoses failures, fixes root causes, and re-runs validation.
- Stops only when green, blocked, or the 5-iteration safety limit is reached.
- Writes and removes a loop marker file so a stop-hook can prevent premature stopping.

## Triggering

### Automatic

In a Flutter/Dart project, the skill activates automatically for requests such as:

- "Implement the profile screen."
- "Add a shopping cart feature."
- "Fix the overflow on the checkout screen."
- "Create a reusable button widget."
- "Set up go_router for declarative routing."
- "Add unit tests for this parser."
- "Refactor the authentication service."
- "Add localization for French."

### Manual

```
/loops-flutter <describe the change>
```

## Project detection

The loop looks for these signals, in order of confidence:

1. `pubspec.yaml` at the project root.
2. Flutter SDK dependency (`sdk: flutter` in `dependencies` or `dev_dependencies`).
3. `lib/`, `test/`, or `integration_test/` directories.
4. Dart source files under `lib/` or `bin/`.

It differentiates between Flutter applications, Flutter packages/plugins, and pure Dart packages, then chooses the right validation commands for each.

## Agent integration

For Devin, install the provided stop-hook so the loop cannot be interrupted until the marker file is removed. See `integrations/devin/loops-flutter/` in this collection.

For other agents, the loop marker is `<project-root>/.agents/state/loops-flutter.active` by default, following the universal loop-state convention. Adapt the path and hook mechanism only to whatever the agent's lifecycle conventions require.

## Safety

- Maximum 5 consecutive red iterations on the same root symptom before escalating to the user.
- Loop marker must be deleted on success.
- Stop only when validation passes, a genuine blocker exists, or a product/business decision is required.
