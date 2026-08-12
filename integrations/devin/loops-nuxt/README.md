# Devin integration for loops-nuxt

This folder contains the Devin-specific stop-hook integration for `loops-nuxt`.

## What it does

The stop-hook checks for a marker file whenever Devin tries to stop a turn. If the marker exists, Devin is blocked from stopping and is reminded to complete validation first.

The current (universal) marker path is `.agents/state/loops-nuxt.active`.

For backward compatibility during migration, this hook also recognizes the legacy marker `.devin/.loops-nuxt-active`. New installations should rely on `.agents/state/loops-nuxt.active`.

This is the only Devin-specific part of the `loops-nuxt` skill. The portable core lives in `skills/loops-nuxt/`.

## Install globally (all repositories)

1. Open your Devin user config:
   - Windows: `%APPDATA%\devin\config.json`
   - Linux/macOS: `~/.config/devin/config.json`

2. Merge the `Stop` array from `stop-hook.json` under a top-level `hooks` key. For example:

   ```json
   {
     "hooks": {
       "Stop": [
         {
           "matcher": "",
           "hooks": [
             {
               "type": "command",
               "command": "node -e \"const fs=require('fs'); const newMarker='.agents/state/loops-nuxt.active'; const oldMarker='.devin/.loops-nuxt-active'; if(fs.existsSync(newMarker) || fs.existsSync(oldMarker)) console.log(JSON.stringify({decision:'block',reason:'loops-nuxt validation loop is active. Validate the change before stopping, or delete .agents/state/loops-nuxt.active to bypass.'}));\"",
               "timeout": 5
             }
           ]
         }
       ]
     }
   }
   ```

3. If you already have other `Stop` hooks (for example, `loops-flutter`), append this hook to the existing `Stop` array rather than replacing it.

## Install per-project

Copy the `Stop` array into the project's `.devin/hooks.v1.json` file. Project hooks override or supplement global hooks depending on Devin's config resolution.

## Verify

With the hook installed:

- A normal Devin session (no marker) should stop normally.
- A `/loops-nuxt` session should create `.agents/state/loops-nuxt.active` and be blocked from stopping until the loop completes or the marker is deleted.
- Deleting `.agents/state/loops-nuxt.active` immediately allows stopping.
- The legacy marker `.devin/.loops-nuxt-active` also triggers the block for backward compatibility.

## Migration from `.devin/.loops-nuxt-active`

1. Update your Devin user config to the new hook command shown above.
2. Remove any leftover `.devin/.loops-nuxt-active` files if you see them after the skill update.
3. Add `.agents/state/` to your project's `.gitignore`:

   ```gitignore
   # Agent loop state
   .agents/state/
   ```

## Do not export

This hook template contains no personal settings, secrets, tokens, or unrelated configuration. It only references the marker file path and the `loops-nuxt` loop state.
