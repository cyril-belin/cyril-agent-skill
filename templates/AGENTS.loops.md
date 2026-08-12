# Agent loop policy (fallback)

This project uses stack-specific `loops-*` Agent Skills for implementation and validation work. This file is a concise project-level fallback for agents that do not have a native lifecycle hook that prevents premature stopping.

## What to do

1. **Respect the active loop.** If `.agents/state/loops-*.active` exists, an implementation loop is in progress. Do not treat the task as complete until the loop reaches a valid stopping condition.
2. **Continue validation/fixing.** Run the relevant validation commands for the active stack, diagnose failures, and fix root causes. Re-validate after each fix.
3. **Do not bypass project-flow approval gates.** If a feature plan has not been approved, stop and ask for approval before implementing.
4. **Follow the skill.** The stack-specific skill instructions (`loops-nuxt`, `loops-flutter`, etc.) remain authoritative for detection, implementation, validation order, and MCP usage.
5. **Report genuine blockers.** If a blocker cannot be resolved from the repository, tools, MCPs, documentation, or environment, stop and explain it clearly. Do not hide failures or weaken validation to force success.
6. **Remove the marker only when done.** Delete the marker file on every clean or safe exit: validation success, genuine blocker, safety limit reached, or unavoidable destructive/irreversible decision.

## What the marker means

A marker file at `.agents/state/loops-<stack>.active` means:

- the loop has started;
- the change is not yet validated or the loop has not yet reached a documented stopping condition;
- the agent should continue the implementation/validation sequence described in the corresponding skill.

## Marker cleanup

This state is transient. Add the recommended `.gitignore` rule:

```gitignore
# Agent loop state
.agents/state/
```
