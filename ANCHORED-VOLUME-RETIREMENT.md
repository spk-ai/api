# Anchored Volume Retirement

This dependent proposal extends `feat/resource-anchor-registry`; it is not a
published BSR capability or a stock Agyn release. Regenerate all participants
from this API branch before using the new operations.

See the [native retirement](proto/agynio/api/runner/v1/runner.proto) and
[registry evidence](proto/agynio/api/runners/v1/runners.proto) contracts.

This is explicit workspace retirement, not idle compute release. An ordinary
turn is not authorization to discard the workspace. Unbound first provision
requires separate reconciliation.

Late children of a revoked owner UID can still appear and require cleanup.
This contract does not prove future-write
exclusion, authenticated deletion authority or node/storage fencing.

## Verification And Dependencies

Historical proposal verification, not a new run on this checkout:

`buf lint` and `buf breaking --against '.git#ref=HEAD'` pass for the additive
wire changes. Matching `feat/anchored-volume-removal` branches in
`spk-ai/k8s-runner`, `spk-ai/runners` and `spk-ai/agents-orchestrator` implement
the native handler, additive registry migration `0025` and both controller
paths. Their own reports distinguish source, database and native tests.

Drain and fence all writers before migration or rollout. Do not downgrade the
installed registry or substitute the focused controller for a DNS-compatible
deployment. No upstream PR, BSR publication or installed rollout is claimed.
