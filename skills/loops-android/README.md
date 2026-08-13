# loops-android

Autonomous red/green validation loop for native Android (Kotlin/Java) implementation tasks.

## What it does

- Detects native Android projects (app, library, multi-module, Compose, XML/View, Wear OS, Android TV, Android XR) via Gradle files, `AndroidManifest.xml`, and dependency signals.
- Auto-activates on concrete implementation requests (screens, Compose components, XML layouts, navigation, services, Gradle/AGP config, bug fixes, migrations, refactors).
- Orchestrates existing engineering skills (`/architect`, `/develop`, `/tdd`, `/test`, `/debug`, `/diagnosing-bugs`, `/check`, `/code-review`, etc.).
- Selectively invokes the official Android Agent Skills (`/navigation-3`, `/adaptive`, `/edge-to-edge`, `/camerax`, `/migrate-xml-views-to-jetpack-compose`, `/agp-9-upgrade`, `/r8-analyzer`, `/testing-setup`, `/perfetto-trace-analysis`, etc.) only when the task matches — never mechanically.
- Uses the official **Android CLI** (`android`) as the Android-specific tooling layer for documentation lookups, SDK inspection, and device/emulator operations. There is currently no official Android MCP server.
- Implements the change, respecting the project's existing UI stack (Compose or XML/View — never forces one onto the other).
- Validates with formatting/static checks, Android lint, compilation, relevant tests, Gradle build/assemble, and runtime/UI verification when meaningful and available.
- Diagnoses failures, fixes root causes, and re-runs validation.
- Degrades gracefully when JDK, Android SDK platforms, ADB, an emulator, or Android Studio are missing — it reports exactly what could not be validated instead of claiming false success.
- Stops only when green, blocked, or the 5-iteration safety limit is reached.
- Writes and removes a loop marker file so a stop-hook can prevent premature stopping.

## Triggering

### Automatic

In a native Android project, the skill activates automatically for requests such as:

- "Implement the profile screen."
- "Add a Compose component for the settings list."
- "Migrate this screen from XML to Compose."
- "Make this screen adaptive."
- "Fix this Intent security issue."
- "Add CameraX capture to this screen."
- "Upgrade this module to AGP 9."
- "Fix this R8/Proguard rule that's stripping a class."

### Manual

```
/loops-android <describe the change>
```

## Project detection

The loop looks for these signals, in order of confidence:

1. `settings.gradle` / `settings.gradle.kts` at the project root.
2. `build.gradle` / `build.gradle.kts` applying `com.android.application` or `com.android.library` (root and/or module level).
3. `gradle.properties`, `gradlew` / `gradlew.bat`.
4. A module with `src/main/AndroidManifest.xml`.
5. `libs.versions.toml` (version catalog) when present.
6. Jetpack Compose dependencies/plugin (`androidx.compose.*`, Kotlin Compose plugin) vs. XML layouts under `res/layout/` — a project can use both; the loop does not assume Compose-only.
7. Platform-specific signals: Wear OS (`androidx.wear.*`), Android TV (`androidx.tv.*`, Leanback), Android XR (`androidx.xr.*`).

It differentiates between Android app, library, and multi-module projects, then chooses validation commands accordingly.

## Official Android skills and Android CLI

`loops-android` orchestrates the official Android Agent Skills and the official Android CLI rather than redistributing them:

- Official Android skills are installed and maintained independently (see the root `README.md` "Recommended prerequisites" section). This loop only references them by name and invokes the ones relevant to the current task.
- The Android CLI (`android`) is the preferred Android-specific agent tooling layer for documentation lookups, SDK inspection, and device/emulator operations, since there is currently no official Android MCP server. The loop inspects `android --help` / the relevant subcommand's `--help` before relying on exact syntax rather than inventing subcommands.

## Agent integration

For Devin, install the provided stop-hook so the loop cannot be interrupted until the marker file is removed. See `integrations/devin/loops-android/` in this collection.

For other agents, the loop marker is `<project-root>/.agents/state/loops-android.active` by default, following the universal loop-state convention. Adapt the path and hook mechanism only to whatever the agent's lifecycle conventions require.

## Safety

- Maximum 5 consecutive red iterations on the same root symptom before escalating to the user.
- Loop marker must be deleted on success.
- Stop only when validation passes, a genuine blocker exists, or a product/business decision is required.
- Never claim a build, test, lint, or runtime/device check ran if it did not.
