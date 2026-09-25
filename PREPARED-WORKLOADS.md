# Prepared Workload Contract

Dependent proposal on the combined backend-identity API (`ad5405b`). This is not
a published capability or a drop-in platform upgrade. The dependent registry
contract below adds persistence commands; controller migration is still required.

## Contract Owners

The native lifecycle and no-fallback rules live beside `RunnerService`,
`PrepareWorkloadRequest`, `WorkloadBinding`, `ActivateWorkloadRequest` and
`RemovePreparedWorkloadResponse` in
[runner.proto](proto/agynio/api/runner/v1/runner.proto).
Registry authority and admission live beside `PreparedWorkloadPhase` and
`UpdatePreparedWorkloadRequest` in
[runners.proto](proto/agynio/api/runners/v1/runners.proto).

## Kubernetes Implementation

The prototype requires Kubernetes >=1.30, strict Pod-create field validation,
an intact trusted admission chain and scheduling-gate-aware schedulers. Prepared
Pods carry the binding without credentials. Activation requires the native Pod
UID, all claim UIDs/owners, live namespace identity and optimistic concurrency
preconditions. A per-Pod claim finalizer prevents name reuse during activation
and execution. Only absence of that Pod UID permits release of its hold.

Preparation attaches temporary Secrets to the returned Pod UID. An interrupted
preparation can still leave resources requiring reconciliation; no successful
reply means no authority to activate an inferred replacement. Secret garbage
collection is asynchronous, separate from the native workload absence receipt.

## Remaining Work

- Integrate the dependent registry contract and its matching database guards.
- Migrate both agent and sandbox controllers, including lost prepare/activate
  replies, cancellation, stale caller leases and recovery after process death.
- Audit authenticated routes and all native writers; protect gates, holds and
  identity annotations from modification outside this protocol. Version and
  namespace IDs are assertions by the backend, not proof of its authority.
- Reconcile late gated creates, delayed hold writes and orphaned startup Secrets.
  A delayed activation cannot create a Pod; a delayed preparation can still
  create a gated orphan. Node partitions, forced deletion, cluster cloning and
  storage fencing remain separate requirements.
- Coordinate rollout and full A2A/model lifecycle acceptance. Native model-free
  acceptance does not establish these control-plane or security guarantees.

## Registry Coordination

The source-adjacent registry contract above requires coordinated database guards
and controller migration. Caller authentication, native route enforcement,
late-operation reconciliation and node/storage fencing remain separate gates.
Cancellation after ACTIVATING cannot retract an already in-flight native call.

## Lost Preparation Observation

`feat/prepared-outcome-observation` is an additive follow-up to the inspection
API `24b73ca`. It introduces `ObserveWorkloadPreparation` without changing the
registry schema or the existing prepare/activate/remove messages. Lint and
breaking-change checks against the inspection proposal pass.

Discovery input, bounded output and retirement-only use are documented beside
`ObserveWorkloadPreparation` and its messages in
[runner.proto](proto/agynio/api/runner/v1/runner.proto).

NotFound, Unimplemented, old ownership semantics, changed snapshots and any
identity mismatch retain admission. There is deliberately no absent/safe-to-
retry discovery state. Initially absent or delayed Pod/PVC creates, old ownerless
Secrets, external credential revocation, authenticated routes, all-writer
enforcement and node/storage fencing still require separate mechanisms.

## Resource Anchors

`feat/resource-anchors` depends on the observation API `d6449dd`. It adds
`ReserveResourceAnchor`, `PrepareAnchoredWorkload` and `RemoveWorkloadAnchor`.
The registry must persist the exact workload and volume anchor UIDs **before**
authorizing any native Pod/PVC creation. This registry/controller integration
is not yet implemented by this API proposal.

Owner lifetimes, exact UID selection, activation/revocation ordering and
workload-versus-volume removal are documented beside `ResourceAnchor`,
`PrepareAnchoredWorkloadRequest` and `RemoveWorkloadAnchorRequest` in
[runner.proto](proto/agynio/api/runner/v1/runner.proto). Registry persistence and
dual revisions belong to `WorkloadResourceAnchors` in
[runners.proto](proto/agynio/api/runners/v1/runners.proto).
Registry persistence, all-writer guards, delayed hold reconciliation, legacy
adoption, authenticated owner/backend routes, node/storage fencing and a
coordinated rollout remain required. This is not a drop-in or production API.

## References

[Scheduling readiness](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-scheduling-readiness/)
defines gates and their stable version. [API concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)
describe strict validation and resource versions. [Finalizers](https://kubernetes.io/docs/concepts/overview/working-with-objects/finalizers/)
retain deleting objects until the responsible controller removes its key.
