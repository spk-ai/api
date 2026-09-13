# agynio/api

Agyn API contracts (IDL) repository.

## Protobuf layout
Protobuf sources live under:

- `proto/<name>/<version>/*.proto`

Example:
- `proto/runner/v1/runner.proto`

## Buf / BSR
We use **Buf** for linting and codegen orchestration, and publish this module to **Buf Schema Registry (BSR)**.

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
