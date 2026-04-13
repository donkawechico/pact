# Changelog

## Unreleased

- Reframed PACT v1 around a small set of fixed named profiles instead of user-assembled crypto/layout knobs.
- Added neutral config fixtures for `pact-psk1` and `pact-box1`.
- Added `pact-psk2` as a conservative inline-base64 shared-secret stock profile.
- Removed implementation-shaped examples from the core spec repo.
- Added documented config conformance fixture schema.
- Added canonical valid and invalid config fixtures for the profile-first `PACT v1` draft.
- Added profile-independent `transportData.charRemap` support for text-surface adaptation.
- Added deterministic crypto vectors for `pact-psk1`.
- Added canonical config and crypto fixtures for `pact-psk2`.
- Pinned the `pact-box1` wire format, X25519 key format, and cryptographic contract.
- Added deterministic crypto vectors for `pact-box1` single-recipient and multi-recipient envelopes.
- Expanded the spec with canonical serialization and fixture guidance.
- Clarified the intended split between the spec repo and implementation repos.
- Added additive self-describing encrypted message preambles with compact profile IDs and compact character-remap encoding.
- Defined message prefixes as bracket-serialized tokens for config-bound encrypted messages.
