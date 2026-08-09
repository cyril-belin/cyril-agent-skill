# cyril-agent-skill

A personal collection of reusable Agent Skills for software engineering across multiple stacks and coding agents.

This repository is the single source of truth for custom skills such as `project-flow` and `loops-nuxt`, and future skills like `loops-flutter`, `loops-react-native`, `loops-ios`, `loops-android`, and other reusable engineering workflows.

## Repository structure

```
cyril-agent-skill/
├── README.md
├── LICENSE
├── .gitignore
├── docs/
│   └── portability.md          # Adapting skills for different agents
├── skills/
│   ├── project-flow/
│   │   ├── SKILL.md            # Portable skill instructions
│   │   └── README.md           # Skill-specific usage notes
│   └── loops-nuxt/
│       ├── SKILL.md            # Portable skill instructions
│       └── README.md           # Skill-specific usage notes
├── integrations/
│   └── devin/
│       └── loops-nuxt/
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

Agent-specific wiring. Not every agent needs an integration. For Devin, `loops-nuxt` needs a Stop hook to prevent the agent from stopping while the loop marker is present.

### `scripts/`

Optional cross-skill helper scripts. Currently empty.

### `docs/`

Repository-wide documentation, including the portability guide.

## Available skills

| Skill | Stack | Purpose |
|-------|-------|---------|
| [project-flow](skills/project-flow) | Any (stack-agnostic) | Orchestrates a project from discovery through feature-by-feature implementation, with persistent Markdown context and human approval gates. |
| [loops-nuxt](skills/loops-nuxt) | Nuxt / Vue | Autonomous red/green implementation and validation loop. |

## Installation

Install the whole collection or individual skills with the [open `skills` CLI](https://github.com/vercel-labs/skills):

```bash
# Install interactively (select which skills you want)
npx skills add https://github.com/<username>/cyril-agent-skill

# List available skills without installing
npx skills add https://github.com/<username>/cyril-agent-skill --list

# Install one skill
npx skills add https://github.com/<username>/cyril-agent-skill --skill loops-nuxt
npx skills add https://github.com/<username>/cyril-agent-skill --skill project-flow

# Install all skills
npx skills add https://github.com/<username>/cyril-agent-skill --all
```

Skills are discovered from the `skills/` directory using the Agent Skills convention (`skills/*/SKILL.md`).

## Agent-specific setup

### Devin

After installing `loops-nuxt`, add the Devin stop-hook so the loop cannot be interrupted prematurely:

1. Copy the `Stop` array from [`integrations/devin/loops-nuxt/stop-hook.json`](integrations/devin/loops-nuxt/stop-hook.json) into your Devin user config.
   - Windows: `%APPDATA%\devin\config.json`
   - Linux/macOS: `~/.config/devin/config.json`
2. Merge it under the `hooks` key. Do not replace existing settings.
3. See [`integrations/devin/loops-nuxt/README.md`](integrations/devin/loops-nuxt/README.md) for details.

### Other agents

See [`docs/portability.md`](docs/portability.md) for notes on adapting skills to Claude Code, Codex, Windsurf, and other Agent Skills-compatible agents.

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
