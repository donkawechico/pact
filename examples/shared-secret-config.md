# Shared-secret Config Example

This example shows the smallest useful PACT v1 config for a shared-secret deployment:

- one recognizable message prefix
- one fixed shared-secret profile
- no embedded secret material

```json
{
  "messagePrefix": "pact1",
  "profile": "pact-psk1"
}
```

The shared secret still needs to be exchanged separately.
