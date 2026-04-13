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

For config-bound fixtures, `ciphertext` begins with the configured `messagePrefix` token serialized inside brackets. For self-describing message fixtures, `ciphertext` is the full `[pact]:v1:<profile-id>:<remap-spec>:<encoded-payload>` message. The `configString` remains available as deterministic encryption context, especially for profiles such as `pact-box1` that still need recipient public keys when encrypting.

Conformance expectations:

- for config-bound fixtures, decrypting `ciphertext` with `configString` and `secret` yields `plaintext`
- a deterministic encrypt operation using the provided IV and optional salt yields exactly `ciphertext`
- the implementation should recognize config-bound `ciphertext` as a valid encrypted payload for that config
- decrypting a self-describing `ciphertext` with the local matching secret material yields `plaintext` without requiring the receiver to be given `configString`

Auto-detection expectations:

- `pact-psk1`: after inverse remap and compact payload decoding, a candidate shared secret matches only if the profile's authenticated decrypt operation succeeds
- `pact-psk2`: after inverse remap and standard-Base64 decoding, a candidate shared secret matches only if AES-GCM tag verification succeeds
- `pact-box1`: after inverse remap and envelope parsing, a candidate private key matches only if one `wrappedKey` unwrap authenticates and the payload AES-GCM decrypt authenticates
- implementations should not use plaintext readability, language detection, or UTF-8 decoding alone as the success condition
