# Recipient Config Example

This example shows a PACT v1 config for encrypting to one or more recipients using public keys:

```json
{
  "messagePrefix": "pact1",
  "profile": "pact-box1",
  "profileData": {
    "recipients": [
      {
        "keyId": "alice-main",
        "publicKey": "B6N8vBQgk8i3VdwbEOhstCY3StFqqFPtC9_AsrhtHHw"
      },
      {
        "keyId": "bob-main",
        "publicKey": "WGmv9FBUlzLLqu1eXfmzCm2jHLDldCutWtShp2jxpns"
      }
    ]
  }
}
```

This config carries only recipient public information. Private keys remain out of band.
