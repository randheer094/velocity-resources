# Build, dependencies, layout

## Build

- Gradle wrapper is checked in; every invocation goes through
  `./gradlew`.
- Build scripts are Kotlin DSL (`build.gradle.kts`).
- Every dependency and version lives in
  `gradle/libs.versions.toml`; module scripts reference catalog
  aliases.
- Android / Kotlin / Compose configuration lives in convention
  plugins under `build-logic/`. Feature modules apply plugins,
  not raw config blocks.
- JVM toolchain is pinned in the convention plugin
  (`kotlin { jvmToolchain(17) }`); the K2 compiler is enabled.
- Kotlin compilation runs with `allWarningsAsErrors = true`.
- Android lint runs with `abortOnError = true` and
  `warningsAsErrors = true`; `lintRelease` is part of the gate.
- Annotation processing is KSP only; kapt is forbidden.
- Gradle configuration cache, build cache, and parallel execution
  are on by default in `gradle.properties`.
- Dependency versions in the catalog are exact (no `+`, no
  `latest.release`).
- `release` builds enable R8, code minification, and resource
  shrinking; `debug` does not.
- Signing config reads keystore path and passwords from env vars
  injected by CI; keystores aren't committed.

## Module layout

- `app/` — main application module; wires the Hilt graph, owns
  navigation.
- `<feature>/` — feature modules; each owns its `State` / `Intent`
  / `Effect` / `ViewModel`, its Composables, and its Hilt module.
  Cross-module visibility uses `internal` aggressively.
- `core/` — cross-feature code (design system, networking,
  persistence). Exposes interfaces; Hilt-bound impls live in the
  owning module.
- `build-logic/` — shared Gradle convention plugins.
