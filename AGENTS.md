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
- Keep `docs/catalog.json` as the version-1 index of meaningful Markdown and
  `AGENTS.md`, not a source inventory. Preserve its stable document IDs.
- Use the workspace Navigator first when available: `repos [--worktrees]`,
  then `scan --repo api`, then batch `inspect api::owner api::related-owner`.
  Use `docs --repo api` and `doc api::document-id` for guides. Worktrees use
  Git-discovered basenames, optionally selected by `--worktree`.
  Standalone contributors need no Navigator: use `git diff upstream/main...HEAD`
  (or the reviewed base), `rg`, Go/Buf tools and adjacent tests.
  Do not copy Navigator tooling, dependencies or machine-specific paths here.
  Optional cross-repo `@see repo::extensionless/component` links belong at genuine
  contract owners; same-repo `@see` paths retain the source extension.
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
