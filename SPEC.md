# PACT v1 Specification Draft

PACT stands for **Portable Application-layer Cryptography Template**.

## 1. Purpose

PACT v1 defines a portable, shareable configuration string for interoperable application-layer cryptography over host applications that are not natively aware of the encrypted content.

PACT v1 is **profile-first**:

- a PACT string identifies a **named protocol profile**
- a profile pins a concrete interoperability contract
- a config string may carry only the **non-secret public parameters** that profile needs

PACT strings MUST NOT contain secret key material.

## 2. Design Model

PACT intentionally separates:

- **profile selection**: which interoperable cryptographic contract is in use
- **public configuration**: non-secret data needed by that contract
- **secret material**: passphrases, symmetric keys, private keys, or tokens exchanged out of band

PACT v1 does **not** standardize UI, trust establishment, or transport security.

## 3. Terminology

- **Host application**: the app through which encrypted payloads are sent.
- **PACT-compatible application**: an app that can parse or emit PACT config strings.
- **Profile**: a named, fixed interoperability contract.
- **Profile data**: non-secret public configuration required by a specific profile.
- **Runtime config**: the normalized executable interpretation of a PACT string.

## 4. Canonical String Format

PACT v1 strings use the following wrapper:

`pact:v1:<base64url(json)>`

Where:

- `pact` is the fixed scheme tag
- `v1` is the protocol version
- `<base64url(json)>` is an unpadded Base64URL-encoded UTF-8 JSON object

Implementations MUST reject strings with an unknown scheme or version.

## 5. JSON Body

The decoded JSON object MAY contain unknown fields. Unknown fields MUST be ignored by parsers that do not understand them.

### 5.1 Required fields

- `messagePrefix`: string
- `profile`: string enum

### 5.2 Optional fields

- `profileData`: object

### 5.3 Standard profile names

- `pact-psk1`
- `pact-box1`

Implementations MUST reject unknown `profile` values.

## 6. Standard Profiles

### 6.1 `pact-psk1`

`pact-psk1` is the shared-secret profile.

Intended use:

- pairwise messaging where both sides already share a secret
- group messaging where multiple recipients share the same secret
- compact local-first encryption overlays

Security model:

- one symmetric secret is shared out of band
- every holder of that secret can decrypt the same message

`pact-psk1` does not require `profileData`.
If `profileData` is present, it MUST be an empty object.

### 6.2 `pact-box1`

`pact-box1` is the recipient-public-key envelope profile.

Intended use:

- direct 1:1 encryption using recipient public keys
- small-group messaging where one message is decryptable by multiple recipients

Security model:

- the sender encrypts one message for one or more recipients
- each recipient is described by non-secret public key material in the config

`pact-box1` requires `profileData.recipients`.

`profileData.recipients` MUST be a non-empty array of recipient objects.

Each recipient object MUST contain:

- `keyId`: string
- `publicKey`: string

`keyId` is an application-level recipient identifier.
`publicKey` is the profile-defined textual public-key representation.

Unknown recipient object fields MAY be preserved by implementations when feasible.

## 7. Secret Handling

PACT strings MUST NOT embed:

- passphrases
- raw symmetric keys
- private keys
- tokens
- any other secret material

Secret exchange is explicitly out of scope.

## 8. Validation Rules

Implementations MUST reject configs when:

- required fields are missing
- required fields are of the wrong type
- `profile` is unknown
- `profileData` is present but is not an object
- `profile` is `pact-psk1` and `profileData` is non-empty
- `profile` is `pact-box1` and `profileData.recipients` is missing
- `profile` is `pact-box1` and `profileData.recipients` is not a non-empty array
- any `pact-box1` recipient is missing `keyId` or `publicKey`
- any required profile field is of the wrong type

## 9. Forward Compatibility

PACT parsers MUST:

- reject unknown versions
- ignore unknown top-level fields within a known version
- preserve unknown fields when feasible during parse/serialize round-trips

## 10. Canonical Serialization Notes

PACT v1 implementations SHOULD emit canonical strings using:

- the `pact:v1:` wrapper
- unpadded Base64URL for the outer body encoding
- canonical key ordering of:
  - `messagePrefix`
  - `profile`
  - `profileData`
- no insignificant JSON whitespace

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

`fixtures/crypto/` contains deterministic encryption and decryption vectors for profiles whose wire behavior is already pinned tightly enough for cross-implementation comparison.

PACT v1 crypto fixtures currently cover `pact-psk1`.

## 12. Examples

### 12.1 Shared-secret config

```json
{
  "messagePrefix": "pact1:",
  "profile": "pact-psk1"
}
```

### 12.2 Recipient config

```json
{
  "messagePrefix": "pact1:",
  "profile": "pact-box1",
  "profileData": {
    "recipients": [
      {
        "keyId": "alice-main",
        "publicKey": "MDEyMzQ1Njc4OWFiY2RlZjAxMjM0NTY3ODlhYmNkZWY"
      }
    ]
  }
}
```
