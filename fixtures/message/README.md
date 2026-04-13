# Message Fixture Schema

PACT message fixtures cover self-describing encrypted messages that use the
`[pact]:v1:<profile-id>:<remap-spec>:<encrypted-payload>` preamble.

## Invalid fixtures

Required fields:

- `name`
- `message`
- `expectedErrorContains`

Implementations should fail parsing or validation and surface an error
containing the expected substring.
