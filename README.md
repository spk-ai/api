# agynio/api

Agyn API contracts (IDL) repository.

The checked-volume lifecycle on this branch is a coordinated contract proposal,
not a released capability. See [CHECKED-VOLUMES.md](CHECKED-VOLUMES.md).

The dependent [anchored retirement contract](ANCHORED-VOLUME-RETIREMENT.md)
separates explicit workspace deletion from idle compute release.

## Protobuf layout
Protobuf sources live under:

- `proto/<name>/<version>/*.proto`

Example:
- `proto/runner/v1/runner.proto`

## Buf / BSR
We use **Buf** for linting and codegen orchestration, and publish this module to **Buf Schema Registry (BSR)**.

## Workload Removal Confirmation

`Workload.removal_confirmed_at` separates lifecycle-confirmed absence from
`removed_at`, which ends the metered lifetime and can be set by a terminal
status report. A stop acknowledgement, failed status or historical billing
timestamp does not populate the new field. Existing records remain unverified.

`UpdateWorkloadRequest.removal_confirmed_at` is an explicit internal lifecycle
write for stopped/failed workloads. The Runners service retains the first
confirmation; runner state reports cannot supply it. Consumers must fail closed
when it is absent. This is runner-observed absence, not node-partition fencing
or a guarantee against delayed workload creation.

Deploy the additive Runners migration and regenerate the Runners service,
orchestrator and Gateway before clients rely on the JSON field. Existing
metering consumers keep using `removed_at`; never backfill confirmation from it.

## Runner compute resources

`runner.v1.ContainerSpec.resources` adds typed CPU/memory requests and limits.
The required `compute-resources` capability is the compatibility guard: clients
must request it, and runners must reject unsupported capabilities rather than
silently ignoring unknown resource fields. All four fields must be valid positive
quantities, requests must not exceed limits, and runners reject values they
cannot enforce.

Opted-in workloads require explicit main bounds. Supporting containers may omit
the entire message only when the runner provides complete operator-configured
bounds, including for containers the runner injects. Explicit empty/partial
messages are invalid. These are per-container allocations, not one task budget.

Publish this additive contract before deploying its runner and orchestrator
consumers. Enable profiles only after a configured runner advertises the
capability. No existing field numbers or legacy workloads are changed.

```bash
buf lint
buf breaking --against '.git#branch=main'
```
