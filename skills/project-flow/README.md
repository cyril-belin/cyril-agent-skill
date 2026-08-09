# project-flow

Stack-agnostic project orchestration: discovery through feature-by-feature implementation, with persistent Markdown context and mandatory human approval gates.

## What it does

- Reads `AGENTS.md`/`CLAUDE.md`, `docs/plan.md`, `docs/handoff.md`, and `docs/features/` before asking anything — never repeats a question the repo already answers.
- For a new or insufficiently planned project: runs a targeted discovery (using `/grill-me`, `/scope`, `/domain-modeling`, `/architect`, `/codebase-design`, `/research`, `/prototype` only where they add value), then writes `docs/plan.md`, `docs/handoff.md`, and ordered `docs/features/01-...md` files.
- Stops for **explicit human approval** of the global plan before any implementation.
- For an existing project: reads context, identifies the target feature, and only asks questions for **material** unresolved decisions (data model, permissions, architecture, business rules, conflicts) — not for routine details.
- Writes a per-feature `## Implementation Plan` inside the feature file and stops again for **feature approval** before touching code.
- After approval, delegates implementation/validation to the appropriate skill — a stack-specific loop (e.g. `/loops-nuxt`) when one exists, otherwise `/develop`, `/tdd`, `/test`, `/debug`, `/check`, `/code-review`.
- After a feature is validated, updates the feature file, `docs/plan.md`, and `docs/handoff.md`, then stops and names the next feature without starting it.

## Triggering

### Automatic

- "Let's build this app." / "Start this project." / "Plan this project."
- "What feature should we implement next?"
- "Implement the dashboard." / "Prepare the dashboard feature."
- "Continue the project." / "Resume from the handoff."

Does not trigger for isolated conceptual questions, trivial unrelated edits, or pure explanation requests (e.g. "explain how auth works").

### Manual

```
/project-flow <describe the project or feature>
```

## Files it owns

- `docs/plan.md` — global roadmap.
- `docs/handoff.md` — operational memory for a fresh session or agent.
- `docs/features/XX-<feature>.md` — one file per feature, numbered by implementation order.

## Approval gates

1. **Global approval** — after discovery, before any implementation.
2. **Feature approval** — after the per-feature implementation plan, before its code.

Both are mandatory and never skipped, even when the next step looks obvious.

## Stack-agnostic

Works with Nuxt, Flutter, React Native, Android, iOS, backend projects, or any other stack. Delegates the technical implementation/validation loop to a stack-specific skill (e.g. `loops-nuxt` in this collection) when one exists.
