# velocity migration prompt

Paste the body below into a velocity arch/code run (or run it yourself
in the velocity repo) to externalize the LLM prompts and failure
templates. The target repo is
[`randheer094/velocity`](https://github.com/randheer094/velocity); the
source of truth for prompts is this repo,
[`randheer094/velocity-resources`](https://github.com/randheer094/velocity-resources),
under `prompts/`.

---

## Goal

Replace velocity's hardcoded prompt strings (`internal/arch/prompt.go`,
`internal/code/prompt.go`) and failure-comment formatters
(`internal/arch/failure.go`, `internal/code/prompt.go` —
`formatFailureComment`, `formatIterateJiraComment`,
`formatIteratePRComment`) with a runtime fetch from the
`velocity-resources` repo. velocity should fetch
`prompts/manifest.yaml` and the referenced templates on daemon
startup, parse them with `text/template`, cache them, and render at
each call site.

## Files to change

- `internal/config/config.go` — add a `PromptsConfig` struct and wire
  it into `Config`. Defaults to:
  ```yaml
  prompts:
    source_url: https://raw.githubusercontent.com/randheer094/velocity-resources/main/prompts
    fetch_timeout_sec: 10
  ```
  Validate that `source_url` parses as http(s).
- `config.example.yaml` — document the new `prompts:` block.
- `internal/prompts/prompts.go` (new package) — owns:
  - `Manifest` struct mirroring `manifest.yaml` (`Version`,
    `Prompts []Entry{ID, Path, Description, Placeholders}`).
  - `Load(ctx, sourceURL string, timeout time.Duration) (*Store, error)`
    that fetches the manifest, then each template, parses each with
    `text/template.New(id).Option("missingkey=error")`, and returns a
    `*Store` keyed by id.
  - `(*Store).Render(id string, data any) (string, error)` — looks up
    the cached `*template.Template` and executes it into a
    `bytes.Buffer`. Returns an error if the id is unknown or
    `Execute` fails.
  - Bundled fallbacks: embed the current prompt strings via
    `//go:embed defaults/*.md defaults/manifest.yaml` so the daemon
    is bootable when the source URL is unreachable. On fetch
    failure, log a warning and load from the embedded set instead.
- `internal/prompts/defaults/` (new directory) — copy the six
  template files plus `manifest.yaml` from
  [`velocity-resources/prompts/`](https://github.com/randheer094/velocity-resources/tree/main/prompts).
- `cmd/<daemon-entrypoint>/main.go` (the binary that calls
  `config.SetDir` / `config.Reload`) — after config loads, call
  `prompts.Load(...)` once and stash the `*Store` in a package-level
  `prompts.Shared()` accessor (mirror the `jira.Shared()` pattern).
  Fail-soft: log the error, fall back to defaults, keep starting.
- `internal/arch/prompt.go` — delete `archSystemPrompt`, `planBegin`,
  `planEnd`, `buildArchPrompt`. Replace `buildArchPrompt` with a
  thin wrapper that calls
  `prompts.Shared().Render("arch_plan", archPlanData{PlanBegin, PlanEnd, ParentKey, Requirement})`.
  Keep `planBegin`/`planEnd` as package consts since `extractPlan` in
  `arch.go` still references them.
- `internal/code/prompt.go` — delete `codeSystemPrompt`,
  `iterateSystemPrompt`, `buildCodePrompt`, `buildIteratePrompt`,
  `formatFailureComment`, `formatIterateJiraComment`,
  `formatIteratePRComment`. Replace each with a thin wrapper that
  calls `prompts.Shared().Render(<id>, ...)`. Keep `BuildPRBody` —
  the PR body is not in the manifest.
- `internal/arch/failure.go` — replace the local
  `formatFailureComment` with
  `prompts.Shared().Render("failure_jira", ...)`.
- `internal/arch/arch.go` and `internal/code/code.go` /
  `internal/code/iterate.go` — call sites of the deleted helpers
  must use the new `prompts.Shared()` paths. Renderer errors should
  be returned up the existing `*stage` error path so a missing/
  malformed template surfaces as a stage failure rather than a
  silent empty prompt.
- Tests:
  - `internal/prompts/prompts_test.go` — covers manifest parse, fetch
    + parse end-to-end against an `httptest.Server`, render with a
    valid struct, render error on unknown id, render error on
    missing key, fallback to embedded defaults when the URL 404s.
  - Update `internal/arch/*_test.go` and `internal/code/*_test.go`
    helpers (e.g. `main_test.go`, `setup_test.go`) so the prompt
    store is initialized from the embedded defaults at test setup.
    The simplest hook is `prompts.SetSharedForTest(*Store)`.

## Constraints

- Match existing code style: lowercase package, `slog` for logging,
  `errors.New` / `fmt.Errorf("...: %w", err)` for errors, no panics
  outside `recover` boundaries.
- The fetch must time out (use `cfg.Prompts.FetchTimeoutSec`,
  default 10s) and respect `context.Context` so daemon shutdown is
  not blocked.
- `prompts.Shared()` must be safe to call from multiple goroutines —
  arch and code runs are concurrent. Either guard the package-level
  `*Store` with a `sync.RWMutex`, or set it once during startup and
  treat it as immutable thereafter (preferred).
- Do NOT remove `planBegin` / `planEnd` constants from
  `internal/arch/`. They are referenced by `extractPlan`; the
  template just needs them as render-time inputs.
- Output formatting MUST be byte-identical to today's prompts after
  rendering against representative inputs. Diff the output against
  the current `fmt.Sprintf` results in a unit test before deleting
  the old strings.
- Failure-comment rendering must not itself panic on render error.
  If `prompts.Shared().Render("failure_jira", ...)` errors, log it
  and fall through to a minimal hardcoded fallback like
  `fmt.Sprintf("Velocity %s failed at stage %s: %s", role, stage, msg)`
  — losing the formatted comment is preferable to losing the
  failure record.

## Acceptance criteria

- `go build ./...` and `go test ./...` pass.
- Starting the daemon against the live source URL logs
  `prompts: loaded N templates from <url>` and renders the same
  output as before for a sample arch/code/iterate/failure render.
- Starting the daemon with `prompts.source_url` pointing to an
  unreachable host logs a warning and continues using the embedded
  defaults. A subsequent arch run still produces a valid plan
  prompt.
- Editing `prompts/arch/plan.md` in `velocity-resources` and
  restarting the daemon (no rebuild) picks up the change on the
  next arch run.
- No call site outside `internal/prompts/` references the
  hardcoded prompt strings; `git grep 'You are a senior software'`
  returns matches only inside `internal/prompts/defaults/`.

## Out of scope

- Reload-without-restart (`velocity reload-prompts`). Restart is
  fine for v1.
- Auth / signed templates. The repo is public; we trust GitHub.
- Prompt versioning beyond the `version:` integer in
  `manifest.yaml`. Don't add per-prompt version pinning yet.
- Touching the `android/` and `go/` Claude templates in
  `velocity-resources` — those are unrelated.
- The `BuildPRBody` helper in `internal/code/prompt.go`. PR bodies
  stay inline.

## Context

- `manifest.yaml` schema (already published at
  `prompts/manifest.yaml` in `velocity-resources`):
  ```yaml
  version: 1
  prompts:
    - id: <string>
      path: <relative path from manifest.yaml>
      description: <human-readable>
      placeholders: [<field>, ...]
  ```
- Placeholder schema for each prompt is in the manifest; the data
  structs you pass to `Render` should mirror those names exactly.
  Example for `arch_plan`:
  ```go
  type archPlanData struct {
      PlanBegin   string
      PlanEnd     string
      ParentKey   string
      Requirement string
  }
  ```
- `text/template.Option("missingkey=error")` is required — a typo in
  a placeholder name should fail render, not silently produce
  `<no value>` in a Jira comment.
- The current `formatFailureComment` lives in BOTH
  `internal/arch/failure.go` and `internal/code/prompt.go` with
  identical bodies. Both call sites map to the same `failure_jira`
  template id.
- Existing config validation in `internal/config/config.go` is
  field-by-field with `errors.New`. Match that style for the
  `Prompts` block.
- Workspace / status / queue plumbing is unchanged. This change is
  purely a string-substitution refactor plus one new package.
