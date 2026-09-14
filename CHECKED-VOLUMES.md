# Checked Volume Lifecycle Proposal

This branch adds a proposed control-plane/runner contract. It is not an accepted
Agyn architecture decision or a claim that deployed services implement it.

## Sequence

1. `CreateVolumeChecked` creates an unbound `PROVISIONING` record with
   `checked_lifecycle=true` and a positive `lifecycle_revision`. It never reopens
   an existing record implicitly.
2. After provisioning, validate inventory against the record's persistent
   owner, logical key and runner. `UpdateVolumeChecked(bind)` atomically pins
   its physical name, opaque backend UID and persistent identity labels and
   activates the record. Workload replacement does not change this binding.
3. `UpdateVolumeChecked(begin_removal)` reserves a durable removal intent for
   that binding, using the expected lifecycle revision. Only after this commit
   may a caller invoke the runner. Lost acknowledgements require rereading the
   record, not constructing another deletion target from current inventory.
4. Call `RemoveVolumeBound` with the intent's exact target. A mismatch is a
   conflict, not absence and not permission to adopt the replacement.
5. `PENDING` means the object still exists or deletion was requested. Only
   `ABSENT` authorizes `UpdateVolumeChecked(confirm_removal)` with the same
   intent ID, verified backend ID and record revision. Both response states must
   identify the expected backend. Billing `removed_at` is not this evidence.
6. Explicit revision-checked `reopen` preserves all logical ownership fields.
   A pending deletion prevents reopen; a confirmed deleted generation clears
   its old binding/intent before a different physical incarnation is bound.

Provisioning failure is distinct from removal. `fail_provisioning` records a
failure without inventing absence evidence. Reopening it does not establish
that an earlier in-flight backend create cannot still finish.

Every checked update uses compare-and-swap. Revision zero indicates an older
server and is never usable as an initial token. Metering-only updates do not
advance the lifecycle revision. A retry of begin keeps the existing intent;
the successful checked update still advances the record revision.

## Compatibility And Boundaries

### Storage Backend Identity

The dependent backend-identity extension adds a separate `RemoveVolumeBound`
RPC. Callers must not fall back to `RemoveVolumeChecked` or name-only removal:
older runners cannot enforce the new precondition and return `Unimplemented`
for the new method. Updated runners reject the old checked deletion method.

The extension carries `backend_id` in every inventory
item, the inventory envelope (including empty inventories), the checked removal
response and the registry confirmation request. It is an opaque, nonempty token
of at most 512 UTF-8 bytes without surrounding whitespace. It identifies the
storage scope, not the runner process, route address or PVC name. A runner restart
must preserve it; replacing or redirecting the backend must change it.

Bindings and removal intents retain that identity immutably. The runner must
verify the current backend before touching the target and before reporting
absence. The controller must reject missing/mixed inventory identities and
pending/absent responses that do not match the stored intent. The registry must
reject backend-less or mismatched confirmations and new unidentified bindings,
including through old SQL writers. No value may be inferred from an old record
or copied from unverified request metadata.

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
