# Preparation Revocation

Dependent, unreleased extension of the resource-anchor contract. It addresses
an authorized preparation whose Pod/PVC creation response was never recorded.
There is no change to A2A task routing, agent profiles or workflow code.

## Contract Owners

See the [native proof](proto/agynio/api/runner/v1/runner.proto) and
[registry confirmation](proto/agynio/api/runners/v1/runners.proto) contracts.

## Boundaries

Current absence is not future-write exclusion or node-partition fencing.
Interrupted agent side effects still need reconciliation; this is not retry
authorization.

Coordinate registry migration 0026, native runner, controller, regenerated
consumers and all writers before enabling this capability. These sources are a
proposal, not a published BSR module or an installed production upgrade. Receipt
retention, authenticated future-write fencing and hardened deployment remain
separate requirements. Existing repository licensing is unchanged.

Validation on 2026-09-15: `buf lint` and `buf breaking --against '.git#ref=HEAD'`
passed against the preceding anchored-retirement contract.
