---
name: project-flow
description: "Stack-agnostic project orchestration — from discovery through feature-by-feature implementation, with persistent Markdown context and human approval gates. Use when the user wants to start, plan, or resume a project ('start this project', 'plan this project', 'let's build this app'), asks what feature to build next or to implement/prepare/continue a specific feature, or says 'resume from the handoff'. Do not use for isolated conceptual questions, trivial unrelated edits, or pure explanation requests."
license: MIT
compatibility: "Any repository and any stack. Requires an agent with file, shell, search, and skill-invocation tool support. Delegates implementation to stack-specific loop skills (e.g. loops-nuxt) where one exists."
metadata:
  author: cyril
  version: "1.0.0"
  collection: cyril-agent-skill
triggers:
  - user
  - model
---

# project-flow — Project orchestration with persistent context and approval gates

The repository is the source of truth. Persist product decisions, assumptions, architecture decisions, implementation order, feature specs, feature status, handoff context, and validation state in `docs/` — never rely on chat history for anything that can live in a file.

Always read existing project documentation before asking questions or making decisions. Never repeat a question the repository already answers.

## Activation

Auto-invoke for: starting/planning a project, asking what to build next, asking to implement/prepare/continue a named feature, or resuming from the handoff.

Do not invoke for: isolated conceptual questions, trivial unrelated edits, or pure explanation requests (e.g. "explain how auth currently works").

## 0. Orient

Before anything else, check what exists:

1. `AGENTS.md` / `CLAUDE.md` at the project root.
2. `docs/plan.md`
3. `docs/handoff.md`
4. `docs/features/` (list files, note numeric order and each `## Status`)

If `docs/plan.md` is absent or clearly insufficient (no scope, no phases), this is a **new project** → go to **1. Discovery**.

If `docs/plan.md`, `docs/handoff.md`, and `docs/features/` exist, this is an **ongoing project** → go to **4. Starting a feature**.

## 1. Discovery (new or insufficiently planned project)

Reuse anything the repository already states; only ask about genuine gaps.

- Use `/grill-me` for the initial interview **only when substantial product or technical ambiguity remains** after reading the repo. Do not invoke it mechanically.
- Reach for other skills as needed: `/scope`, `/domain-modeling`, `/architect`, `/codebase-design`, `/research`, `/prototype`.
- Ask high-value questions only. Respect explicitly chosen technologies and constraints. Flag contradictions instead of silently picking a side. When the user has no answer, record an assumption in `docs/plan.md` rather than blocking.
- Use current official documentation (via `/research` or MCP docs tools) when framework/library versions matter.
- Stop interviewing as soon as there is enough information for a reliable plan — do not chase completeness for its own sake.

### Write the project docs

Create or update, in this order:

1. `docs/plan.md` — see **Plan template** below.
2. `docs/handoff.md` — see **Handoff template** below.
3. `docs/features/01-<feature>.md`, `02-<feature>.md`, ... — see **Feature file template** below. Order by logical dependency and implementation sequence, never arbitrarily.

Initial feature files describe **what** must be built, not a full implementation plan — that comes later, per feature, in **5. Feature implementation plan**.

### Stop for global approval

After writing plan, handoff, and feature files, STOP. Report the plan summary and the feature list, and explicitly request approval. Do not write implementation code before the user approves.

## Plan template (`docs/plan.md`)

```markdown
# Project Plan

## Product Goal
## Users
## Scope
### In Scope
### Out of Scope
## Confirmed Constraints
## Architecture Summary
## Core Domain
## Implementation Phases
- [ ] 01-foundation
- [ ] 02-authentication
- [ ] 03-dashboard
## Feature Dependencies
## Assumptions
## Open Risks
## Definition of Done
## Current Status
```

Keep it concise — a roadmap, not a duplicate of the feature files.

## Feature file template (`docs/features/XX-name.md`)

```markdown
# XX - Feature Name

## Status
Planned

## Objective
## User Value
## Scope
## Out of Scope
## Dependencies
## Functional Requirements
## Acceptance Criteria
## Technical Constraints
## Validation Requirements
## Open Questions
```

Status progresses: Planned → Planning → Waiting for approval → Approved → In progress → Implemented → Validated → Completed.

## Handoff template (`docs/handoff.md`)

```markdown
# Project Handoff

## Current State
## Completed Features
## Current / Next Feature
## Confirmed Decisions
## Important Architecture Context
## Known Issues
## Open Risks
## Files / Areas Worth Reading
## Next Recommended Action
```

Operational memory for a fresh session or another agent — not a transcript. Update it after meaningful milestones and completed features (use `/handoff` if useful).

## 4. Starting a feature (ongoing project)

Read, in order: project `AGENTS.md`/`CLAUDE.md`, `docs/plan.md`, `docs/handoff.md`, the target `docs/features/XX-feature.md`, then the relevant existing code.

Do not re-run a full project interview and do not invoke `/grill-me` merely because a feature is starting.

Only ask the user (or invoke `/grill-me`) when a **material** unresolved decision remains after reading the context above — e.g. a data-model change, a permission-model ambiguity, a significant architecture choice, an undefined business rule, or conflicting requirements. Resolve minor implementation details yourself using existing architecture, conventions, and reasonable defaults.

## 5. Feature implementation plan

Before touching code, add or update an `## Implementation Plan` section inside the feature file:

```markdown
## Implementation Plan

### Steps
### Expected Files / Areas
### Validation Strategy
### Risks / Decisions
```

Only list meaningful implementation risks or decisions — not routine steps.

Set `## Status` to `Waiting for approval` and STOP. Do not modify implementation code yet.

Valid approvals: "approved", "go", "ok implement", or an equivalent explicit confirmation. If the user changes the plan, update the feature file and request approval again.

## 6. After approval — implementation

Delegate to the appropriate skill(s) for the stack and task:

- A stack-specific implementation loop when one exists (e.g. `/loops-nuxt` for Nuxt/Vue, `/loops-flutter` for Flutter/Dart, `/loops-react-native` for React Native / Expo) — preferred whenever applicable, since it already owns implementation + validation.
- Otherwise compose from `/develop`, `/implement`, `/tdd`, `/test`, `/debug`, `/diagnosing-bugs`, `/check`, `/code-review`, `/research` as needed.

`project-flow` orchestrates the project lifecycle and approval gates; it does not duplicate a stack loop's low-level validation. For the contract a stack loop integration must honor (marker convention, approval gates, safety limit, cleanup, reporting), see `docs/loop-integration-contract.md`.

## 7. Feature completion

After successful implementation and validation, update the feature file:

- Advance `## Status` (see progression above).
- Add `## Implementation Record` — a concise summary of what was built.
- Add `## Validation` with checkboxes for what actually ran successfully (lint, typecheck, tests, build, code review, manual verification). Never check an item that did not really pass.

Then:

1. Mark the feature complete in `docs/plan.md`.
2. Update `docs/handoff.md`.
3. Identify the next logical feature and state it.
4. STOP — do not start implementing the next feature without approval.

## Approval gates

Two mandatory stops, never bypassed because a next step "seems obvious":

- **Global approval** — after discovery, once plan/handoff/feature files exist, before any implementation.
- **Feature approval** — after writing a feature's `## Implementation Plan`, before touching its code.

## Context efficiency

Read targeted project files instead of re-running interviews. Invoke specialist skills only when they add value, not by default on every feature:

- `/grill-me` — unresolved high-impact ambiguity only.
- `/research` — external/current technical facts.
- `/architect` — a significant, undecided architecture choice.
- `/prototype` — a concrete uncertain technical/design question.
- `/domain-modeling` — the domain model needs clarification or evolution.
- `/handoff` — refresh operational context.

## Stack-agnostic boundary

`project-flow` works identically for Nuxt, Flutter, React Native, Android, iOS, backend, or any other stack. It never assumes a specific framework. Stack-specific implementation/validation loops (`loops-nuxt`, `loops-flutter`, `loops-react-native`, `loops-android`, `loops-ios`) are separate skills it delegates to in step 6.
