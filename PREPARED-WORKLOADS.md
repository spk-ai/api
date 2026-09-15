# Prepared Workload Contract

Dependent proposal on the combined backend-identity API (`ad5405b`). This is not
a published capability or a drop-in platform upgrade. The dependent registry
contract below adds persistence commands; controller migration is still required.

## Lifecycle

1. Reserve the task's execution in the durable control plane. Select and verify
   its backend. Supply every existing immutable volume binding to
   `PrepareWorkload.expected_volumes`; omission is permitted only for a volume
   proven to be first-provision, never for a lost workspace.
2. `PrepareWorkload` creates an unschedulable workload and returns a
   `WorkloadBinding`: backend, workload ID, native workload UID and every named
   volume's immutable identity. No init, sidecar or main code runs yet.
3. Compare the returned identities with the registry, bind new volumes using the
   checked-volume contract, and persist the exact workload binding **before**
   sending `ActivateWorkload`. The binding must be immutable and admission must
   exclude concurrent removal/predecessor execution.
4. `ActivateWorkload` protects the bound claims and enables only the exact native
   workload incarnation. Repeating this same activation can acknowledge an
   already-active incarnation; it never creates a new workload or replays an
   agent message. Uncertain preparation is not permission to prepare again.
5. `RemovePreparedWorkload` uses the durable binding, never a replacement fetched
   by name. PENDING is not absence. ABSENT attests native workload absence in the
   stated backend; it is not node/storage fencing or Secret/PVC deletion. PVCs
   remain task-owned. Their eventual deletion still uses `RemoveVolumeBound`.

These are distinct RPC capabilities: old servers return Unimplemented rather
than dropping newly added preconditions on `StartWorkload`. Callers must never
fall back to the legacy lifecycle methods. Existing methods remain in the wire
schema; all writers need migration before the new semantics may be trusted.

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

## Registry Contract

`CreatePreparedWorkload` reserves a STARTING workload with immutable backend and
registry volume IDs. It does not authorize native preparation. Only a successful
revision-checked `UpdatePreparedWorkload(begin_preparation)` permits that call.
Bind the checked volumes, persist the exact returned workload binding, then CAS
`begin_activation` before invoking native activation. Native status and billing
fields cannot replace these transitions.

The lifecycle is RESERVED -> PREPARING -> BOUND -> ACTIVATING -> ACTIVE, then
REMOVING -> REMOVED. Any post-reservation nonterminal phase may begin removal.
A RESERVED record may instead abort because preparation has not been authorized.
PREPARING cannot be aborted on a generic error: a delayed/lost native response
may hide a gated Pod. A late binding can be attached while REMOVING only for
cleanup, never to restore activation authorization. Removal requires the exact
binding and native ABSENT observation, both retained with the confirmation.

Read the exact record after a lost CAS reply; do not repeat native preparation
or replay agent messages. Caller authentication, native route enforcement,
late-operation reconciliation and node/storage fencing remain separate gates.
Cancellation after ACTIVATING cannot retract an already in-flight native call.
These registry methods are distinct capabilities with no legacy fallback.

## Lost Preparation Observation

`feat/prepared-outcome-observation` is an additive follow-up to the inspection
API `24b73ca`. It introduces `ObserveWorkloadPreparation` without changing the
registry schema or the existing prepare/activate/remove messages. Lint and
breaking-change checks against the inspection proposal pass.

The request carries the original durable workload intent and backend ID. The
native implementation may return only an unactivated, unscheduled, gated Pod
with no container-execution evidence and confirmed atomic startup-Secret
ownership semantics. The response contains its complete exact binding, bounded
owner/manager labels, Pod resource version, setup-complete flag and pending-
deletion flag. Neither boolean authorizes activation or retry.

The controller must first durably enter REMOVING, compare the observation with
its complete owner/volume intent, persist exact checked-volume and workload
bindings, then use the existing exact-binding removal contract. A late prepare
reply may supply the same binding into REMOVING; it cannot reopen execution.

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

An anchor is a metadata-only native owner with bounded identities. Workload
anchors own gated Pods; separate volume anchors own persistent workspaces.
Each Pod/PVC CREATE carries its owner's exact UID. Pod credentials remain
owned by that Pod UID. Workload/volume bindings and inventory retain the
anchors; callers must not discard them or fall back to unanchored APIs.

Before creating credentials, the runner pins one Pod UID on its workload
anchor. Before any activation writes, it records an activation claim on that
same anchor. Selection, activation and revocation use exact UID/revision
preconditions. Revocation either wins that conflict or must retire the exact
potentially activated Pod first, even if a gate PATCH has not completed.
Repeated activation of the same binding is not preparation or agent replay.

A delayed Pod CREATE after revocation retains the old owner UID and remains
gated. The caller must separately observe child garbage collection; anchor
ABSENT is not workload/credential absence. A replacement same-name anchor is
not the original authority. Delayed PVC creation retains the separate volume
anchor and does not justify deleting the workspace.

Workload-anchor retirement cannot delete volume anchors. Anchored volume
retirement needs a distinct checked contract; the old `RemoveVolumeBound`
method must reject an explicitly anchored target even if its PVC is absent.
Registry persistence, all-writer guards, delayed hold reconciliation, legacy
adoption, authenticated owner/backend routes, node/storage fencing and a
coordinated rollout remain required. This is not a drop-in or production API.

## References

[Scheduling readiness](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-scheduling-readiness/)
defines gates and their stable version. [API concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)
describe strict validation and resource versions. [Finalizers](https://kubernetes.io/docs/concepts/overview/working-with-objects/finalizers/)
retain deleting objects until the responsible controller removes its key.
