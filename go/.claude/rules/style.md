# Code style

- Run `gofmt` + `goimports` before every commit.
- Exported symbols have doc comments.
- Default to no inline comments; add one only when the WHY is
  non-obvious (hidden constraint, subtle invariant, bug
  workaround).
- Keep functions short enough that the body fits on one screen.
- Avoid package-name stutter — use names like `foo.Client`,
  `foo.New`.
- Use `any` instead of `interface{}`.
- Accept interfaces, return concrete types; define interfaces
  near the consumer, not the producer.
- Preallocate slices (`make([]T, 0, n)`) when the size is known.
- Times at boundaries are UTC; durations come from `time.Since`
  on monotonic clocks.
- `golangci-lint` (errcheck, govet, staticcheck, ineffassign,
  revive, gocritic) is the lint gate; warnings are errors in CI.
