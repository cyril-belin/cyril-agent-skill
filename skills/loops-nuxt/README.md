# loops-nuxt

Autonomous red/green validation loop for Nuxt/Vue implementation tasks.

## What it does

- Detects Nuxt/Vue repositories.
- Auto-activates on concrete implementation requests (pages, components, composables, routes, middleware, plugins, modules, auth, CMS features, bug fixes, refactors).
- Orchestrates existing engineering skills (`/architect`, `/develop`, `/tdd`, `/test`, `/debug`, `/diagnosing-bugs`, `/check`, `/code-review`, etc.).
- Consults the Nuxt MCP for framework-level questions.
- Implements the change.
- Validates with lint, typecheck, relevant tests, and `nuxt build` when appropriate.
- Diagnoses failures, fixes root causes, and re-runs validation.
- Stops only when green, blocked, or the 5-iteration safety limit is reached.
- Writes and removes a loop marker file so a stop-hook can prevent premature stopping.

## Triggering

### Automatic

In a Nuxt/Vue project, the skill activates automatically for requests such as:

- "Implement the admin dashboard."
- "Add user management."
- "Fix the hydration bug on the settings page."
- "Create a server route for user profiles."
- "Refactor the auth middleware."

### Manual

```
/loops-nuxt <describe the change>
```

## Agent integration

For Devin, install the provided stop-hook so the loop cannot be interrupted until the marker file is removed. See `integrations/devin/loops-nuxt/` in this collection.

For other agents, the loop marker is `<project-root>/.agents/state/loops-nuxt.active` by default, following the universal loop-state convention. Adapt the path and hook mechanism only to whatever the agent's lifecycle conventions require.

## Safety

- Maximum 5 consecutive red iterations on the same root symptom before escalating to the user.
- Loop marker must be deleted on success.
- Stop only when validation passes, a genuine blocker exists, or a product/business decision is required.
