# Code style

- Default to no comments; add one only when the WHY is non-obvious
  (hidden constraint, subtle invariant, workaround).
- Doc comments: one line where possible.
- Coroutines use an explicit dispatcher (`Dispatchers.IO` /
  `Default`), injected via Hilt so tests can substitute
  `StandardTestDispatcher`.
- Logging goes through Timber (debug tree only in debug builds);
  direct `Log.*` calls are caught by lint.
- ktlint / detekt is the source of truth for layout, imports, and
  naming — fix findings rather than suppress them. Suppression
  needs reviewer sign-off.
- Prefer `internal` over `public` for module-internal symbols;
  `data class` over hand-rolled `equals`/`hashCode`; `value class`
  for type-safe wrappers around primitives.
