# Devin integration for loops-nuxt

This folder contains the Devin-specific stop-hook integration for `loops-nuxt`.

## What it does

The stop-hook checks for a marker file at `.devin/.loops-nuxt-active` whenever Devin tries to stop a turn. If the marker exists, Devin is blocked from stopping and is reminded to complete validation first.

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
               "command": "node -e \"const fs=require('fs'); if(fs.existsSync('.devin/.loops-nuxt-active')) console.log(JSON.stringify({decision:'block',reason:'loops-nuxt validation loop is active. Validate the change before stopping, or delete .devin/.loops-nuxt-active to bypass.'}));\"",
               "timeout": 5
             }
           ]
         }
       ]
     }
   }
   ```

3. If you already have other `Stop` hooks, append this hook to the existing `Stop` array rather than replacing it.

## Install per-project

Copy the `Stop` array into the project's `.devin/hooks.v1.json` file. Project hooks override or supplement global hooks depending on Devin's config resolution.

## Verify

With the hook installed:

- A normal Devin session (no `.devin/.loops-nuxt-active`) should stop normally.
- A `/loops-nuxt` session should create `.devin/.loops-nuxt-active` and be blocked from stopping until the loop completes or the marker is deleted.
- Deleting `.devin/.loops-nuxt-active` immediately allows stopping.

## Do not export

This hook template contains no personal settings, secrets, tokens, or unrelated configuration. It only references the marker file path and the `loops-nuxt` loop state.
