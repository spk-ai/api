# API Contribution Guide

## Owners
- `proto/agynio/api/runner/v1/runner.proto`: native binding, anchors, adoption,
  revocation and exact removal.
- `proto/agynio/api/runners/v1/runners.proto`: registry revisions, admission,
  migration provenance and removal confirmation.
- Coordinate these contracts with Runners, k8s-runner, agents-orchestrator and
  Gateway. Additive fields alone are not capability negotiation.

## Documentation
- Keep implementation invariants beside their handwritten Go or protobuf owner.
  Update those comments and focused tests when behavior changes; Markdown holds
  operations, cross-repository decisions, security boundaries and dated evidence.
- Start at `docs/catalog.json`. Maintain its version-1 document entries
  (`id`, `path`, `title`, `purpose`, `kind`) for meaningful Markdown and
  `AGENTS.md` only, using repository-relative paths. Do not index generated code.
- When a compatible structural navigator is available, discover repositories and
  components first, then batch-inspect selected owners and their related tests.
  Otherwise use native declarations, imports, RPC types and adjacent tests;
  `git diff upstream/main...HEAD` (or the reviewed base) identifies the changes.
  Do not add a navigator dependency or machine-specific paths to this repository.
  Navigator commands are `repos [--worktrees]`,
  `scan --repo REPO`, `inspect REPO::path` and `docs --repo REPO`.
  IDs here are `api`, `runners`, `orchestrator`, `k8s-runner` and `gateway`.
  Worktrees use Git-discovered basenames, optionally selected by `--worktree`.
  At genuine cross-repo owners, optional `@see repo::extensionless/component`
  references can aid navigation; same-repo `@see` paths retain the extension.
- Preserve dated verification, failures, skips and dependency revisions as
  historical evidence; do not silently turn them into current acceptance claims.
- Do not edit generated sources or applied SQL migrations, including comments.
  Migration bytes participate in recovery/backup checks. Explain SQL behavior
  beside the owning Go caller and link the original migration; schema changes
  require a separately reviewed additive migration. Preserve licensing/notices.

## Verification
Run `buf lint` and `buf breaking --against '.git#ref=HEAD'` for local
comment-only edits; use the reviewed integration base for schema changes.
Compare `buf build --exclude-source-info --as-file-descriptor-set` outputs
before/after comment edits to demonstrate wire invariance. Do not regenerate
consumers merely to change comments.
