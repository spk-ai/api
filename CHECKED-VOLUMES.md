# Checked Volume Lifecycle Proposal

This branch adds a proposed control-plane/runner contract. It is not an accepted
Agyn architecture decision or a claim that deployed services implement it.

## Contract Owners

See the [registry lifecycle](proto/agynio/api/runners/v1/runners.proto) and
[native identity/removal](proto/agynio/api/runner/v1/runner.proto) contracts.

## Compatibility And Boundaries

### Storage Backend Identity

Missing identities require explicit reconciliation, not an inferred backend or
rewritten binding. The wire-additive extension requires coordinated clients and
servers; it does not authenticate callers or protect cloned/restored identities.

The separate RPC names are deliberate: older servers return `Unimplemented`
instead of silently ignoring new request preconditions. Callers must not fall
back to legacy creation/update/removal on that error. Read-only additive fields
are insufficient for feature negotiation or deletion authorization.

Drain and audit all writers before rollout, including legacy name-only removal
and workload cleanup routes; source compatibility is not deployment compatibility.
Legacy record binding/reopening requires an explicit ownership audit. Do not
automatically protect whichever physical object happens to occupy a name.

Never strip finalizers or use unsafe deletion as a retry mechanism. The native
contract requires backend-enforced deletion preconditions, not operator name matching.

Identity labels are not caller authentication. Service authorization, trusted
confirmation reporters, delayed creates, partitioned nodes, storage-level
fencing, garbage collection and coordinated all-writer rollout remain separate
requirements. Absence at one observation is not a guarantee against a future
late create, and a registry confirmation endpoint trusts its authorized caller
to have obtained the matching runner evidence.
