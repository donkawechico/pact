# PACT v1 Specification Draft

PACT stands for **Portable Application-layer Cryptography Template**.

## 1. Purpose

PACT defines a portable configuration template for interoperable application-layer cryptography over host applications that are not natively aware of the encryption protocol in use.

A PACT template describes:

- message prefixing
- key handling mode
- payload layout
- payload encoding
- optional character remapping
- optional cryptographic metadata

PACT templates MUST NOT contain secret key material.

## 2. Terminology

- **Host application**: the app through which encrypted payloads are sent.
- **PACT-compatible application**: an app that can parse or emit PACT templates.
- **Template**: a structured, shareable protocol description.
- **Runtime config**: the normalized executable interpretation of a PACT template.

## 3. Canonical String Format

PACT v1 strings use the following wrapper:

`pact:v1:<base64url(json)>`

Where:

- `pact` is the fixed scheme tag.
- `v1` is the protocol version.
- `<base64url(json)>` is an unpadded Base64URL-encoded UTF-8 JSON object.

Implementations MUST reject strings with an unknown scheme or version.

## 4. JSON Body

The decoded JSON object MAY contain unknown fields. Unknown fields MUST be ignored by parsers that do not understand them.

### 4.1 Required fields

- `messagePrefix`: string
- `keyHandling`: string enum
- `payloadLayout`: string enum

### 4.2 Optional fields

- `multipartSeparator`: string
- `packedEncoding`: string enum
- `charRemap`: object mapping one-character strings to one-character strings
- `crypto`: object

### 4.3 Enumerations

#### `keyHandling`

- `passphrase-pbkdf2`
- `raw-base64-key`

#### `payloadLayout`

- `multipart`
- `packed`

#### `packedEncoding`

- `base64url-no-padding`
- `base64-standard-no-padding`
- `dot-bang-base64-no-padding`

`dot-bang-base64-no-padding` is equivalent to standard Base64 without padding plus the outbound remap `{"+":".","/":"!"}`.

## 5. Cryptographic Metadata

The `crypto` object is optional and reserved for executable cryptographic details.

Known `crypto` fields in v1:

- `algorithm`: string, for example `aes-256-gcm`
- `ivBytes`: integer
- `tagBits`: integer
- `kdf`: object

Known `kdf` fields in v1:

- `type`: string, for example `pbkdf2-hmac-sha256`
- `iterations`: integer
- `saltBytes`: integer

Unknown `crypto` and `kdf` fields MUST be preserved when possible during round-trip serialization.

## 6. Secret Handling

PACT strings MUST NOT embed:

- passphrases
- raw AES keys
- private keys
- tokens
- any other secret material

Secret exchange is explicitly out of scope.

## 7. Character Remapping

If `charRemap` is present, it defines an outbound character substitution map applied after encoding.

Decoders MUST invert that map when decoding inbound ciphertext.

Example:

```json
{
  "+": ".",
  "/": "!"
}
```

## 8. Validation Rules

Implementations MUST reject templates when:

- required fields are missing
- required fields are of the wrong type
- enum values are unknown
- `charRemap` keys or values are not one character long
- `payloadLayout` is `multipart` and `multipartSeparator` is absent
- `payloadLayout` is `packed` and `packedEncoding` is absent

## 9. Forward Compatibility

PACT parsers MUST:

- reject unknown versions
- ignore unknown top-level fields within a known version
- preserve unknown fields when feasible during parse/serialize round-trips

## 10. Canonical Serialization Notes

PACT v1 implementations SHOULD emit canonical strings using:

- the `pact:v1:` wrapper
- unpadded Base64URL for the outer body encoding
- the spec enum spellings from this document
- `dot-bang-base64-no-padding` when standard Base64 encoding is paired with the exact remap `{"+":".","/":"!"}`

## 11. Conformance Fixtures

The `fixtures/` directory is the machine-readable conformance contract for independent implementations.

### 11.1 Config fixtures

`fixtures/config/valid/*.json` contain:

- `name`
- `json`: a decoded PACT JSON body
- `canonicalString`: the expected canonical `pact:v1:...` string
- `expectedNormalized`: the normalized runtime interpretation

`fixtures/config/invalid/*.json` contain:

- `name`
- either `json` or `pactString`
- `expectedErrorContains`

Implementations SHOULD run these fixtures in automated tests.

### 11.2 Crypto fixtures

`fixtures/crypto/` is reserved for deterministic encryption and decryption vectors shared across implementations.

## 12. Example

Decoded JSON:

```json
{
  "messagePrefix": "[ENC]",
  "keyHandling": "raw-base64-key",
  "payloadLayout": "packed",
  "packedEncoding": "dot-bang-base64-no-padding",
  "crypto": {
    "algorithm": "aes-256-gcm",
    "ivBytes": 12,
    "tagBits": 128
  }
}
```

Canonical string form is the Base64URL-encoded representation of that JSON wrapped as `pact:v1:...`.
