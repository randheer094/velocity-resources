# HTTP

## Server

- `http.Server` always sets `ReadHeaderTimeout`, `ReadTimeout`,
  `WriteTimeout`, and `IdleTimeout`. No naked
  `http.ListenAndServe`.
- Shutdown is graceful: `signal.NotifyContext(ctx, SIGINT,
  SIGTERM)` cancels the root context, which triggers
  `srv.Shutdown(timeoutCtx)`.
- Per-request context flows from `r.Context()` into every
  downstream call.
- Handlers validate input, then delegate; business logic does not
  live in `http` packages.
- A request-ID middleware adds an ID to the context and to every
  log line on that request.
- `/healthz` (liveness) and `/readyz` (readiness, including DB
  ping) are wired for any service that runs as a daemon.
- `pprof` endpoints are gated behind admin auth or a
  loopback-only listener — never exposed publicly.

## Client

- One `http.Client` per remote, reused across calls; never
  `http.DefaultClient` for production traffic.
- The client sets `Timeout` (overall) and a `Transport` with
  `MaxIdleConnsPerHost` tuned for the workload.
- Retries use exponential backoff with jitter and respect
  `Retry-After`; non-idempotent verbs need an idempotency key.
- Response bodies are always closed (`defer resp.Body.Close()`),
  even on error paths.
