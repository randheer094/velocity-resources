# Testing

- Every exported function has at least one unit test covering a
  happy path and the error path(s) a caller would hit.
- Bug fixes ship with a regression test that fails before the fix.
- Anything with > 1 case is table-driven; subtests use `t.Run`.
- `go test -race ./...` passes.
- External dependencies (DB, HTTP, filesystem) are exercised via a
  harness and skipped when the harness isn't available.
- Per-package statement coverage stays ≥ **90%** (a thin
  `cmd/<binary>/main.go` shim is the only exemption).
- Parsers, decoders, and any code that consumes untrusted bytes
  ship a `Fuzz*` target; CI runs each with `-fuzztime=30s`. Fuzz
  corpus entries (`testdata/fuzz/`) are committed.
- Hot paths covered by `Benchmark*` with `-benchmem`; performance
  regressions flagged in review.
- `t.Parallel()` on tests that have no shared state — and never
  on tests that do.
