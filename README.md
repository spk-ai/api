# agynio/api

Agyn API contracts (IDL) repository.

The checked-volume lifecycle on this branch is a coordinated contract proposal,
not a released capability. See [CHECKED-VOLUMES.md](CHECKED-VOLUMES.md).

## Protobuf layout
Protobuf sources live under:

- `proto/<name>/<version>/*.proto`

Example:
- `proto/runner/v1/runner.proto`

## Buf / BSR
We use **Buf** for linting and codegen orchestration, and publish this module to **Buf Schema Registry (BSR)**.
