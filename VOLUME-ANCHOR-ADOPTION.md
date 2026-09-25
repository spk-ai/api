# Existing Volume Anchor Adoption

Dependent, additive RunnerService proposal for migrating a previously bound
workspace to a persistent resource anchor. It requires coordinated registry and
controller changes; these four native methods alone are not a rollout protocol.
Existing repository licensing is unchanged.

## Contract Owners

The native sequence, original-PVC identity and distinct migration receipt live
beside `VolumeAnchorAdoption` and its four RPCs in
[runner.proto](proto/agynio/api/runner/v1/runner.proto). Owner-wide admission and
append-only progress live beside `VolumeAnchorMigration` in
[runners.proto](proto/agynio/api/runners/v1/runners.proto).

Drain all old writers, including accepted operations, before migration. Storage
adoption is not first allocation or permission to retry an interrupted turn.
Registry guards, the operator coordinator and native runner must be coordinated;
publishing these RPCs alone is not a rollout.

## Limits

An API observation cannot fence a partitioned node or a delayed old writer.
Authenticated all-writer enforcement, a durable owner admission gate and the
coordinated deployment/drain are required. Restore must preserve the native
identities or use explicit reconciliation, not silently rebind names.

Native receipts require retention/backup policy. Do not deploy this contract by
publishing an API alone or by falling back to older RPCs on `Unimplemented`.

## Verification

`buf lint` and `buf breaking . --against '.git#ref=23d3073'` validate this additive
extension against the preceding preparation-revocation contract. Native fake-API
and actual Kubernetes/process acceptance live in the dependent k8s-runner branch.
