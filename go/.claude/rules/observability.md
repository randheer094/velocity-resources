# Logging & observability

- Use `log/slog` with key/value fields; one logger per process,
  enriched by middleware (request ID, user ID where safe).
- Redact secrets at the log boundary (tokens, credentials,
  cookies, full request bodies).
- Errors surface with enough context to debug from the log alone:
  caller identity, inputs, wrapped cause.
- Metrics + traces use OpenTelemetry; spans wrap external calls
  (DB, HTTP, queue) at minimum.
- Log levels are used with intent: `Debug` for development noise,
  `Info` for state changes, `Warn` for recoverable anomalies,
  `Error` only for failures a human should see.
