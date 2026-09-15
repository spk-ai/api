# Preparation Revocation

Dependent, unreleased extension of the resource-anchor contract. It addresses
an authorized preparation whose Pod/PVC creation response was never recorded.
NotFound alone cannot settle that attempt: a native CREATE may still arrive.
There is no change to A2A task routing, agent profiles or workflow code.

## Protocol

1. Persist an unbound anchored workload in `REMOVING` before native recovery.
2. `RevokeWorkloadPreparation` atomically competes with activation using the
   workload anchor's UID and resource version. It rejects a claimed activation,
   validates the complete persistent-volume anchor set, and returns a durable
   `PreparationRevocation` record. Its `instance_uid` is the revocation record's
   UID, not a fabricated Pod UID. `selected_pod_uid` is optional.
3. Persist that exact receipt with `RecordPreparationRevocation` through
   `UpdateAnchoredWorkload`, checking both preparation and resource revisions.
   This does not yet release admission or create a `WorkloadBinding`.
4. `ObservePreparationRevocation` reads the exact persisted receipt and observes
   Pod/owner cleanup and the full volume partition. `PENDING` retains admission.
   `POD_ABSENT` contains exact found PVC bindings and absent volume IDs.
5. Bind newly discovered PVCs through the existing checked-volume API first.
   Never replace a known PVC UID or classify its absence as first provisioning.
6. Persist `ConfirmPreparationRevocation` as a separate dual-revision operation.
   The registry checks the volume records under its owner/admission lock before
   marking `REMOVED`. Proof and observation remain immutable workload history.

The two registry messages live in `WorkloadResourceAnchors` fields 4 and 5.
Update operations use previously unused oneof fields 10 and 11. No existing
field number or ordinary Pod-binding/removal meaning changes.

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
