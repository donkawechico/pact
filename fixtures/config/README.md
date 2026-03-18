# Config Fixture Schema

PACT config conformance fixtures are split into `valid/` and `invalid/`.

## Valid fixtures

Required fields:

- `name`
- `json`
- `canonicalString`
- `expectedNormalized`

`expectedNormalized` uses implementation-facing enum names for easier automated assertions:

- `keyHandling`: `PASSPHRASE_PBKDF2` or `RAW_BASE64_KEY`
- `payloadLayout`: `MULTIPART` or `PACKED`
- `packedEncoding`: `URL_SAFE_NO_PADDING` or `STANDARD_NO_PADDING`

## Invalid fixtures

Required fields:

- `name`
- either `json` or `pactString`
- `expectedErrorContains`

Implementations should fail parsing or validation and surface an error containing the expected substring.
