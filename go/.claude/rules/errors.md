# Errors

- Library code returns `error`.
- Panic is reserved for impossible states that represent
  programmer bugs.
- Wrap with `%w` when adding context:
  `fmt.Errorf("load %s: %w", path, err)`.
- Sentinel errors are exported vars (`ErrNotFound`), checked with
  `errors.Is` / `errors.As`.
- Custom error types are structs with exported fields when
  callers need to branch on more than identity.
- Return the error as the last value.
- Don't log and return — the caller decides whether the error is
  worth logging.
