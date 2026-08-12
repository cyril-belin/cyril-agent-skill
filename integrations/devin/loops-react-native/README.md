# Devin integration for loops-react-native

This folder contains the Devin-specific stop-hook integration for `loops-react-native`.

## What it does

The stop-hook checks for a marker file at `.agents/state/loops-react-native.active` whenever Devin tries to stop a turn. If the marker exists, Devin is blocked from stopping and is reminded to complete validation first.

This is the only Devin-specific part of the `loops-react-native` skill. The portable core lives in `skills/loops-react-native/`.

The hook only checks the `loops-react-native` marker. It does **not** check for or interfere with the `loops-nuxt` marker (`.agents/state/loops-nuxt.active`) or the `loops-flutter` marker (`.agents/state/loops-flutter.active`), so all loops can coexist safely in the same collection and the same Devin configuration.

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
               "command": "node -e \"const fs=require('fs'); const marker='.agents/state/loops-react-native.active'; if(fs.existsSync(marker)) console.log(JSON.stringify({decision:'block',reason:'loops-react-native validation loop is active. Validate the change before stopping, or delete .agents/state/loops-react-native.active to bypass.'}));\"",
               "timeout": 5
             }
           ]
         }
       ]
     }
   }
   ```

3. If you already have other `Stop` hooks (for example, `loops-nuxt` or `loops-flutter`), append this hook to the existing `Stop` array rather than replacing it. Each hook independently guards its own marker file.

## Install per-project

Copy the `Stop` array into the project's `.devin/hooks.v1.json` file. Project hooks override or supplement global hooks depending on Devin's config resolution.

## Verify

With the hook installed:

- A normal Devin session (no `.agents/state/loops-react-native.active`) should stop normally.
- A `/loops-react-native` session should create `.agents/state/loops-react-native.active` and be blocked from stopping until the loop completes or the marker is deleted.
- Deleting `.agents/state/loops-react-native.active` immediately allows stopping.
- Active `/loops-nuxt` or `/loops-flutter` sessions should not be affected by the `loops-react-native` hook.

## Do not export

This hook template contains no personal settings, secrets, tokens, or unrelated configuration. It only references the marker file path and the `loops-react-native` loop state.
