# Changelog

## Unreleased

- Reframed PACT v1 around a small set of fixed named profiles instead of user-assembled crypto/layout knobs.
- Added neutral config fixtures for `pact-psk1` and `pact-box1`.
- Removed implementation-shaped examples from the core spec repo.
- Added documented config conformance fixture schema.
- Added canonical valid and invalid config fixtures for the profile-first `PACT v1` draft.
- Added deterministic crypto vectors for `pact-psk1`.
- Pinned the `pact-box1` wire format, X25519 key format, and cryptographic contract.
- Added deterministic crypto vectors for `pact-box1` single-recipient and multi-recipient envelopes.
- Expanded the spec with canonical serialization and fixture guidance.
- Clarified the intended split between the spec repo and implementation repos.
