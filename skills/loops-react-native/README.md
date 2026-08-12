# loops-react-native

Autonomous red/green validation loop for React Native / Expo implementation tasks.

## What it does

- Detects React Native / Expo projects (Expo managed, Expo CNG, dev-client, Expo Router, bare React Native, libraries/packages).
- Auto-activates on concrete implementation requests (screens, components, routes, navigation, state layers, API clients, hooks, assets, native modules, Expo features, bug fixes, refactors).
- Orchestrates existing engineering skills (`/architect`, `/develop`, `/tdd`, `/test`, `/debug`, `/diagnosing-bugs`, `/check`, `/code-review`, etc.).
- Selectively invokes official Expo skills (`/expo-router`, `/expo-data-fetching`, `/expo-ui`, `/expo-native-ui`, `/expo-dev-client`, `/expo-module`, etc.) and EAS skills (`/eas-workflows`, `/eas-app-stores`, etc.) only when the task matches.
- Consults the official **Expo MCP** (`https://mcp.expo.dev/mcp`) for current Expo documentation, dependency guidance, and project/runtime tooling.
- Implements the change.
- Validates with format, lint, typecheck, relevant tests, project health checks, build/runtime verification when meaningful.
- Diagnoses failures, fixes root causes, and re-runs validation.
- Stops only when green, blocked, or the 5-iteration safety limit is reached.
- Writes and removes a loop marker file so a stop-hook can prevent premature stopping.

## Triggering

### Automatic

In a React Native / Expo project, the skill activates automatically for requests such as:

- "Implement the profile screen."
- "Add a settings route with Expo Router."
- "Fix the broken API loading state."
- "Create a reusable Button component."
- "Add SQLite with basic CRUD."
- "Refactor the authentication hook."
- "Set up Tailwind CSS for Expo."

### Manual

```
/loops-react-native <describe the change>
```

## Project detection

The loop looks for these signals, in order of confidence:

1. `package.json` at the project root.
2. `expo` and/or `react-native` in `dependencies`.
3. `expo-router` with an `app/` directory for file-based routing.
4. `app.json` / `app.config.*` for Expo configuration.
5. `android/`, `ios/`, `src/`, `app/`, `metro.config.*`, `babel.config.*`.

It differentiates between Expo managed/CNG, dev-client, bare React Native, Expo Router, and React Native libraries/packages, then chooses the right validation commands for each.

## Agent integration

For Devin, install the provided stop-hook so the loop cannot be interrupted until the marker file is removed. See `integrations/devin/loops-react-native/` in this collection.

For other agents, the loop marker is `<project-root>/.agents/state/loops-react-native.active` by default, following the universal loop-state convention. Adapt the path and hook mechanism only to whatever the agent's lifecycle conventions require.

## Safety

- Maximum 5 consecutive red iterations on the same root symptom before escalating to the user.
- Loop marker must be deleted on success.
- Stop only when validation passes, a genuine blocker exists, or a product/business decision is required.
