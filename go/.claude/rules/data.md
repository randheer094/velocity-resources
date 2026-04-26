# Database

- `*sql.DB` (or pgx pool) is constructed once and shared.
- `SetMaxOpenConns`, `SetMaxIdleConns`, and `SetConnMaxLifetime`
  are configured explicitly — defaults are wrong for production.
- Every query uses parameter placeholders; string concatenation
  into SQL is a security bug.
- Transactions are short; rollback is `defer`red immediately
  after `Begin`, and the function returns nil only after the
  explicit `Commit` succeeds.
- `Rows.Close()` is `defer`red right after a successful `Query`;
  `rows.Err()` is checked after the loop.
- Migrations live in version-controlled SQL files run by a
  dedicated tool (`migrate`, `goose`, `dbmate`); the application
  never auto-migrates on startup in production.
