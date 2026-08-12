# Loop Integration Contract

This document defines the contract that any coding agent or IDE integration must implement to fully support a `loops-*` skill from the `cyril-agent-skill` collection.

It is intentionally **stack-agnostic**. A `loops-*` skill targets a specific stack (for example, Nuxt/Vue or Flutter/Dart), but the integration contract itself is the same for every loop.

## Scope

This contract covers the integration layer between an agent and a `loops-*` skill. It does not replace the skill's own instructions. The skill remains the authoritative source for stack-specific detection, validation commands, MCP usage, and safety limits.

## Marker convention

A running loop exposes its state through a marker file:

```text
.agents/state/loops-<stack>.active
```

where `<stack>` is the short stack identifier from the skill name (`nuxt`, `flutter`, `react-native`, `android`, `ios`, etc.).

The marker file is transient execution state. It should normally be ignored by Git. A recommended `.gitignore` rule:

```gitignore
# Agent loop state
.agents/state/
```

The marker file content is informational. A minimal marker contains:

- the ISO timestamp when the loop started;
- a one-line task summary.

Example:

```text
started=2026-01-15T09:42:00Z
task=Implement the profile screen
```

## Integration requirements

An integration that claims full support for a `loops-*` skill must satisfy the following requirements.

### 1. Skill discovery

Make the Agent Skill discoverable by the agent using the standard [Agent Skills format](https://agentskills.io/specification). Typically this means the skill's `SKILL.md` is available in the agent's skills directory or registry.

### 2. Respect auto-activation

Do not short-circuit the skill's auto-activation rules. The skill decides when it fires based on project detection and the user's request. The integration may invoke the skill explicitly through a slash command or equivalent, but it should not silently bypass the skill's activation logic.

### 3. Read project instructions first

Before the loop begins implementation, the agent must read relevant project instructions and context, including when present:

- `AGENTS.md`
- `CLAUDE.md`
- `CONTRIBUTING.md`
- `README.md`
- `docs/plan.md`
- `docs/handoff.md`
- target `docs/features/XX-feature.md`
- stack-specific project files (for example, `package.json` or `pubspec.yaml`)

The skill itself may list additional files. The integration must not prevent the skill from reading them.

### 4. Preserve project-flow approval gates

If `project-flow` is active and an approval gate has not been passed, the integration must not allow the loop to bypass it. The skill's instructions already enforce this; the integration must not override it.

### 5. Create the loop marker on entry

When the loop begins, create the marker file at `.agents/state/loops-<stack>.active` relative to the project root. Do this before any validation or implementation work starts.

If the `.agents/state/` directory does not exist, create it.

### 6. Execute the stack-specific loop

Run the implementation and validation sequence described in the skill's `SKILL.md`. This typically includes:

- detecting the project type;
- understanding the requested change;
- planning and implementing;
- validating in a stack-specific order;
- diagnosing failures;
- fixing root causes;
- re-running validation.

The integration should provide the skill with the tools it needs: file read/write, shell execution, search, skill invocation, and MCP access where applicable.

### 7. Retry until a valid stopping condition

The loop must continue until one of the following is true:

- the change is implemented and relevant validation passes;
- a genuine external blocker cannot be resolved from the repository, tools, MCPs, documentation, or environment;
- a missing product or business decision blocks further progress;
- the next step requires destructive or irreversible human approval;
- the skill's documented safety limit is reached (for example, 5 consecutive red iterations on the same root symptom).

Do not stop merely because one validation command failed. Failure triggers the diagnose/fix/revalidate loop.

### 8. Prevent premature completion where supported

If the agent or IDE provides a lifecycle hook or other supported mechanism for blocking premature termination while work is incomplete, the integration should use it to guard the marker file state. The integration must only block stopping while the marker exists; it must not interfere with normal sessions.

If the agent has no such mechanism, the marker file is still informative and can be reinforced through project instructions (see `templates/AGENTS.loops.md`).

### 9. Remove the marker on every clean or safe exit

Delete `.agents/state/loops-<stack>.active` when:

- validation succeeds;
- a genuine blocker is reached;
- the safety limit is reached;
- the loop exits for any reason that the user would recognize as "done trying."

Leave the marker in place only if the loop was interrupted abnormally and the user should know that the work is incomplete. In that case, the final report must explain that the marker remains and how to remove it.

### 10. Do not weaken validation

Never suppress analyzer errors, weaken tests, remove lint rules, or otherwise lower the bar merely to obtain a green result. A blocker should be reported, not hidden.

### 11. Produce a concise report

On exit, produce a concise report covering:

- what changed: files touched and a one-line summary;
- what was validated: the commands or checks that passed;
- result: green or blocked;
- remaining non-blocking warnings, if any.

If blocked:

- what is blocking progress;
- what was already tried;
- the exact human decision or missing dependency required.

## Backward compatibility

Older versions of these loops used Devin-specific marker paths under `.devin/`:

- `.devin/.loops-nuxt-active`
- `.devin/.loops-flutter-active`

A migration-aware integration may continue to recognize those legacy markers for a transition period, but the portable core now uses only `.agents/state/loops-<stack>.active`. New integrations should implement the `.agents/state/` convention and document any legacy fallback they support.

## Future loops

Any future `loops-*` skill can follow this contract by:

1. Naming its marker `.agents/state/loops-<stack>.active`.
2. Documenting the same activation, validation, safety-limit, and marker-cleanup behavior in its `SKILL.md`.
3. Providing an agent-specific integration under `integrations/<agent>/loops-<stack>/` only when a supported lifecycle mechanism exists.

`project-flow` does not need to change. It remains stack-agnostic and delegates to the appropriate `loops-*` skill by name.
