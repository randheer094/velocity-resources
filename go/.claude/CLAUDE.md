# Go — Claude Code guide

Technical guide for Claude Code on this Go module. Project-
specific context (what this service does, owners, deployment
targets, runbooks) belongs in the repo-root `CLAUDE.md`, not here.

## Build, test, run

- `go build ./...` — compile every package.
- `go test ./...` — unit test suite.
- `go test -race ./...` — race detector; must pass in CI.
- `go vet ./...` — static analysis.
- `gofmt -l .` — must print nothing; `gofmt -w .` to fix.
- `goimports -w .` — import order + missing imports.
- `go mod tidy` — must produce no diff.
- `golangci-lint run` — lint gate when configured.

## Rules — read what applies to the current change

- [`rules/errors.md`](./rules/errors.md) — wrapping, sentinels,
  panic policy.
- [`rules/concurrency.md`](./rules/concurrency.md) — context,
  goroutines, channels, mutex, leak detection.
- [`rules/http.md`](./rules/http.md) — server timeouts +
  graceful shutdown; client reuse + timeouts; pprof gating.
- [`rules/data.md`](./rules/data.md) — `sql.DB` pool,
  parameterised queries, transactions, migrations.
- [`rules/config.md`](./rules/config.md) — env vars, typed
  config, startup validation.
- [`rules/observability.md`](./rules/observability.md) —
  `log/slog`, OpenTelemetry, redaction, log levels.
- [`rules/testing.md`](./rules/testing.md) — table tests, race,
  coverage, fuzz, bench.
- [`rules/security.md`](./rules/security.md) — input validation,
  TLS, constant-time compare.
- [`rules/style.md`](./rules/style.md) — gofmt, naming,
  golangci-lint.
- [`rules/build.md`](./rules/build.md) — build flags, deps,
  module layout.

## Before a PR

[`skills/prepare-for-pr/SKILL.md`](./skills/prepare-for-pr/SKILL.md).
Stop at the first red gate; fix, then re-run from that gate. Do
not open a PR with any non-zero exit from the listed commands.
