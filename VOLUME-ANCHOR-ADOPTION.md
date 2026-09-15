# Existing Volume Anchor Adoption

Dependent, additive RunnerService proposal for migrating a previously bound
workspace to a persistent resource anchor. It requires coordinated registry and
controller changes; these four native methods alone are not a rollout protocol.
Existing repository licensing is unchanged.

## Contract

The caller first durably blocks admission for the entire owner and drains all
old writers, including already accepted operations. It pins the original checked
`VolumeListItem` and an operation UUID. This is not first allocation: the original
PVC must exist, and an absent/replaced object requires reconciliation.

1. `ReserveVolumeAnchorAdoption` creates only owner/journal metadata. The original
   PVC UID/name, ownership labels, backend incarnation and native spec SHA-256
   identify the source. The receipt has separate persistent-owner and immutable
   journal UIDs. The native owner pins the journal before returning it. Persist
   the complete receipt before applying it; compare retries with stored identity.
2. `ApplyVolumeAnchorAdoption` atomically attaches that owner and a migration
   hold to the same PVC. `APPLIED` does not mean reusable. Persist the resulting
   original-PVC binding while the owner-wide admission block remains held.
3. `ObserveVolumeAnchorAdoption` is read-only. It requires the exact complete
   receipt and distinguishes `RESERVED`, `APPLIED` and `READY`; missing or changed
   evidence is an error, not absence, success, or permission to recreate anything.
4. `FinalizeVolumeAnchorAdoption` authorizes only that owner transition and
   removal of that operation's migration hold. A partial finalize remains
   resumable but blocked. Commit separately observed `READY` evidence before
   reopening owner admission.

The original checked volume, adoption receipt and resulting anchored binding
must remain distinguishable from `VolumeAnchorReservation`, which represents
first provisioning. A registry migration must not manufacture an allocation
reservation for an existing PVC. This API branch does not yet add those registry
fields, guards or a coordinator.

No RPC creates, resizes, renames, replaces or deletes workspace storage; none
creates compute, supplies credentials or replays a task. UID/revision conflicts
return errors instead of retargeting. A `READY` observation is not authority to
retry a previously interrupted agent turn.

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
