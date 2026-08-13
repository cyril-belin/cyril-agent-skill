# Devin integration for loops-android

This folder contains the Devin-specific stop-hook integration for `loops-android`.

## What it does

The stop-hook checks for a marker file at `.agents/state/loops-android.active` whenever Devin tries to stop a turn. If the marker exists, Devin is blocked from stopping and is reminded to complete validation first.

This is the only Devin-specific part of the `loops-android` skill. The portable core lives in `skills/loops-android/`.

The hook only checks the `loops-android` marker. It does **not** check for or interfere with the `loops-nuxt` marker (`.agents/state/loops-nuxt.active`), the `loops-flutter` marker (`.agents/state/loops-flutter.active`), or the `loops-react-native` marker (`.agents/state/loops-react-native.active`), so all loops can coexist safely in the same collection and the same Devin configuration.

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
               "command": "node -e \"const fs=require('fs'); const marker='.agents/state/loops-android.active'; if(fs.existsSync(marker)) console.log(JSON.stringify({decision:'block',reason:'loops-android validation loop is active. Validate the change before stopping, or delete .agents/state/loops-android.active to bypass.'}));\"",
               "timeout": 5
             }
           ]
         }
       ]
     }
   }
   ```

3. If you already have other `Stop` hooks (for example, `loops-nuxt`, `loops-flutter`, or `loops-react-native`), append this hook to the existing `Stop` array rather than replacing it. Each hook independently guards its own marker file.

## Install per-project

Copy the `Stop` array into the project's `.devin/hooks.v1.json` file. Project hooks override or supplement global hooks depending on Devin's config resolution.

## Verify

With the hook installed:

- A normal Devin session (no `.agents/state/loops-android.active`) should stop normally.
- A `/loops-android` session should create `.agents/state/loops-android.active` and be blocked from stopping until the loop completes or the marker is deleted.
- Deleting `.agents/state/loops-android.active` immediately allows stopping.
- Active `/loops-nuxt`, `/loops-flutter`, or `/loops-react-native` sessions should not be affected by the `loops-android` hook, and vice versa.

## Do not export

This hook template contains no personal settings, secrets, tokens, or unrelated configuration. It only references the marker file path and the `loops-android` loop state.

## Note: this repository task does not install the hook for you

This integration file is provided so you (or any consumer of this repository) can opt in. Installing it means editing your own Devin user config (`%APPDATA%\devin\config.json` or `~/.config/devin/config.json`), which is a machine-level change outside this repository. This repository only ships the template; it does not modify your global Devin configuration.
