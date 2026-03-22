# Crypto Fixture Schema

PACT crypto fixtures provide deterministic wire-level examples shared across implementations.

Each fixture file contains:

- `name`
- `configString`
- `secret`
- `plaintext`
- `ciphertext`
- `deterministicInputs`

These fixtures currently cover the standardized `pact-psk1`, `pact-psk2`, and `pact-box1` wire behavior.

`deterministicInputs` contains profile-specific values:

- `ivBase64Url` for `pact-psk1`
- `ivBase64Url` for `pact-psk2`
- optional `saltBase64Url` for passphrase-based profiles
- `payloadIvBase64Url` for `pact-box1`
- `payloadKeyBase64Url` for `pact-box1`
- `ephemeralPrivateKeyBase64Url` for `pact-box1`

When a config fixture includes `transportData.charRemap`, the expected ciphertext already reflects that remapping.

Conformance expectations:

- decrypting `ciphertext` with `configString` and `secret` yields `plaintext`
- a deterministic encrypt operation using the provided IV and optional salt yields exactly `ciphertext`
- the implementation should recognize `ciphertext` as a valid encrypted payload for that config
