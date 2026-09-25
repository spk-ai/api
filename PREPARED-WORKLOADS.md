# Prepared Workload Contract

Dependent proposal on the combined backend-identity API (`ad5405b`). This is not
a published capability or a drop-in platform upgrade. The dependent registry
contract below adds persistence commands; controller migration is still required.
Dependency milestones and remaining-work lists below retain their original
proposal scope, not a current deployment compatibility claim.

## Contract Owners

See the [native lifecycle](proto/agynio/api/runner/v1/runner.proto) and
[registry admission](proto/agynio/api/runners/v1/runners.proto) contracts.

## Kubernetes Implementation

The prototype requires Kubernetes >=1.30, strict Pod-create field validation,
an intact trusted admission chain and scheduling-gate-aware schedulers.
Secret garbage collection and interrupted-preparation reconciliation remain
separate operational obligations from native workload absence.

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

## Lost Preparation Observation

`feat/prepared-outcome-observation` is an additive follow-up to the inspection
API `24b73ca`. It introduces `ObserveWorkloadPreparation` without changing the
registry schema or the existing prepare/activate/remove messages. Lint and
breaking-change checks against the inspection proposal pass.

The observation contract is in [runner.proto](proto/agynio/api/runner/v1/runner.proto).
Initially absent or delayed Pod/PVC creates, old ownerless Secrets, external
credential revocation, authenticated routes, all-writer
enforcement and node/storage fencing still require separate mechanisms.

## Resource Anchors

The original `feat/resource-anchors` proposal depends on observation API
`d6449dd`; that API-only contribution did not implement registry/controller
integration. The source owners above define the anchor contracts.
Registry persistence, all-writer guards, delayed hold reconciliation, legacy
adoption, authenticated owner/backend routes, node/storage fencing and a
coordinated rollout remain required. This is not a drop-in or production API.

## References

[Scheduling readiness](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-scheduling-readiness/)
defines gates and their stable version. [API concepts](https://kubernetes.io/docs/reference/using-api/api-concepts/)
describe strict validation and resource versions. [Finalizers](https://kubernetes.io/docs/concepts/overview/working-with-objects/finalizers/)
retain deleting objects until the responsible controller removes its key.
