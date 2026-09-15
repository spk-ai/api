# Anchored Volume Retirement

This dependent proposal extends `feat/resource-anchor-registry`; it is not a
published BSR capability or a stock Agyn release. Regenerate all participants
from this API branch before using the new operations.

`RunnerService.RemoveVolumeAnchored` is a distinct capability. Its request
contains the complete persisted bound PVC identity, including the immutable
volume owner and backend. An old runner must return `Unimplemented`; clients
must not fall back to `RemoveVolume` or `RemoveVolumeBound`.

`UpdateVolumeChecked.begin_anchored_removal` persists retirement intent while
excluding workload admission for the owner. `confirm_anchored_removal` matches
that intent with the native ABSENT response. `Volume` retains the original
binding, owner, reservation receipt and native observation after deletion.
Neither old checked operations nor reopening may replace this history.

This is explicit workspace retirement, not idle compute release. An ordinary
turn retains its workspace and volume owner. Unbound first provision requires
separate reconciliation; absence observations do not authorize adopting a PVC.

Native ABSENT observes both PVC and persistent-owner absence in the pinned
backend. A deletion acknowledgement, finalizer, garbage-collection request or
metering timestamp is not sufficient. Late children of a revoked owner UID can
still appear and require cleanup. This contract does not prove future-write
exclusion, authenticated deletion authority or node/storage fencing.

## Verification And Dependencies

`buf lint` and `buf breaking --against '.git#ref=HEAD'` pass for the additive
wire changes. Matching `feat/anchored-volume-removal` branches in
`spk-ai/k8s-runner`, `spk-ai/runners` and `spk-ai/agents-orchestrator` implement
the native handler, additive registry migration `0025` and both controller
paths. Their own reports distinguish source, database and native tests.

Drain and fence all writers before migration or rollout. Do not downgrade the
installed registry or substitute the focused controller for a DNS-compatible
deployment. No upstream PR, BSR publication or installed rollout is claimed.
