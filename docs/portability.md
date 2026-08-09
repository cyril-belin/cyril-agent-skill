# Portability guide

The skills in this collection are written in the open [Agent Skills format](https://agentskills.io/specification): a `SKILL.md` file with YAML frontmatter and Markdown instructions.

The goal is for each skill to be as agent-agnostic as possible, with agent-specific integration layered separately under `integrations/<agent>/`.

## What is in the portable core

A portable skill should only contain:

- `name` and `description` in the frontmatter.
- Domain-specific instructions (what to detect, what to validate, how to diagnose).
- Relative file references (e.g., `package.json`, `nuxt.config.ts`, `.devin/.loops-nuxt-active`).
- Generic tool names where possible (read, write, exec, grep, search, MCP).

## What belongs in agent integrations

- Lifecycle hooks (e.g., Devin's `Stop` hook).
- Tool naming if an agent uses non-standard tool identifiers.
- Skill invocation syntax (e.g., `/loops-nuxt` vs. other slash commands).
- Marker-file directory conventions.
- Authorization or environment setup that one agent requires.

## loops-nuxt portability notes

### Core assumptions

- The project is Nuxt/Vue: it has `nuxt.config.*` and relevant dependencies.
- Validation commands come from `package.json` scripts and the package manager.
- The agent can read files, run shell commands, search the codebase, and invoke other skills.
- A Nuxt MCP is available for framework-level questions.
- The stop-hook checks for a marker file. The default marker path is `.devin/.loops-nuxt-active`.

### Devin

Devin supports the `Stop` lifecycle hook. The integration is in `integrations/devin/loops-nuxt/`. No unsupported APIs are used.

### Claude Code

Claude Code does not have a `Stop` hook in the same form as Devin. To adapt `loops-nuxt`:

- Keep the marker file convention or move it to `.claude/.loops-nuxt-active`.
- Add a reminder in `CLAUDE.md` or the project instructions: "Do not stop until the marker is removed and validation passes."
- Skill invocation via slash commands is not standard in Claude Code; users can paste the skill prompt or use a custom command.

### Codex

Codex uses Agent Skills natively. Install with:

```bash
npx skills add https://github.com/<username>/cyril-agent-skill --skill loops-nuxt
```

For the stop-hook behavior, Codex does not currently expose a `Stop` hook. The 5-iteration safety limit in `SKILL.md` still prevents runaway loops. The marker file is informational unless Codex later adds a comparable lifecycle hook.

### Windsurf

Windsurf can load skills from `.codeium/<channel>/skills/` or `.devin/skills/`. Use `npx skills add` with `--agent windsurf` or copy the skill directory manually. The stop-hook is Devin-specific, so Windsurf would need its own lifecycle integration if one becomes available.

### Other agents

For any Agent Skills-compatible agent:

1. Install `skills/loops-nuxt/` into the agent's skills directory.
2. Adapt the marker path and stop mechanism to whatever the agent supports.
3. Keep the validation and MCP rules unchanged.
4. Do not weaken the 5-iteration safety limit.

## Avoiding agent lock-in

- Keep Devin-specific YAML keys (`triggers`) in the frontmatter only if the agent ignores unknown keys gracefully, or if the skill is intended primarily for Devin. Unknown keys should not break other agents.
- Use generic Markdown; do not rely on one agent's system prompt or tool list.
- Reference files by relative paths from the project root, never by machine-specific absolute paths.
- Document any unavoidable agent-specific behavior here.
