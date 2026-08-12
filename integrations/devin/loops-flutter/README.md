# Devin integration for loops-flutter

This folder contains the Devin-specific stop-hook integration for `loops-flutter`.

## What it does

The stop-hook checks for a marker file at `.devin/.loops-flutter-active` whenever Devin tries to stop a turn. If the marker exists, Devin is blocked from stopping and is reminded to complete validation first.

This is the only Devin-specific part of the `loops-flutter` skill. The portable core lives in `skills/loops-flutter/`.

The hook only checks the `loops-flutter` marker. It does **not** check for or interfere with the `loops-nuxt` marker (`.devin/.loops-nuxt-active`), so both loops can coexist safely in the same collection and the same Devin configuration.

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
               "command": "node -e \"const fs=require('fs'); if(fs.existsSync('.devin/.loops-flutter-active')) console.log(JSON.stringify({decision:'block',reason:'loops-flutter validation loop is active. Validate the change before stopping, or delete .devin/.loops-flutter-active to bypass.'}));\"",
               "timeout": 5
             }
           ]
         }
       ]
     }
   }
   ```

3. If you already have other `Stop` hooks (for example, the `loops-nuxt` hook), append this hook to the existing `Stop` array rather than replacing it. Each hook independently guards its own marker file.

## Install per-project

Copy the `Stop` array into the project's `.devin/hooks.v1.json` file. Project hooks override or supplement global hooks depending on Devin's config resolution.

## Verify

With the hook installed:

- A normal Devin session (no `.devin/.loops-flutter-active`) should stop normally.
- A `/loops-flutter` session should create `.devin/.loops-flutter-active` and be blocked from stopping until the loop completes or the marker is deleted.
- Deleting `.devin/.loops-flutter-active` immediately allows stopping.
- An active `/loops-nuxt` session (`.devin/.loops-nuxt-active` present, `.devin/.loops-flutter-active` absent) should not be affected by the `loops-flutter` hook.

## Do not export

This hook template contains no personal settings, secrets, tokens, or unrelated configuration. It only references the marker file path and the `loops-flutter` loop state.
