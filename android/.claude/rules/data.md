# Persistence & networking

## Persistence

- Room for relational storage, processed via KSP. The `schemas/`
  directory is exported and committed; every schema bump ships a
  `Migration` with a migration test.
- DataStore (Preferences or Proto) for key/value config — never
  `SharedPreferences`.
- Secrets at rest go through Jetpack Security
  (`EncryptedSharedPreferences` or a Tink-backed wrapper around
  DataStore); plaintext keys never touch disk.

## Networking

- Retrofit + OkHttp + Kotlinx Serialization.
- A single `OkHttpClient` is provided by Hilt with explicit
  timeouts (connect / read / write = 30s) and a shared
  `ConnectionPool`.
- `HttpLoggingInterceptor` is wired only in debug builds.
- No request/response bodies in release logs; redact
  `Authorization` and `Cookie` headers at the interceptor.
- Retries with exponential backoff + jitter for idempotent calls;
  non-idempotent verbs need an idempotency key.
