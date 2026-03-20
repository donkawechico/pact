# Config Fixture Schema

PACT config conformance fixtures are split into `valid/` and `invalid/`.

## Valid fixtures

Required fields:

- `name`
- `json`
- `canonicalString`
- `expectedNormalized`

`expectedNormalized` uses implementation-facing normalized names for easier automated assertions:

- `profile`: `PACT_PSK1` or `PACT_BOX1`
- `profileData`: normalized profile-specific public parameters

## Invalid fixtures

Required fields:

- `name`
- either `json` or `pactString`
- `expectedErrorContains`

Implementations should fail parsing or validation and surface an error containing the expected substring.
