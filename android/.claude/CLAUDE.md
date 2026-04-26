# Android — Claude Code guide

Technical guide for Claude Code on this Android project. Project-
specific context (what this app does, owners, key URLs, runbooks)
belongs in the repo-root `CLAUDE.md`, not here.

## Build, test, run

- `./gradlew assembleDebug` — debug APK for every variant.
- `./gradlew test` — JVM unit tests across modules.
- `./gradlew connectedAndroidTest` — instrumented / E2E tests on
  a connected device or emulator.
- `./gradlew lint lintRelease` — Android lint, debug + release.
- `./gradlew detekt` — Kotlin static analysis (when configured).
- `./gradlew ktlintCheck` — Kotlin style (when configured).
- `./gradlew check` — umbrella: unit tests + lint + detekt /
  ktlintCheck (whichever plugins are applied).

## Rules — read what applies to the current change

- [`rules/architecture.md`](./rules/architecture.md) — MVI,
  Compose, Hilt, coroutines & Flow.
- [`rules/ui.md`](./rules/ui.md) — strings, dimensions,
  accessibility, image loading.
- [`rules/data.md`](./rules/data.md) — persistence (Room,
  DataStore), networking (Retrofit / OkHttp).
- [`rules/testing.md`](./rules/testing.md) — JVM and instrumented
  test requirements.
- [`rules/security.md`](./rules/security.md) — manifest,
  secrets, exported components, logging boundaries.
- [`rules/performance.md`](./rules/performance.md) — baseline
  profiles, LeakCanary, recomposition, WorkManager.
- [`rules/style.md`](./rules/style.md) — comments, logging,
  detekt / ktlint posture.
- [`rules/build.md`](./rules/build.md) — Gradle, KSP, R8,
  module layout.

## Before a PR

[`skills/prepare-for-pr/SKILL.md`](./skills/prepare-for-pr/SKILL.md).
Stop at the first red gate; fix, then re-run from that gate. Do
not open a PR with any non-zero exit from the listed commands.
