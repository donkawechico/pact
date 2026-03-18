# Discord-GPT-style Profile Example

This example shows how a PACT template can describe a host-app overlay protocol that:

- prefixes messages with `[ENC]`
- expects a raw Base64 key
- packs IV and ciphertext into a single payload
- applies a `+ -> .` and `/ -> !` remap after Base64 encoding

Example JSON body:

```json
{
  "messagePrefix": "[ENC]",
  "keyHandling": "raw-base64-key",
  "payloadLayout": "packed",
  "packedEncoding": "dot-bang-base64-no-padding"
}
```

The corresponding secret still needs to be exchanged separately.
