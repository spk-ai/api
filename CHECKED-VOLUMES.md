# Checked Volume Lifecycle Proposal

This branch adds a proposed control-plane/runner contract. It is not an accepted
Agyn architecture decision or a claim that deployed services implement it.

## Contract Owners

[Registry protobuf](proto/agynio/api/runners/v1/runners.proto):
`CreateVolumeCheckedRequest`, `UpdateVolumeCheckedRequest`, `Volume` and
`VolumeRemovalIntent` own creation, CAS, binding, reopen and confirmation.
[Native protobuf](proto/agynio/api/runner/v1/runner.proto):
`VolumeListItem`, `ListVolumesResponse` and `RemoveVolumeBound` own backend
identity, immutable deletion targets and PENDING/ABSENT evidence.

## Compatibility And Boundaries

### Storage Backend Identity

The dependent backend-identity extension adds a separate `RemoveVolumeBound`
RPC. Callers must not fall back to `RemoveVolumeChecked` or name-only removal:
older runners cannot enforce the new precondition and return `Unimplemented`
for the new method. Updated runners reject the old checked deletion method.

Backend-token shape, inventory completeness and matching removal evidence are
specified beside those protobuf messages, not inferred from routing metadata.

The Kubernetes implementation uses the configured namespace name and API-issued
UID, checked before and after the namespaced operation. Namespace disappearance
or replacement is an error, not volume absence. Missing identities require
explicit reconciliation; migration must not invent a backend or rewrite an
immutable binding. This extension is wire-additive but intentionally requires
coordinated clients and servers. It does not add workload-start fencing,
authenticate the caller, protect cloned/restored cluster identities or fence
partitioned nodes and delayed operations.

The separate RPC names are deliberate: older servers return `Unimplemented`
instead of silently ignoring new request preconditions. Callers must not fall
back to legacy creation/update/removal on that error. Read-only additive fields
are insufficient for feature negotiation or deletion authorization.

The registry must prevent older binaries from mutating a checked lifecycle,
including after reopen. Legacy name-only runner removal and workload cleanup
with `remove_volumes=true` must not bypass the checked path. Drain and audit all
writers before rollout; source compatibility is not deployment compatibility.
Legacy record binding/reopening requires an explicit ownership audit. Do not
automatically protect whichever physical object happens to occupy a name.

Runner implementations must verify the expected incarnation and ownership
atomically with deletion. Kubernetes supports
[UID and resource-version preconditions](https://kubernetes.io/docs/reference/kubernetes-api/definitions/preconditions-v1-meta/).
A fresh GET can supply a fresh resource version after harmless metadata updates,
but cannot change the durable expected UID or owner. Never strip finalizers or
use unsafe deletion as a retry mechanism.

Identity labels are not caller authentication. Service authorization, trusted
confirmation reporters, delayed creates, partitioned nodes, storage-level
fencing, garbage collection and coordinated all-writer rollout remain separate
requirements. Absence at one observation is not a guarantee against a future
late create, and a registry confirmation endpoint trusts its authorized caller
to have obtained the matching runner evidence.
