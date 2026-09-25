# agynio/api

Agyn API contracts (IDL) repository.

See [AGENTS.md](AGENTS.md) for source owners and contribution rules, and
[docs/catalog.json](docs/catalog.json) for operational and historical documents.

The checked-volume lifecycle on this branch is a coordinated contract proposal,
not a released capability. See [CHECKED-VOLUMES.md](CHECKED-VOLUMES.md).

The dependent [anchored retirement contract](ANCHORED-VOLUME-RETIREMENT.md)
separates explicit workspace deletion from idle compute release.

[Preparation revocation](PREPARATION-REVOCATION.md) adds durable recovery for
interrupted, unbound provisioning without replaying execution.

[Existing volume adoption](VOLUME-ANCHOR-ADOPTION.md) adds an explicit native
migration receipt for original PVCs, distinct from first allocation.

## Protobuf layout
Protobuf sources live under:

- `proto/<name>/<version>/*.proto`

Example:
- `proto/runner/v1/runner.proto`

## Buf / BSR
We use **Buf** for linting and codegen orchestration, and publish this module to **Buf Schema Registry (BSR)**.

## Workload Removal Confirmation

Field semantics live beside `Workload` and `UpdateWorkloadRequest` in
[the registry protobuf](proto/agynio/api/runners/v1/runners.proto).
Billing end is not physical-removal evidence.

Deploy the additive Runners migration and regenerate the Runners service,
orchestrator and Gateway before clients rely on the JSON field. Existing
metering consumers keep using `removed_at`; never backfill confirmation from it.

## Runner compute resources

The typed quantity, capability and per-container allocation contract lives beside
`ComputeResources` and `ContainerSpec.resources` in
[the native protobuf](proto/agynio/api/runner/v1/runner.proto).

Publish this additive contract before deploying its runner and orchestrator
consumers. Enable profiles only after a configured runner advertises the
capability. No existing field numbers or legacy workloads are changed.

```bash
buf lint
buf breaking --against '.git#branch=main'
```
