---
name: prepare-for-pr
description: Run before opening a PR on this Go module. Formats, vets, lints, builds, runs unit + race + coverage, exercises any fuzz targets, and drives the fix-and-rerun loop on any failure so the caller can proceed straight to opening the PR.
---

# Prepare for PR (Go)

**Stop at the first red gate.** Each command below must exit 0.
Lint warnings, vet findings, race failures, and uncommitted
formatter / tidy diff are all "red" — fix the underlying issue
and re-run from that gate. Do not silence findings via inline
`//nolint` or `.golangci.yml` ad-hoc disables; suppression is a
last resort that needs reviewer sign-off.

Rules live in [`../../rules/`](../../rules/).

## Contract with the invoker

This skill **runs the gates and fixes the failures it finds.** It
is not a reporter. The invoker proceeds directly to PR creation
once this skill returns — do not finish with "tests failed,
please fix"; the PR step is queued behind you.

On a red gate:

1. Read the failure (test output, lint report, vet message) and
   find the root cause.
2. Fix the offending code. Lint / tidy / module config only
   changes with the user's sign-off.
3. Re-run from that gate; on green, continue down the list.
4. Re-run any earlier gate whose inputs were touched by the fix
   (e.g. a refactor invalidates `vet`, `test`, and `-race`).

Return control only when every gate has exited 0 and `git status`
is clean of formatter / tidy artifacts.

Escalation: if the same gate fails twice in a row for the same
root cause, stop and ask the user. Do not "fix" by adding
`t.Skip`, `//nolint`, weakening `.golangci.yml`, dropping
coverage thresholds, or making changes outside the scope of the
original task.

## Gates

Run sequentially from the module root. Each must exit 0.

1. `gofmt -w .` and `goimports -w .` — `git diff` is empty after.
2. `go vet ./...`.
3. `golangci-lint run` — warnings are errors. If the repo uses
   `staticcheck` directly, run that instead. Confirm
   `.golangci.yml` does not silently disable rules added since
   the last green CI run.
4. `go build ./...`.
5. `go mod tidy` — `git diff go.mod go.sum` is empty after.
6. `go test ./...`.
7. `go test -race ./...`.
8. `go test -cover ./...` — every package ≥ 90% statement
   coverage (a thin `cmd/<binary>/main.go` shim is the only
   exemption).
9. For each package with `Fuzz*` targets, run
   `go test -run=^$ -fuzz=FuzzName -fuzztime=30s ./<pkg>` per
   target. New targets land with at least one corpus entry under
   `testdata/fuzz/`.
10. DB / integration harness (`scripts/test-db.sh`,
    `make test-e2e`) if the repo has one.
11. `git diff origin/main...HEAD` — scrub debug prints, `t.Skip`
    without an issue link, unjustified deps, SQL string concat,
    new panics in library code, naked `http.ListenAndServe`,
    `http.DefaultClient`, and any context stored in a struct.
12. PR: title imperative, under 70 chars. Body = what, why, how
    to verify.

## Why PRs slip past lint / vet

Almost always one of:

- The gate ran a subset (e.g. `./pkg/...`). Always run from the
  module root with `./...`.
- `golangci-lint` config disabled rules silently; check
  `.golangci.yml` and remove ad-hoc disables.
- A failing test was marked `t.Skip` without follow-up — treat
  any new `t.Skip` in the diff as a red gate.
- A fuzz finding sits in `testdata/fuzz/` but no one re-ran the
  fuzzer. Re-run any fuzzer whose target package changed.
- `//nolint` was added to make the linter quiet. Remove and fix.
