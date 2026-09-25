# Preparation Revocation

Dependent, unreleased extension of the resource-anchor contract. It addresses
an authorized preparation whose Pod/PVC creation response was never recorded.
NotFound alone cannot settle that attempt: a native CREATE may still arrive.
There is no change to A2A task routing, agent profiles or workflow code.

## Contract Owners

The native proof and cleanup partition are defined beside `PreparationRevocation`,
`RevokeWorkloadPreparation` and `ObservePreparationRevocationResponse` in
[runner.proto](proto/agynio/api/runner/v1/runner.proto). Separate record/confirm
writes and dual revisions are defined beside `WorkloadResourceAnchors`,
`RecordPreparationRevocation` and `ConfirmPreparationRevocation` in
[runners.proto](proto/agynio/api/runners/v1/runners.proto).

## Boundaries

The receipt proves that native activation was revoked before it was claimed.
`POD_ABSENT` proves a current observation, not exclusion of all future writes or
node-partition fencing. A late PVC must retain its original volume anchor and
any previously recorded physical UID. Recovery never retries the agent turn.

Missing native ownership without a matching revocation record remains unknown.
An unsupported RPC or registry operation fails closed; callers must not fall
back to deleting the workload owner and inferring completion from its absence.

Coordinate registry migration 0026, native runner, controller, regenerated
consumers and all writers before enabling this capability. These sources are a
proposal, not a published BSR module or an installed production upgrade. Receipt
retention, authenticated future-write fencing and hardened deployment remain
separate requirements. Existing repository licensing is unchanged.

Validation on 2026-09-15: `buf lint` and `buf breaking --against '.git#ref=HEAD'`
passed against the preceding anchored-retirement contract.
