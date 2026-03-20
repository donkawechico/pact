# Recipient Config Example

This example shows a PACT v1 config for encrypting to one or more recipients using public keys:

```json
{
  "messagePrefix": "pact1:",
  "profile": "pact-box1",
  "profileData": {
    "recipients": [
      {
        "keyId": "alice-main",
        "publicKey": "MDEyMzQ1Njc4OWFiY2RlZjAxMjM0NTY3ODlhYmNkZWY"
      },
      {
        "keyId": "bob-main",
        "publicKey": "YWJjZGVmMDEyMzQ1Njc4OWFiY2RlZjAxMjM0NTY3ODk"
      }
    ]
  }
}
```

This config carries only recipient public information. Private keys remain out of band.
