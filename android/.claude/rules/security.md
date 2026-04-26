# Security

- `android:usesCleartextTraffic="false"` in the manifest;
  exceptions live in `network_security_config.xml` with a comment
  explaining why.
- `WebView` runs with `setJavaScriptEnabled(true)` only when the
  loaded origin is trusted and HTTPS-only. `setAllowFileAccess`,
  `setAllowContentAccess`, and `setAllowFileAccessFromFileURLs`
  default to false.
- Signing keys, API keys, and Firebase prod configs are injected
  via env vars in CI; `local.properties`, keystores, and
  service-account JSON stay out of git.
- New runtime permissions ship a documented rationale UI.
- No PII, tokens, or auth headers in logs (any variant).
- Exported Activities / Services / Receivers / Providers declare
  `android:exported` explicitly; default to `false` unless an
  external caller is intended.
