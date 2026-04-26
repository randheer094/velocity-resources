# Configuration

- Secrets come from environment variables.
- Runtime config is a typed struct loaded once at startup and
  passed explicitly into constructors — no globals.
- Feature flags are config.
- Config validation runs at startup; the process exits non-zero
  on missing or malformed values rather than failing later under
  load.
- `init()` is reserved for registering things that have no other
  hook (e.g., `database/sql` drivers); construction goes through
  explicit `New*` functions.
