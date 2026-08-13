---
name: loops-android
description: "Android (Kotlin/Java) implementation loop — for implementation work only. Use when the user asks to implement, build, create, add, modify, fix, migrate, optimize, or refactor any native Android surface (screen, Compose component, XML layout, navigation, service, module, test, or Gradle/AGP config), or on explicit /loops-android."
license: MIT
compatibility: "Native Android (Kotlin/Java) repositories with a Gradle build (settings.gradle(.kts), build.gradle(.kts)). Requires an agent with file, shell, search, and skill-invocation tool support. Uses the official Android CLI (`android`) as the Android-specific tooling layer; no official Android MCP currently exists. Stop-hook behavior is agent-specific."
metadata:
  author: cyril
  version: "1.0.0"
  collection: cyril-agent-skill
triggers:
  - user
  - model
---

# loops-android — Autonomous Android validation loop

Run a tight red/green loop for a single native Android change. Do not stop until the change is implemented and the relevant available validation is green, or you hit a genuine blocker that requires a human decision.

## Activation

This skill is the orchestration layer for implementation tasks in native Android (Kotlin/Java) repositories.

Auto-invoke when **all** of these are true:

1. The repository is a native Android project (see **1. Detect the project shape**).
2. The user asks for a concrete code change using verbs such as **implement, build, create, add, modify, fix, migrate, optimize, or refactor**.
3. The requested change targets an Android surface: screen, Jetpack Compose component, XML/View layout, navigation, service, ViewModel/state layer, repository/data layer, native module, CameraX/Media3 integration, authentication, API integration, Gradle/AGP/version-catalog configuration, R8/Proguard rules, Wear OS/Android TV/Android XR surface, or test.

Also invoke on explicit `/loops-android` regardless of the task wording.

Do not auto-invoke for:

- Conceptual questions ("Explain Navigation 3." / "How does R8 shrinking work?")
- Reading or summarizing files ("What does this file do?")
- Brainstorming or architecture-only discussion ("What do you think about this module structure?")
- Trivial copy/text-only changes unless validation is genuinely useful.

If in doubt, prefer invoking the skill whenever the user wants Android code that must be written, validated, and made production-ready.

## Loop marker

Create a loop marker file at the start and remove it at the end. This lets agent stop-hooks prevent premature termination while the loop is running.

The default marker path is `<project-root>/.agents/state/loops-android.active`. This is the universal loop-state convention used across all `loops-*` skills (see `docs/loop-integration-contract.md`). An agent-specific integration may enforce this state through its native lifecycle mechanism (for example, Devin's Stop hook); such mechanisms are documented under `integrations/<agent>/` and are not required for the skill to function. Do not use a Devin-specific marker in this portable core.

1. At start: resolve the project root, create `.agents/state/` if it does not exist, then write the marker file containing the current ISO timestamp and the task summary.
2. At successful end: delete the marker file.
3. If blocked and stopping: leave the file in place, include the blocker in your final report, and tell the user to delete the marker manually if they want to bypass any active stop enforcement.

## 1. Detect the project shape

Before writing code, confirm this is a native Android project and determine which kind:

- Look for `settings.gradle` / `settings.gradle.kts` at the project root, plus `build.gradle` / `build.gradle.kts` in the root and in an `app/` (or other) module.
- Confirm the Android Gradle Plugin is applied: `com.android.application`, `com.android.library`, or the Kotlin equivalent plugin ids in `build.gradle(.kts)` or `libs.versions.toml`.
- Look for `gradle.properties`, `gradlew` / `gradlew.bat`, and a module with `src/main/AndroidManifest.xml`.
- Determine the module type per module: **app** (`com.android.application`), **library** (`com.android.library`), or part of a **multi-module** project (multiple Gradle modules under `settings.gradle(.kts)`).
- Determine the UI stack: **Jetpack Compose** (`androidx.compose.*` / `org.jetbrains.kotlin.plugin.compose` in dependencies) vs. **XML/View-based** (layout XML under `res/layout/`, `Activity`/`Fragment` with `setContentView`/`ViewBinding`/`DataBinding`). A project can mix both — do not assume Compose-only.
- Note platform targets if present: Wear OS (`androidx.wear.*`, `horologist`), Android TV (`androidx.tv.*`, Leanback), Android XR (`androidx.xr.*`, Compose Glimmer).
- Note the dependency management style: version catalog (`gradle/libs.versions.toml`) vs. inline versions.

Also read project instructions if they exist: `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `README.md`, `docs/plan.md`, `docs/handoff.md`, and the target `docs/features/XX-feature.md`. Then read `settings.gradle(.kts)`, the relevant module's `build.gradle(.kts)`, `gradle.properties`, `libs.versions.toml`, `AndroidManifest.xml`, and existing architecture/source conventions and relevant tests for the affected area.

If the task is being executed through `project-flow`, respect its approved feature implementation plan. Do not bypass the human approval gate.

If the project is not native Android, stop and report that this skill is the wrong tool.

## 2. Understand the change

From the user's request and conversation context:

- Identify the affected surface (screen, Compose component, XML layout, navigation graph, ViewModel/state, repository/service, module, Gradle/AGP config, test).
- Identify files likely to change. Use `grep` / `find_file_by_name` to locate related code, existing tests, and architecture conventions.
- If the request is ambiguous, ask a focused clarifying question rather than guessing.

## 3. Plan and implement

1. Build a **tight** validation signal first if possible: a specific test, lint/compile output, or build task that will go red before the fix and green after.
2. Implement the change using the project's existing patterns. Do not force Compose into an XML/View project, and do not force XML into a Compose project — respect the existing UI stack unless migration is explicitly requested (see **12. XML/View projects**).
3. Prefer existing skills when they fit, but invoke only the ones relevant to the task. Do not mechanically invoke every official Android skill.

   Official Android skills to reach for when the task matches:
   - `/android-cli` for Android CLI usage itself (project creation, SDK inspection, run/install, layout/screen inspection, documentation lookup).
   - `/navigation-3` for Jetpack Navigation 3 setup, deep links, backstacks, scenes.
   - `/adaptive` for multi-window-size, multi-input, multi-pane adaptive Compose UI.
   - `/edge-to-edge` for edge-to-edge migration and system-bar/inset issues.
   - `/styles` for the Jetpack Compose Styles API and component theming.
   - `/camerax` for CameraX capture/recording and ML Kit/Media3 effects integration.
   - `/media3-cast-integration` for Google Cast via Jetpack Media3.
   - `/appfunctions` for exposing app workflows to AppFunctions/system agents.
   - `/verified-email` for Credential Manager verified-email sign-up flows.
   - `/android-intent-security` for Intent redirection and component-exposure audits.
   - `/migrate-xml-views-to-jetpack-compose` when an XML → Compose migration is explicitly requested or part of the approved plan.
   - `/agp-9-upgrade` when the task actually involves an AGP 9 migration.
   - `/r8-analyzer` when R8/Proguard keep-rule behavior is relevant.
   - `/testing-setup` when testing architecture/configuration itself is part of the task.
   - `/perfetto-trace-analysis` / `/perfetto-sql` for latency, jank, startup, or memory investigations from a trace.
   - `/play-billing-library-version-upgrade`, `/play-policy-insights`, `/engage-sdk-integration` for the corresponding Play-specific tasks.
   - `/leanback-to-compose-tv-migration` for Android TV Leanback → Compose for TV migrations.
   - `/wear-compose-m3` for Wear OS Compose Material3 work.
   - `/display-glasses-with-jetpack-compose-glimmer` for Android XR display-glasses UI.

   General engineering skills to reach for when the task matches:
   - `/architect` for any load-bearing design choice that is not already decided.
   - `/develop` for the build step when a spec exists.
   - `/prototype` for a throwaway proof-of-concept.
   - `/research` for framework or API facts.
   - `/tdd` for test-first work.
   - `/test` for writing tests after implementation.
   - `/check verify` for final acceptance verification.
   - `/debug` for a clear, isolated bug.
   - `/diagnosing-bugs` for a hard or non-deterministic failure.
   - `/code-review` for a final review before declaring done.

Do not introduce unrelated refactors. Keep the change scoped. Do not duplicate specialist skill content inside this loop — invoke the skill instead of reimplementing its instructions here.

There is currently no official Android MCP server. When the task or a failure involves version-sensitive Android/Gradle/AGP/Kotlin tooling behavior, use the official **Android CLI** (`android`) as the Android-specific tooling layer instead of relying on model memory:

- Run `android --help` or `android <command> --help` before relying on exact syntax you are not certain of. Do not invent Android CLI subcommands.
- Use `android docs` for official Android documentation lookups.
- Use `android info` to inspect the SDK location, connected devices, and environment configuration.
- Use `android sdk list` / `android sdk install` to inspect or (only when explicitly requested) install SDK packages.
- Use `android run`, `android install`, `android layout`, and `android screen` for device/emulator operations, layout inspection, and screenshots when a device or emulator is available.
- Use `android studio check` / the `android studio *` commands only when Android Studio integration is relevant and running.

Do not substitute an unofficial Android MCP server for any of the above.

## 4. Validate the change

Do not assume one fixed command set for every Android repo — inspect the actual project first (Gradle tasks, module structure, test configuration, CI scripts, project-specific validation commands) and adapt.

Prefer the project's Gradle wrapper over a globally installed Gradle: `./gradlew` on Linux/macOS, `gradlew.bat` (or `./gradlew.bat`) on Windows.

Run the relevant project commands in this order when available, stopping at the first red signal and jumping to **5. Diagnose and fix**:

1. **Formatting / static checks** — the project's formatter/static-analysis task if configured (e.g. `ktlint`, `detekt`, `spotless` Gradle tasks). Run this first because it is fast and deterministic.
2. **Lint** — `./gradlew lint` (or the module-scoped variant, e.g. `./gradlew :app:lintDebug`) for Android-specific lint issues.
3. **Compilation** — the narrowest compile task available (e.g. `./gradlew :app:compileDebugKotlin` or `assembleDebug` for a fast smoke check) before running broader tasks.
4. **Relevant tests** — closest to the change first, then broader:
   - JVM unit tests: `./gradlew :module:testDebugUnitTest` (or `test`), scoped to the changed class/package first with `--tests`.
   - Compose UI tests / instrumented tests: `./gradlew :module:connectedDebugAndroidTest` or `./gradlew :module:testDebugUnitTest` for Robolectric-based tests, when a device/emulator or Robolectric is available.
   - Screenshot tests when the project already uses them.
   - Full test task when the change is broad.
5. **Build** — `./gradlew :app:assembleDebug` (or the project's equivalent) when the change touches Gradle configuration, dependencies, AGP, manifest, resources, or release behavior. For a trivial, purely logical change already covered by tests, a full assemble is optional but still run if cheap.
6. **Runtime/UI verification** — for Compose or XML/View UI changes where meaningful and where a device/emulator is available (see **11. Jetpack Compose / UI validation**).

Choose the fastest command that still covers the change. Tighten the loop by running module-/class-scoped checks before full-suite checks.

## 5. Diagnose and fix (red loop)

If any validation command goes red:

1. Capture the **exact** error message, file path, and line number.
2. Determine whether the failure is related to the requested change or pre-existing.
3. If framework/tooling-level behavior is involved (Gradle, AGP, Kotlin/Android compiler, Jetpack libraries, Compose runtime, R8/Proguard, Perfetto traces):
   - Consult the relevant official Android skill (see **3. Plan and implement**) and/or run `android docs` / `android <command> --help` before assuming.
4. Form a falsifiable hypothesis and a minimal fix.
5. Apply the fix. Prefer fixing the root cause over suppressing errors.
6. Re-run the failing validation command (and any cheaper checks it depends on) before running the full suite again.
7. Never suppress compiler errors, disable lint, weaken tests, remove failing assertions, or edit Gradle configuration solely to hide failures.

## 6. Iterate safely

- Maintain a `todo_write` list so the loop is visible and scoped.
- Keep a failure log mentally: if the same error repeats after a fix, your hypothesis was wrong. Re-diagnose.
- Limit: after **5 consecutive red iterations on the same root symptom** (implementation → validation → diagnosis/fix cycles, not individual tool calls), escalate to the user with what was tried and what is blocking.
- Stop looping if continuing would require a product/business decision, an external dependency, or a destructive/irreversible action.

## 7. Tool availability must degrade gracefully

The machine running this loop may not have a JDK, Android SDK platforms/build-tools, ADB, an emulator, Android Studio, or a physical device.

Before validation, detect what is actually available (for example: `android info`, `where adb` / `which adb`, `./gradlew --version`, checking `ANDROID_HOME`/`ANDROID_SDK_ROOT`). Then:

- Perform all validation that is still possible with what is available.
- Clearly report what could not be run and why.
- Identify the exact missing dependency (e.g. "no JDK on PATH", "no `platforms;android-34` installed", "no emulator/device connected").
- Never falsely claim that a build, test, or runtime check passed if it was not actually run.

Do not automatically install a JDK, large SDK platform/build-tools images, an emulator system image, or Android Studio unless the user explicitly requests it. Report the gap instead and let the user decide.

## 8. Implementation flow

1. Detect Android project type and affected area (**1**).
2. Read project instructions and any approved `project-flow` context (**1**, **4** below).
3. Understand the requested change (**2**).
4. Select only the relevant general and official Android skills (**3**).
5. Consult the Android CLI / current official documentation where relevant (**3**).
6. Implement the change (**3**).
7. Validate narrowly first, then expand validation before declaring completion (**4**).
8. If validation fails: diagnose the root cause, use Gradle/ADB/Perfetto/Android-CLI tooling where useful, fix, and re-run the relevant validation (**5**).
9. Repeat until green, a genuine blocker, or the safety limit is reached (**6**).
10. Remove the active marker.
11. Produce a concise completion/blocker report (**7. Stop and report**).

## 9. Read project context first

Before implementation, read relevant context when present: `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `README.md`, `docs/plan.md`, `docs/handoff.md`, the target `docs/features/XX-feature.md`, `settings.gradle(.kts)`, the affected module's `build.gradle(.kts)`, `gradle.properties`, `libs.versions.toml`, `AndroidManifest.xml`, existing architecture/source conventions, and relevant tests.

If execution comes from `project-flow`, respect the approved feature implementation plan. Never bypass its human approval gates.

## 10. Baseline Android validation

For a standard Android app with a functioning SDK/JDK, completion should normally include relevant equivalents of: formatting/static checks, Android lint, Kotlin/Java compilation, relevant unit tests, relevant UI/instrumentation tests when applicable, and an appropriate Gradle assemble/build task.

Do not run every expensive Gradle task for every tiny change — use narrow validation during iterations and broader validation before completion.

Never suppress compiler errors, disable lint to force a green result, weaken tests, remove failing assertions, or edit Gradle configuration solely to hide failures.

## 11. Jetpack Compose / UI validation

For Compose UI changes, runtime/UI verification is strongly preferred when tooling is available.

When possible:

1. Build the relevant module/app.
2. Launch/install on an emulator or device (`android run` / `android install`, or the Gradle equivalent).
3. Navigate to the affected screen.
4. Inspect runtime errors (`android screen`, logcat/`adb logcat` if available).
5. Inspect the layout (`android layout`).
6. Interact with the relevant flow.

Pay attention to: crashes, recomposition/runtime issues, clipping/overflow, adaptive-layout problems, navigation failures, accessibility issues, and edge-to-edge/inset problems.

Use relevant official Android skills such as `/adaptive`, `/edge-to-edge`, `/testing-setup`, and `/styles` when they match the task.

If no device/emulator exists, report that runtime/UI verification could not be performed — do not claim it was.

## 12. XML/View projects

Do not force Compose into existing XML/View projects. Respect the project's current UI stack.

Only invoke `/migrate-xml-views-to-jetpack-compose` when migration is explicitly requested or clearly part of the approved `project-flow` feature plan.

## 13. Gradle / AGP validation

When changes affect Gradle, AGP, dependencies, build configuration, version catalogs, or plugins:

- Inspect current versions first (`libs.versions.toml`, `build.gradle(.kts)`).
- Use the project's Gradle wrapper, not a globally installed Gradle.
- Consult official Android tooling/skills and `android docs` where relevant.
- Run the relevant Gradle configuration/build tasks.
- Diagnose dependency/plugin resolution failures at the root cause instead of bypassing them.

Use `/agp-9-upgrade` only when the task actually involves AGP 9 migration. Use `/r8-analyzer` when R8/Proguard behavior is relevant.

## 14. Testing

Use `/testing-setup` when testing architecture/configuration itself is part of the task. Otherwise use the project's existing test stack — do not introduce an unrelated test framework merely because it exists elsewhere.

Possible validation may include JVM unit tests, Robolectric (only if already used by the project), Compose UI tests, instrumented Android tests, screenshot tests, and end-to-end tests.

## 15. Performance / tracing

For performance, jank, startup, latency, memory, or trace-related tasks, use `/perfetto-trace-analysis` and/or `/perfetto-sql`. Only use these when the problem genuinely warrants performance analysis — do not run Perfetto for ordinary feature implementation.

## 16. Play / platform-specific tooling

Use Play-related skills only when directly relevant: `/play-billing-library-version-upgrade`, `/play-policy-insights`, `/engage-sdk-integration`.

Use Wear/TV/XR-specific skills (`/wear-compose-m3`, `/leanback-to-compose-tv-migration`, `/display-glasses-with-jetpack-compose-glimmer`) only when the project or feature actually targets those platforms. Do not invoke them for normal phone/tablet apps.

## 17. Stop and report

Only stop when one of these is true:

1. The requested implementation is complete and all relevant available validation passes.
2. A genuine external/environment blocker exists that cannot be resolved from the repository, tools, documentation, or environment (for example: missing JDK, missing Android SDK platform, missing emulator/device, missing signing credentials, unavailable Play Console access).
3. A missing product/business decision blocks further progress.
4. The next step requires destructive or irreversible human approval.
5. The 5-iteration safety limit (see **6. Iterate safely**) is reached.

At the safety limit: stop safely, preserve evidence of the remaining failures, explain what remains unresolved, and never falsely report success.

Final report (concise):

- What changed: files/modules touched and a one-line summary.
- Official Android skills used, where notable.
- Android CLI/tooling used, where notable.
- What was validated: the commands/checks that passed.
- Runtime/device verification performed, if any.
- Validations that were not possible, and why (missing JDK/SDK/device/etc.).
- Result: green / blocked.
- Remaining non-blocking warnings, if any.

If blocked:

- What is blocking progress.
- What was already tried.
- The exact human decision or missing dependency required.

## Bypass and portability

- To disable any active stop enforcement for a session, delete the loop marker file (`.agents/state/loops-android.active` by default).
- To skip the loop entirely, do not invoke `/loops-android`; issue instructions normally.
- To adjust the loop, edit the installed skill file in your agent's skills directory.

### Portability note

This skill is agent-neutral. It assumes only that the agent can read files, run shell commands, search the codebase, and invoke other skills. It relies on the official Android CLI (`android`) rather than any MCP server, since no official Android MCP currently exists. Agent-specific lifecycle enforcement (such as a Stop hook) belongs in `integrations/<agent>/loops-android/` and is optional.
