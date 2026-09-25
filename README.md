# agynio/api

Agyn API contracts (IDL) repository.

See [AGENTS.md](AGENTS.md) for source owners and contribution rules, and
[docs/catalog.json](docs/catalog.json) for operational and historical documents.

The checked-volume lifecycle on this branch is a coordinated contract proposal,
not a released capability. See [CHECKED-VOLUMES.md](CHECKED-VOLUMES.md).

The dependent [anchored retirement contract](ANCHORED-VOLUME-RETIREMENT.md)
separates explicit workspace deletion from idle compute release.

See [preparation revocation](PREPARATION-REVOCATION.md) and
[existing volume adoption](VOLUME-ANCHOR-ADOPTION.md) for rollout limits.

## Protobuf layout
Browse the [protobuf sources](proto/agynio/api) for service and message contracts.

## Buf / BSR
We use **Buf** for linting and codegen orchestration, and publish this module to **Buf Schema Registry (BSR)**.
Use [buf.yaml](buf.yaml) for the module and lint policy. Each consumer owns its
generation template; a schema publication alone does not upgrade deployed clients.

## Workload Removal Confirmation

See [the registry protobuf](proto/agynio/api/runners/v1/runners.proto) for the
removal-confirmation contract.

Deploy the additive Runners migration and regenerate the Runners service,
orchestrator and Gateway before clients rely on the JSON field. Existing
metering consumers keep using `removed_at`; never backfill confirmation from it.

## Runner compute resources

See [the native protobuf](proto/agynio/api/runner/v1/runner.proto) for the
compute-resource contract.

Publish this additive contract before deploying its runner and orchestrator
consumers. Enable profiles only after a configured runner advertises the
capability.

```bash
buf lint
buf breaking --against '.git#branch=main'
```
