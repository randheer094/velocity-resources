---
name: prepare-for-pr
description: Run before opening a PR on this Android project. Boots an emulator if needed, then runs build, unit + connectedAndroidTest, lint (debug + release), detekt/ktlint as configured, verifies report directories are actually empty, and drives the fix-and-rerun loop on any failure so the caller can proceed straight to opening the PR.
---

# Prepare for PR (Android)

**Stop at the first red gate.** Every command below must exit 0
*and* leave its report directory free of new findings. Non-zero
exit, lint errors, detekt findings, ktlint findings, and warnings
treated as errors are all "red" — fix the underlying issue and
re-run from that gate. Do not silence findings with `@Suppress`,
baseline files, or rule overrides; suppression is a last resort
that needs reviewer sign-off.

Rules live in [`../../rules/`](../../rules/).

## Contract with the invoker

This skill **runs the gates and fixes the failures it finds.** It
is not a reporter. The invoker proceeds directly to PR creation
once this skill returns — do not finish with "lint failed, please
fix"; the PR step is queued behind you.

On a red gate:

1. Open the report (`*/build/reports/...`) and find the root
   cause.
2. Fix the offending code. Project config (lint / detekt rules,
   convention plugins, baseline files) only changes with the
   user's sign-off.
3. Re-run from that gate; on green, continue down the list.
4. Re-run any earlier gate whose inputs were touched by the fix
   (e.g. an architecture change invalidates `assembleDebug` and
   `test`).

Return control only when every gate has exited 0 with empty
reports and `git status` shows no stray artifacts.

Escalation: if the same gate fails twice in a row for the same
root cause, stop and ask the user. Do not "fix" by deleting
tests, adding `@Suppress`, baselining findings, weakening
`abortOnError` / `failFast`, or making changes outside the scope
of the original task.

## Required project configuration

If a gate "passes" while findings exist, the project's config is
wrong, not the gate. Verify before running:

- `android { lint { abortOnError = true; warningsAsErrors = true } }`
  in the convention plugin.
- The detekt config has `failFast: true` (or the relevant rules
  set to `severity: error`); the plugin uses
  `buildUponDefaultConfig = true`.
- The detekt and ktlint plugins are applied to **every** module,
  not just `app`. Verify with
  `./gradlew tasks --all | grep -iE 'detekt|ktlint'` — every
  module should show up.

## Boot an emulator (only if connectedAndroidTest is needed)

Reuse before creating. If `adb devices` already lists a booted
emulator, skip this section entirely. Otherwise pick the most
recently used AVD and boot it; only create a new one if none
exist.

```bash
# Skip if an emulator is already booted.
adb devices | awk 'NR>1 && $2=="device" && $1 ~ /^emulator-/' | grep -q . && exit 0

# Pick the most recently modified existing AVD.
avd=$(ls -t ~/.android/avd/*.ini 2>/dev/null | head -1 \
        | xargs -I{} basename {} .ini)

# Create one only if none exist.
if [[ -z "$avd" ]]; then
  android sdk install platform-tools "system-images;android-34;google_apis;x86_64"
  android avd create --profile pixel_6
  avd=$(ls -t ~/.android/avd/*.ini | head -1 | xargs -I{} basename {} .ini)
fi

android emulator --avd "$avd" &
adb wait-for-device
adb shell 'while [[ -z "$(getprop sys.boot_completed)" ]]; do sleep 1; done'
```

## Gates

Run sequentially from the repo root. Each must exit 0.

1. `android analyze` — module graph sane.
2. `./gradlew assembleDebug` — every variant builds.
3. `./gradlew test` — JVM unit suite, all modules.
4. `./gradlew lint lintRelease` — debug + release lint.
   `lintRelease` catches shrinker- and minify-only issues that
   debug lint misses. Exit 0 alone is not enough — confirm:
   ```bash
   find . -path '*/build/reports/lint-results-*.xml' \
        -exec grep -l 'severity="Error"\|severity="Fatal"' {} +
   ```
   Any output is a red gate.
5. `./gradlew detekt` — every module's detekt task runs. After
   it exits 0, verify no findings actually got through:
   ```bash
   find . -path '*/build/reports/detekt/*.xml' \
        -exec grep -l '<error' {} +
   ```
   Any output is a red gate.
6. `./gradlew ktlintCheck` — Kotlin style.
7. `./gradlew connectedAndroidTest` — instrumented suite on the
   booted emulator.
8. `git diff origin/main...HEAD` — scrub `Log.d` / `println`,
   `collectAsState` (must be `collectAsStateWithLifecycle`),
   hardcoded user-visible strings, new permissions, new
   dependencies missing from the catalog, additions to
   `lint-baseline.xml` or `detekt-baseline.xml`.
9. PR: title imperative, under 70 chars. Body = what, why, how
   to verify. Screenshots / recording for UI changes.

## One-shot equivalent (with device up)

```bash
./gradlew check lintRelease connectedCheck
```

`check` aggregates `test`, `lint`, `detekt`, and `ktlintCheck`
when their plugins are applied; `lintRelease` covers the release
variant; `connectedCheck` covers instrumented tests. On failure,
drill into `*/build/reports/` for the offending module and rule.

## Why PRs slip past lint / detekt

Almost always one of:

- The gate ran `lint` (debug only). Always run `lint lintRelease`.
- Detekt or ktlint isn't wired into every module, so the root
  task succeeds without analysing feature modules.
- Findings were silenced via `lint-baseline.xml` /
  `detekt-baseline.xml` / `@Suppress` instead of fixed.
- `abortOnError = false` or `failFast = false` lets findings pass
  with exit 0. Fix the config, don't fight the symptom.
- The gate was scoped to `:app` (`./gradlew :app:lint`); always
  run unscoped from the repo root.
- Reports were inspected only via the HTML output, which
  collapses identical findings — the XML grep above catches what
  the HTML hides.
