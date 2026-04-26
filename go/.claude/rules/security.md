# Security

- Validate every input at the system boundary (HTTP handler, CLI
  flag, env var, webhook payload).
- Parameterise every SQL query.
- Compare secrets with `crypto/subtle.ConstantTimeCompare`.
- HMAC / signature verification happens before any other parsing
  of untrusted input.
- JSON decoders use `json.Decoder` + `DisallowUnknownFields()` for
  any externally-supplied payload.
- TLS is on for any network listener exposed beyond localhost;
  new code uses `MinVersion: tls.VersionTLS13`.
- Never log full request bodies, `Authorization` headers, or
  cookies, even at debug level.
