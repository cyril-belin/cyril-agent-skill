# Agent integrations

This directory contains optional, agent-specific enforcement layers around the portable `loops-*` skills in `skills/`.

A `loops-*` skill works without an integration. The integration only adds lifecycle enforcement (such as preventing premature stopping) when the agent provides a supported native mechanism.

## Structure

```text
integrations/
└── <agent>/
    └── loops-<stack>/
        ├── stop-hook.json   # Template for the agent's lifecycle hook
        └── README.md        # Agent-specific install and migration notes
```

Example:

```text
integrations/devin/loops-nuxt/
integrations/devin/loops-flutter/
```

## Supported agents

### Devin

Devin supports a `Stop` lifecycle hook. The integration checks for a marker file and blocks the agent from stopping while a loop is active.

- `integrations/devin/loops-nuxt/`
- `integrations/devin/loops-flutter/`

Each hook independently guards its own marker file, so multiple loops can coexist safely.

## Adding an integration for a new agent

1. Verify the agent's actual lifecycle/hook API. Do not invent unsupported mechanisms.
2. Create `integrations/<agent>/loops-<stack>/`.
3. Keep the portable `SKILL.md` unchanged. Only adapt marker paths, hook syntax, or tool naming to the agent's conventions.
4. Document the new marker convention and any backward-compatibility behavior in the integration README.
5. Update `docs/portability.md` if the patterns are reusable across agents.

## Universal marker convention

All `loops-*` skills use this marker convention:

```text
.agents/state/loops-<stack>.active
```

For example:

- `.agents/state/loops-nuxt.active`
- `.agents/state/loops-flutter.active`

This path is agent-neutral. An agent integration may map it to its own lifecycle mechanism, but the portable skills reference only `.agents/state/`.

Older Devin integrations used `.devin/.loops-<stack>-active`. Those paths are now considered legacy; current Devin hooks still recognize them during migration, but new code should use `.agents/state/`.

## Contract

For the full integration contract, see `docs/loop-integration-contract.md`.
