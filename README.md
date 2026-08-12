# cyril-agent-skill

A personal collection of reusable Agent Skills for software engineering across multiple stacks and coding agents.

This repository is the single source of truth for custom skills such as `project-flow` and `loops-nuxt`, and stack-specific implementation loops such as `loops-flutter`, plus future skills like `loops-react-native`, `loops-ios`, `loops-android`, and other reusable engineering workflows.

The skills are designed to be **agent-neutral** at their core. Agent-specific lifecycle enforcement (for example, Devin's Stop hook) is layered separately under `integrations/<agent>/`.

## Repository structure

```
cyril-agent-skill/
├── README.md
├── LICENSE
├── .gitignore
├── docs/
│   ├── portability.md          # Adapting skills for different agents
│   └── loop-integration-contract.md  # Universal contract for loop integrations
├── templates/
│   └── AGENTS.loops.md         # Optional project-level loop policy fallback
├── skills/
│   ├── project-flow/
│   │   ├── SKILL.md            # Portable skill instructions
│   │   └── README.md           # Skill-specific usage notes
│   ├── loops-nuxt/
│   │   ├── SKILL.md            # Portable skill instructions
│   │   └── README.md           # Skill-specific usage notes
│   ├── loops-flutter/
│   │   ├── SKILL.md            # Portable skill instructions
│   │   └── README.md           # Skill-specific usage notes
│   └── loops-react-native/
│       ├── SKILL.md            # Portable skill instructions
│       └── README.md           # Skill-specific usage notes
├── integrations/
│   └── devin/
│       ├── loops-nuxt/
│       │   ├── stop-hook.json  # Devin-specific stop-hook template
│       │   └── README.md       # Devin integration instructions
│       ├── loops-flutter/
│       │   ├── stop-hook.json  # Devin-specific stop-hook template
│       │   └── README.md       # Devin integration instructions
│       └── loops-react-native/
│           ├── stop-hook.json  # Devin-specific stop-hook template
│           └── README.md       # Devin integration instructions
└── scripts/                    # Optional helper scripts
```

### `skills/`

Portable skill cores. Each skill is a directory containing at least a `SKILL.md` file in [Agent Skills format](https://agentskills.io/specification):

- `name` (required): matches the directory name.
- `description` (required): strong model-invocation pointer.
- `license` / `compatibility` / `metadata` (optional).
- Markdown body with step-by-step instructions.

To add another skill, create `skills/<skill-name>/SKILL.md`. For example:

```
skills/loops-flutter/SKILL.md
```

### `integrations/`

Agent-specific wiring. Not every agent needs an integration. For Devin, `loops-nuxt` and `loops-flutter` provide Stop hooks to prevent the agent from stopping while a loop marker is present. See `integrations/README.md` for the architecture and `docs/loop-integration-contract.md` for the universal contract.

### `scripts/`

Optional cross-skill helper scripts. Currently empty.

### `docs/`

Repository-wide documentation, including the portability guide.

## Available skills

| Skill | Stack | Purpose |
|-------|-------|---------|
| [project-flow](skills/project-flow) | Any (stack-agnostic) | Orchestrates a project from discovery through feature-by-feature implementation, with persistent Markdown context and human approval gates. |
| [loops-nuxt](skills/loops-nuxt) | Nuxt / Vue | Autonomous red/green implementation and validation loop. |
| [loops-flutter](skills/loops-flutter) | Flutter / Dart | Autonomous red/green implementation and validation loop. |
| [loops-react-native](skills/loops-react-native) | React Native / Expo | Autonomous red/green implementation and validation loop. |

## Architecture

The collection is organized in three layers:

1. **Portable skills** — `skills/` contains the agent-neutral core of each skill. These files use the standard Agent Skills format and avoid agent-specific APIs, paths, or configuration.
2. **Universal contract** — `docs/loop-integration-contract.md` defines how any agent or IDE should integrate a `loops-*` skill: marker convention, approval-gate preservation, validation loop, safety limit, marker cleanup, and reporting. `templates/AGENTS.loops.md` provides an optional project-level fallback policy for agents without native lifecycle hooks.
3. **Agent-specific adapters** — `integrations/<agent>/` adds optional enforcement layers for agents that provide a supported native mechanism (for example, Devin's Stop hook). Adapters are not required for the skill to work.

This design means the collection works on any Agent Skills-compatible agent even if no adapter exists. Devin currently has the strongest lifecycle enforcement because it supports the Stop hook; other agents can still run the full loop using the skill instructions alone.

## Compatibility levels

### Level 1 — Agent Skills compatible

The agent discovers and executes `SKILL.md`. The loop works through instructions and validation semantics. This is the baseline for any Agent Skills-compatible agent.

### Level 2 — Agent Skills + project instruction support

The agent also reads `AGENTS.md` or equivalent project instructions. You can include the fallback policy from `templates/AGENTS.loops.md` to reinforce loop semantics when the agent has no native lifecycle hook.

### Level 3 — Native lifecycle enforcement

An agent-specific integration prevents premature stopping through a supported native mechanism. Devin currently has this integration in the repository. Other agents can add adapters once their supported mechanisms are verified.

## Recommended prerequisites

`cyril-agent-skill` contains **orchestration** skills (`project-flow`, the `loops-*` implementation loops). It is designed to sit on top of existing high-quality engineering skills rather than duplicate them — `project-flow` and the `loops-*` skills invoke or benefit from these companion skills when they are installed, but do not vendor or require them directly.

### JSMastery Pro Skills

Repository: [jsmastery-pro/skills](https://github.com/jsmastery-pro/skills)

```bash
npx skills add https://github.com/jsmastery-pro/skills
```

Provides the general engineering workflow skills this collection assumes are available: `scope`, `architect`, `develop`, `check`, `test`, `debug`, `document`, `audit`, `sync`, and related workflow skills.

### Matt Pocock Skills

Repository: [mattpocock/skills](https://github.com/mattpocock/skills)

```bash
npx skills add https://github.com/mattpocock/skills
```

This is a larger collection; not every skill in it is required. The ones this workflow specifically calls out or benefits from are:

- `grill-me`
- `domain-modeling`
- `prototype`
- `research`
- `codebase-design`
- `improve-codebase-architecture`
- `to-spec`
- `implement`
- `tdd`
- `diagnosing-bugs`
- `code-review`
- `handoff`
- `writing-for-agents`

Install the ones relevant to your project rather than the whole collection if you prefer a smaller footprint.

### Notes on third-party skills

- Third-party skills remain owned and maintained by their original authors. `cyril-agent-skill` does not copy or redistribute their implementations — it only references them by name.
- Run `npx skills update` periodically to keep installed upstream skills current.
- A missing companion skill should not make the portable core unusable where a graceful fallback is possible (e.g. `project-flow` can still orchestrate without `grill-me`, asking questions directly instead). The *full* intended workflow, however, assumes these companions are installed.

### Official Flutter/Dart Agent Skills and MCP

For `loops-flutter`, install the official upstream Flutter and Dart skills plus the Dart/Flutter MCP:

- Official Flutter skills: [flutter/agent-plugins/skills](https://github.com/flutter/agent-plugins/tree/main/skills)
  - `flutter-add-integration-test`, `flutter-add-widget-preview`, `flutter-add-widget-test`, `flutter-apply-architecture-best-practices`, `flutter-build-responsive-layout`, `flutter-fix-layout-issues`, `flutter-implement-json-serialization`, `flutter-setup-declarative-routing`, `flutter-setup-localization`, `flutter-use-http-package`
- Official Dart skills: [dart-lang/skills](https://github.com/dart-lang/skills)
  - `dart-add-unit-test`, `dart-build-cli-app`, `dart-collect-coverage`, `dart-fix-runtime-errors`, `dart-generate-test-mocks`, `dart-migrate-to-checks-package`, `dart-resolve-package-conflicts`, `dart-run-static-analysis`, `dart-setup-ffi-assets`, `dart-use-ffigen`, `dart-use-pattern-matching`, `dart-use-primary-constructors`
- Dart/Flutter MCP: configured as `dart mcp-server` in your agent's MCP config.

These official upstream skills and MCP remain owned and maintained by the Flutter and Dart teams. Update them independently using `npx skills update -g` or by refreshing the MCP server alongside your Flutter/Dart SDK.

### Official Expo / React Native Agent Skills and MCP

For `loops-react-native`, install the official upstream Expo skills plus the Expo MCP:

- Official Expo / React Native skills: [expo/skills](https://github.com/expo/skills)
  - Framework: `expo-app-clip`, `expo-brownfield`, `expo-data-fetching`, `expo-dev-client`, `expo-dom`, `expo-examples`, `expo-migrate-module`, `expo-module`, `expo-native-ui`, `expo-project-structure`, `expo-router`, `expo-skill-feedback`, `expo-tailwind-setup`, `expo-ui`, `expo-upgrade`, `expo-web-to-native`
  - EAS / delivery: `eas-app-stores`, `eas-hosting`, `eas-observe`, `eas-simulator`, `eas-update-insights`, `eas-workflows`
  - Evaluation: `expo-skill-eval`
- Expo MCP: configured as the remote HTTP server `https://mcp.expo.dev/mcp` in your agent's MCP config. Local runtime capabilities (simulator screenshots, tap interactions, DevTools) additionally require the `expo-mcp` package in the project and a development server launched with `EXPO_UNSTABLE_MCP_SERVER=1 npx expo start`.

These official upstream skills and MCP remain owned and maintained by Expo. Update them independently using `npx skills update -g` and by reconnecting the MCP server as Expo evolves its tooling.

### Recommended installation order

1. Install JSMastery Pro skills.
2. Install the relevant Matt Pocock skills.
3. Install `cyril-agent-skill`.
4. Install stack-specific official skills and MCP/tooling where applicable (e.g. the Nuxt MCP for `loops-nuxt`, the official Flutter/Dart skills and `dart mcp-server` for `loops-flutter`, or the official Expo skills and Expo MCP for `loops-react-native`).
5. Configure any agent-specific integration, such as the Devin Stop hook for `loops-nuxt`, `loops-flutter`, or `loops-react-native` (see **Agent-specific setup** below).

## Installation

Install the whole collection or individual skills with the [open `skills` CLI](https://github.com/vercel-labs/skills):

```bash
# Install interactively (select which skills you want)
npx skills add https://github.com/<username>/cyril-agent-skill

# List available skills without installing
npx skills add https://github.com/<username>/cyril-agent-skill --list

# Install one skill
npx skills add https://github.com/<username>/cyril-agent-skill --skill loops-nuxt
npx skills add https://github.com/<username>/cyril-agent-skill --skill loops-flutter
npx skills add https://github.com/<username>/cyril-agent-skill --skill loops-react-native
npx skills add https://github.com/<username>/cyril-agent-skill --skill project-flow

# Install all skills
npx skills add https://github.com/<username>/cyril-agent-skill --all
```

Skills are discovered from the `skills/` directory using the Agent Skills convention (`skills/*/SKILL.md`).

## Agent-specific setup

### Devin

After installing a loop skill, add its Devin stop-hook so the loop cannot be interrupted prematurely:

- `loops-nuxt`: copy the `Stop` array from [`integrations/devin/loops-nuxt/stop-hook.json`](integrations/devin/loops-nuxt/stop-hook.json). See [`integrations/devin/loops-nuxt/README.md`](integrations/devin/loops-nuxt/README.md) for details.
- `loops-flutter`: copy the `Stop` array from [`integrations/devin/loops-flutter/stop-hook.json`](integrations/devin/loops-flutter/stop-hook.json). See [`integrations/devin/loops-flutter/README.md`](integrations/devin/loops-flutter/README.md) for details.
- `loops-react-native`: copy the `Stop` array from [`integrations/devin/loops-react-native/stop-hook.json`](integrations/devin/loops-react-native/stop-hook.json). See [`integrations/devin/loops-react-native/README.md`](integrations/devin/loops-react-native/README.md) for details.

For each hook:

1. Open your Devin user config.
   - Windows: `%APPDATA%\devin\config.json`
   - Linux/macOS: `~/.config/devin/config.json`
2. Merge the `Stop` array under the `hooks` key. Do not replace existing settings.
3. If you have multiple loop stop-hooks, append each `Stop` entry to the existing `Stop` array. Each hook independently guards its own marker file, so `loops-nuxt` and `loops-flutter` can coexist safely.
4. Add `.agents/state/` to your project's `.gitignore` so loop marker files are not committed:

   ```gitignore
   # Agent loop state
   .agents/state/
   ```

### Other agents

See [`docs/portability.md`](docs/portability.md) and [`docs/loop-integration-contract.md`](docs/loop-integration-contract.md) for notes on adapting skills to Claude Code, Codex, Windsurf, and other Agent Skills-compatible agents. If the agent reads `AGENTS.md`, you can include the fallback policy from [`templates/AGENTS.loops.md`](templates/AGENTS.loops.md) to reinforce loop semantics when the agent has no native lifecycle hook.

## Adding a new skill

1. Create `skills/<skill-name>/SKILL.md` with valid Agent Skills frontmatter.
2. Optionally add `skills/<skill-name>/README.md`, `references/`, `scripts/`, or `assets/`.
3. If the skill needs agent-specific wiring, create `integrations/<agent>/<skill-name>/`.
4. Update this README's Available skills table and any relevant docs.
5. Commit.

Example:

```bash
mkdir skills/loops-flutter
cat > skills/loops-flutter/SKILL.md << 'EOF'
---
name: loops-flutter
description: "Flutter implementation loop. Use when..."
---

# loops-flutter
...
EOF
```

## Updating the collection

Pull the latest changes and re-run `npx skills add` to install updated skills. The skills CLI supports `--copy` if you prefer copied files over symlinks.

## License

MIT. See [LICENSE](LICENSE).
