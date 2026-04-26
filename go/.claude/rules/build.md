# Build, dependencies, layout

## Build

- Single `go.mod` per repo.
- `go` directive pins the minor; `toolchain` directive pins the
  patch.
- Release build: `CGO_ENABLED=0 go build -trimpath
  -ldflags="-s -w -X main.version=<git sha>"`.
- Each `cmd/<binary>/` produces one static binary.

## Dependencies

- Keep `go.mod` minimal. Every new direct dependency is justified
  in the PR body.
- Reach for the standard library first; a third-party package
  needs a clear cost argument in the PR body.
- Run `go mod tidy` before every commit; `go.sum` is committed.
- `replace` directives in the committed `go.mod` require a
  one-line reason comment next to them.
- Vendoring stays off; the module proxy is the cache.

## Layout

- `cmd/<binary>/` — entry points; one directory per binary. Thin
  `main` that wires flags, config, and a context, then calls into
  `internal/`.
- `internal/` — packages private to this module. Most logic lives
  here.
- `pkg/` — only if other modules import it.
- `_test.go` files live next to the code they test.
- No `util` / `common` / `helpers` packages — name by capability.
