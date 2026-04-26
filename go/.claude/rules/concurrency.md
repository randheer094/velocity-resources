# Concurrency

- Every blocking call accepts `context.Context` as its first
  argument and respects cancellation.
- Never store a `context.Context` in a struct; pass it through.
- Every goroutine has a clear owner that waits for it
  (`sync.WaitGroup`, `errgroup.Group`) or is tied to a lifecycle
  with a documented stop signal.
- Parallel fan-out with first-error semantics uses
  `golang.org/x/sync/errgroup` — not hand-rolled wait groups.
- Channels: the **sender** closes; receivers never close. Default
  to unbuffered; pick a buffer size only with a reason.
- A mutex is allowed only to protect state private to a struct,
  held for the smallest scope that's correct. Embed `sync.Mutex`
  as an unexported field, never on the public API.
- `-race` must pass in CI.
- Long-lived goroutines are covered by a `goleak.VerifyNone(t)`
  check or the package-level `TestMain` equivalent.
