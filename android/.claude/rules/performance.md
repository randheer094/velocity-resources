# Performance

- Baseline profiles are generated for the host app module
  (`androidx.benchmark`); regenerate when startup-critical paths
  change.
- LeakCanary is on in `debug`, off in `release`.
- Recomposition discipline: hoist state, prefer `derivedStateOf`
  for derived values, key `remember` correctly, pass stable types.
- `StrictMode` is enabled in `Application.onCreate` for `debug`
  with `penaltyLog()` for both thread and VM policies.
- WorkManager for any background job that must survive process
  death; raw `Service` only with a documented reason.
