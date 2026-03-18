# Crypto Fixture Schema

PACT crypto fixtures provide deterministic wire-level examples shared across implementations.

Each fixture file contains:

- `name`
- `configString`
- `secret`
- `plaintext`
- `ciphertext`
- `deterministicInputs`

`deterministicInputs` contains:

- `ivBase64Url`
- optional `saltBase64Url`

Conformance expectations:

- decrypting `ciphertext` with `configString` and `secret` yields `plaintext`
- a deterministic encrypt operation using the provided IV and optional salt yields exactly `ciphertext`
- the implementation should recognize `ciphertext` as a valid encrypted payload for that config
