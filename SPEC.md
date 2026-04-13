# PACT v1 Specification Draft

PACT stands for **Portable Application-layer Cryptography Template**.

## 1. Purpose

PACT v1 defines portable, interoperable application-layer cryptography over host applications that are not natively aware of the encrypted content.

PACT v1 is **profile-first**:

- a PACT config string identifies a **named protocol profile**
- a self-describing PACT message identifies a profile by compact stable ID
- a profile pins a concrete interoperability contract
- a config string may carry only the **non-secret public parameters** that profile needs
- a self-describing message may carry only the non-secret public parameters needed to recognize and decode that message

PACT strings MUST NOT contain secret key material.

## 2. Design Model

PACT intentionally separates:

- **profile selection**: which interoperable cryptographic contract is in use
- **public configuration**: non-secret data needed by that contract
- **transport adaptation**: optional text-surface adjustments layered on top of a profile
- **secret material**: passphrases, symmetric keys, private keys, or tokens exchanged out of band

PACT v1 does **not** standardize UI, trust establishment, or transport security.

## 3. Terminology

- **Host application**: the app through which encrypted payloads are sent.
- **PACT-compatible application**: an app that can parse or emit PACT config strings or self-describing messages.
- **Profile**: a named, fixed interoperability contract.
- **Profile data**: non-secret public configuration required by a specific profile.
- **Runtime config**: the normalized executable interpretation of a PACT string.
- **Self-describing message**: an encrypted message with a PACT preamble containing compact non-secret decoding metadata.

## 4. String Formats

PACT v1 defines two additive string formats:

- config strings, used to share non-secret encryption settings
- self-describing encrypted messages, used to allow PACT-compatible clients to detect and decode encrypted messages without a separate config string

PACT v1 config strings use the following wrapper:

`pact:v1:<base64url(json)>`

Where:

- `pact` is the fixed scheme tag
- `v1` is the protocol version
- `<base64url(json)>` is an unpadded Base64URL-encoded UTF-8 JSON object

Implementations MUST reject strings with an unknown scheme or version.

### 4.1 Self-Describing Encrypted Message Format

Self-describing encrypted messages use the following preamble:

`[pact]:v1:<profile-id>:<remap-spec>:<encoded-payload>`

Where:

- `pact` is the fixed self-describing message prefix token, serialized as the bracketed preamble tag `[pact]`
- `v1` is the protocol version
- `<profile-id>` is a compact profile ID from the v1 profile ID registry
- `<remap-spec>` is a compact character-remap specification
- `<encoded-payload>` is the profile-defined ciphertext payload after transport remapping

Implementations can auto-detect self-describing PACT messages by scanning for the literal prefix `[pact]:v1:`.

Parsers MUST treat the first four `:` characters as preamble delimiters and MUST treat the rest of the string as `<encoded-payload>`. This allows profile payload alphabets to contain `:` without requiring payload escaping.

The `<encoded-payload>` omits the config string's bracketed message prefix. For example, if the config-bound profile wire format would emit `[ENC]AA...`, the self-describing message payload segment contains only `AA...`.

### 4.2 Compact Profile ID Registry

PACT v1 defines the following compact profile IDs:

- `1`: `pact-psk1`
- `2`: `pact-psk2`
- `3`: `pact-box1`

Implementations MUST reject unknown profile IDs.

### 4.3 Compact Remap Specification

The `<remap-spec>` segment is either empty or a concatenation of 3-character remap entries.

Each remap entry contains:

- two uppercase hexadecimal digits encoding the source character's ASCII code point
- one literal destination character

Example:

`2B.2F!`

This decodes to:

```json
{
  "+": ".",
  "/": "!"
}
```

Rules:

- an empty `<remap-spec>` means no transport remapping
- source hex digits MUST use uppercase `0`-`9` and `A`-`F`
- each source character MUST be unique
- each destination character MUST be unique
- destination characters MUST NOT be `:`
- source characters MUST belong to the selected profile's encoded payload alphabet

Because `:` is the segment delimiter, an attempted `:` destination can surface as a malformed remap spec during parsing.

### 4.4 Auto-Detection Decryption Success

When a client auto-detects `[pact]:v1:`, it MAY try locally configured secret material that is compatible with the compact profile ID. A candidate secret succeeds only when the selected profile's authenticated cryptographic operation succeeds. Implementations MUST NOT treat payload decoding, UTF-8 decoding, language detection, or plaintext readability as proof that a secret worked.

Recommended profile-specific success conditions:

- `pact-psk1` (`profile-id` `1`): inverse-apply the compact remap, decode the compact profile payload, and try each local shared secret using the profile-defined authenticated decrypt operation. Success is authenticated decrypt returning plaintext bytes. Payload decoding alone is not success.
- `pact-psk2` (`profile-id` `2`): inverse-apply the compact remap, standard-Base64 decode the payload, split the decoded bytes according to the profile wire format, and try each local shared secret as the AES-256-GCM key. Success is AES-GCM tag verification and plaintext bytes returned by the decrypt operation.
- `pact-box1` (`profile-id` `3`): inverse-apply the compact remap, Base64URL-decode and parse the payload JSON, then try local recipient private keys against the payload recipient entries. Success requires an authenticated `wrappedKey` unwrap to produce the payload key and a second authenticated AES-GCM decrypt of the payload ciphertext.

Illustrative pseudocode:

```text
detectAndDecrypt(message, localSecrets):
  preamble = parseSelfDescribingPreamble(message)
  payload = inverseApplyRemap(preamble.encodedPayload, preamble.remap)

  if preamble.profileId == "1":
    bytes = ascii85Decode(payload)
    for secret in localSecrets.sharedPsk1Secrets:
      plaintext = tryProfilePsk1AuthenticatedDecrypt(bytes, secret)
      if plaintext.authenticated:
        return plaintext.bytes

  if preamble.profileId == "2":
    bytes = standardBase64Decode(payload)
    iv, ciphertextAndTag = splitPsk2Payload(bytes)
    for secret in localSecrets.sharedPsk2Secrets:
      plaintext = aesGcmDecryptOrFail(secret, iv, ciphertextAndTag)
      if plaintext.authenticated:
        return plaintext.bytes

  if preamble.profileId == "3":
    envelope = parseBox1Payload(payload)
    for privateKey in localSecrets.box1PrivateKeys:
      for recipient in envelope.recipients:
        payloadKey = tryAuthenticatedUnwrap(privateKey, envelope.ephemeralPublicKey, recipient.wrappedKey)
        if payloadKey.authenticated:
          plaintext = aesGcmDecryptOrFail(payloadKey.bytes, envelope.payloadIv, envelope.ciphertext)
          if plaintext.authenticated:
            return plaintext.bytes

  return noMatchingSecret
```

## 5. JSON Config Body

The decoded JSON object MAY contain unknown fields. Unknown fields MUST be ignored by parsers that do not understand them.

### 5.1 Required fields

- `messagePrefix`: string
- `profile`: string enum

`messagePrefix` is the application-chosen prefix token. Configs may choose any non-empty token that does not contain bracket characters. Config-bound encrypted messages serialize that token inside brackets before the profile-defined payload, for example `ENC` becomes `[ENC]`.

### 5.2 Optional fields

- `profileData`: object
- `transportData`: object

### 5.3 Standard profile names

- `pact-psk1`
- `pact-psk2`
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

### 6.2 `pact-psk2`

`pact-psk2` is the shared-secret inline-base64 profile.

Intended use:

- pairwise messaging where both sides already share a secret
- group messaging where multiple recipients share the same secret
- host applications where a conservative base64-derived alphabet is preferred over ASCII85

Security model:

- one symmetric secret is shared out of band
- every holder of that secret can decrypt the same message

`pact-psk2` does not require `profileData`.
If `profileData` is present, it MUST be an empty object.

`pact-psk2` pins the following cryptographic behavior:

- secret type: raw 32-byte AES key encoded as unpadded Base64URL
- payload encryption: AES-256-GCM
- IV: random 12 bytes
- authentication tag: full 16-byte GCM tag
- payload bytes before text encoding: `iv || ciphertext`
- text encoding: unpadded standard Base64
- wire format: `[<messagePrefix>]<encodedPayload>`

### 6.3 `pact-box1`

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

`pact-box1` defines `publicKey` as:

- the raw 32-byte X25519 public key
- encoded using unpadded Base64URL

The corresponding private key format used by implementations is:

- the raw 32-byte X25519 private key
- encoded using unpadded Base64URL

### 6.4 `pact-box1` Wire Format

`pact-box1` ciphertexts are encoded as:

`[<messagePrefix>]<base64url(json)>`

Where the decoded JSON payload uses the canonical key order:

- `profile`
- `ephemeralPublicKey`
- `payloadIv`
- `recipients`
- `ciphertext`

The payload JSON MUST contain:

- `profile`: the fixed string `pact-box1`
- `ephemeralPublicKey`: sender ephemeral X25519 public key, encoded as unpadded Base64URL raw 32 bytes
- `payloadIv`: AES-GCM IV for the shared payload ciphertext, encoded as unpadded Base64URL raw 12 bytes
- `recipients`: non-empty array
- `ciphertext`: AES-GCM payload ciphertext bytes, encoded as unpadded Base64URL

Each payload recipient object MUST use canonical key order:

- `keyId`
- `wrappedKey`

Each payload recipient object MUST contain:

- `keyId`: string
- `wrappedKey`: wrapped payload key bytes encoded as unpadded Base64URL

The `recipients` array order in the ciphertext payload MUST match the `profileData.recipients` order from the config used to encrypt it.

### 6.5 `pact-box1` Cryptographic Contract

`pact-box1` pins the following cryptographic behavior:

- recipient key agreement: X25519
- payload encryption: AES-256-GCM
- payload content key: random 32 bytes
- payload IV: random 12 bytes
- wrapped payload-key encryption: AES-256-GCM
- key derivation for each recipient wrap: HKDF-SHA256

Encryption works as follows:

1. generate a random 32-byte payload content key
2. generate a random 12-byte payload IV
3. encrypt the plaintext with AES-256-GCM using the payload key and payload IV
4. generate a fresh ephemeral X25519 key pair for the message
5. for each recipient:
   - derive a 32-byte X25519 shared secret using the sender ephemeral private key and the recipient public key
   - run HKDF-SHA256 over that shared secret with:
     - salt: empty
     - info: UTF-8 string `pact-box1-wrap`
     - output length: 44 bytes
   - split the HKDF output into:
     - first 32 bytes: AES-256-GCM wrap key
     - next 12 bytes: AES-GCM wrap IV
   - wrap the 32-byte payload content key using AES-256-GCM with the derived wrap key and wrap IV
6. emit one payload recipient entry per config recipient, preserving config order

Decryption works as follows:

1. parse the outer payload JSON
2. for each payload recipient entry:
   - derive the same X25519 shared secret using the recipient private key and the payload `ephemeralPublicKey`
   - derive the same 44-byte HKDF output using `pact-box1-wrap`
   - attempt to decrypt `wrappedKey`
3. the first successful unwrap yields the 32-byte payload key
4. decrypt `ciphertext` using AES-256-GCM and `payloadIv`

If no `wrappedKey` entry can be decrypted with the provided private key, decryption MUST fail.

## 7. Secret Handling

PACT strings MUST NOT embed:

- passphrases
- raw symmetric keys
- private keys
- tokens
- any other secret material

Secret exchange is explicitly out of scope.

## 8. Transport Data

PACT v1 optionally supports transport-layer adaptations that are not part of profile selection.

### 8.1 `transportData.charRemap`

`transportData.charRemap` MAY be present as an object mapping one-character strings to one-character strings.

Intended use:

- adapting encrypted payload text to host applications with awkward character handling
- preserving a fixed profile while allowing text-surface customization on top

Rules:

- remapping is applied after profile-defined text encoding during encryption
- the inverse remapping is applied before profile-defined text decoding during decryption
- remapping does not change the underlying cryptographic profile
- remapping keys MUST be unique one-character strings
- remapping values MUST be unique one-character strings

Profiles MUST NOT redefine or require any particular `charRemap`.

## 9. Validation Rules

Implementations MUST reject configs when:

- required fields are missing
- required fields are of the wrong type
- `messagePrefix` is empty
- `messagePrefix` contains `[` or `]`
- `profile` is unknown
- `profileData` is present but is not an object
- `transportData` is present but is not an object
- `profile` is `pact-psk1` and `profileData` is non-empty
- `profile` is `pact-psk2` and `profileData` is non-empty
- `profile` is `pact-box1` and `profileData.recipients` is missing
- `profile` is `pact-box1` and `profileData.recipients` is not a non-empty array
- any `pact-box1` recipient is missing `keyId` or `publicKey`
- any required profile field is of the wrong type
- any `pact-box1` `publicKey` is not a valid unpadded Base64URL X25519 public key
- any `transportData.charRemap` key is not a one-character string
- any `transportData.charRemap` value is not a one-character string
- `transportData.charRemap` maps multiple source characters to the same destination character

Implementations MUST reject self-describing messages when:

- the message does not contain the four required preamble delimiters
- the preamble tag is not `[pact]`
- the version is unknown
- the compact profile ID is unknown
- the compact remap spec length is not a multiple of 3
- any compact remap source hex is malformed or not uppercase
- any compact remap source character is duplicated
- any compact remap destination character is duplicated
- any compact remap destination character is `:`
- any compact remap source character is outside the selected profile's encoded payload alphabet

## 10. Forward Compatibility

PACT parsers MUST:

- reject unknown versions
- reject unknown compact profile IDs in self-describing messages
- ignore unknown top-level fields within a known version
- preserve unknown fields when feasible during parse/serialize round-trips

## 11. Canonical Serialization Notes

PACT v1 implementations SHOULD emit canonical strings using:

- the `pact:v1:` wrapper
- unpadded Base64URL for the outer body encoding
- canonical key ordering of:
  - `messagePrefix`
  - `profile`
  - `profileData`
  - `transportData`
- no insignificant JSON whitespace

## 12. Conformance Fixtures

The `fixtures/` directory is the machine-readable conformance contract for independent implementations.

### 12.1 Config fixtures

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

### 12.2 Crypto fixtures

`fixtures/crypto/` contains deterministic encryption and decryption vectors for profiles whose wire behavior is already pinned tightly enough for cross-implementation comparison.

PACT v1 crypto fixtures currently cover `pact-psk1`, `pact-psk2`, and `pact-box1`.

Config-bound crypto fixtures include the bracketed `messagePrefix` before the encoded payload in their `ciphertext` field. Self-describing message fixtures use `[pact]:v1:<profile-id>:<remap-spec>:<encoded-payload>` in their `ciphertext` field. Self-describing message fixtures may still include `configString` as encryption context, especially for profiles such as `pact-box1` that require recipient public keys when encrypting.

### 12.3 Message fixtures

`fixtures/message/invalid/*.json` contain malformed self-describing PACT messages with:

- `name`
- `message`
- `expectedErrorContains`

## 13. Examples

### 13.1 Shared-secret config

```json
{
  "messagePrefix": "pact1",
  "profile": "pact-psk1"
}
```

### 13.2 Inline-base64 shared-secret config

```json
{
  "messagePrefix": "ENC",
  "profile": "pact-psk2"
}
```

### 13.3 Shared-secret config with transport remap

```json
{
  "messagePrefix": "ENC",
  "profile": "pact-psk2",
  "transportData": {
    "charRemap": {
      "+": ".",
      "/": "!"
    }
  }
}
```

### 13.4 Recipient config

```json
{
  "messagePrefix": "pact1",
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

### 13.5 `pact-box1` ciphertext payload shape

```json
{
  "profile": "pact-box1",
  "ephemeralPublicKey": "p2V4R1RzRWRfYmFzZTY0dXJsX2VwaGVtZXJhbF9wdWI",
  "payloadIv": "AAECAwQFBgcICQoL",
  "recipients": [
    {
      "keyId": "alice-main",
      "wrappedKey": "WmV4YW1wbGVfd3JhcHBlZF9rZXlfYnl0ZXM"
    }
  ],
  "ciphertext": "ZXhhbXBsZV9jaXBoZXJ0ZXh0X2J5dGVz"
}
```

### 13.6 Self-describing shared-secret message

```text
[pact]:v1:2::AAECAwQFBgcICQoLE2elb6yLpSq/coCH+6wswH/VbxSMiBMtC7k
```

### 13.7 Self-describing shared-secret message with remap

```text
[pact]:v1:2:2B.2F!:AAECAwQFBgcICQoLE2elb6yLpSq!coCH.6wswH!VbxSMiBMtC7k
```

## 14. Open Issues

### 14.1 Transport remap ambiguity

Non-normative note:

`transportData.charRemap` and compact preamble remaps validate single-character mappings with unique destination values, but that alone does not guarantee unambiguous inversion.

Example:

- an encoding alphabet already permits both `/` and `+`
- a transport remap maps `/` to `+`
- ciphertext that originally contained a literal `+` becomes ambiguous after inversion

Compact preamble remap destinations SHOULD NOT collide with the native encoded alphabet of the selected profile unless the resulting mapping remains unambiguous under that profile.

A future revision may tighten all `charRemap` validation so destination characters must not collide with the native encoded alphabet of the selected profile.
