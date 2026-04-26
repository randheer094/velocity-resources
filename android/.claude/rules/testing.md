# Testing

- **Unit tests** under `src/test/`. Every reducer branch /
  use-case / mapper ships a JVM test (Turbine for `Flow`, MockK
  for collaborators).
- **Instrumented / E2E tests** under `src/androidTest/`. Every new
  Composable / screen / navigation edge / DI binding ships an
  instrumented test using `@HiltAndroidTest` + Compose UI test.
- Compose UI assertions prefer roles and content descriptions
  over raw text so the suite stays locale-stable.
- Bug fixes ship with a regression test that fails before the fix.
- Both suites pass before every PR.
