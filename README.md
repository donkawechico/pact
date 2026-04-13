# PACT

PACT stands for **Portable Application-layer Cryptography Template**.

PACT is a protocol and configuration spec for sharing application-layer encryption settings between tools that wrap or overlay existing host applications. A PACT config string names a fixed protocol profile and carries only the non-secret public parameters that profile needs. A self-describing PACT message can also carry compact non-secret decoding metadata in its encrypted-message preamble.

## Goals

- Make encryption profile exchange easy across independent apps.
- Keep the number of stock profiles small and opinionated.
- Keep the wire format versioned and deterministic.
- Allow optional transport-layer adaptations without fragmenting profile semantics.
- Support interoperable application-layer encryption overlays over existing host apps.
- Provide conformance fixtures so multiple implementations can verify compatibility.

## Non-goals

- Carrying passwords, raw keys, or any other secret material.
- Defining transport security such as TLS.
- Standardizing UI, key exchange, or trust establishment.

## Repository Layout

- `SPEC.md`: normative PACT v1 spec draft.
- `fixtures/`: conformance fixtures for valid/invalid config strings, self-describing messages, and crypto vectors.
- `examples/`: human-readable usage examples.
- `CHANGELOG.md`: spec evolution notes.

## Implementation Model

The intended project shape is:

- `pact`: the language-neutral spec and fixture repo
- `pact-kotlin`: a Kotlin/JVM reference implementation
- future implementations such as `pact-python` or `pact-js` consuming the same fixtures

## Status

PACT is currently an early public draft. The current direction for `PACT v1` is:

- two shared-secret profiles for pairwise and group use
  - one compact ASCII85 option
  - one conservative inline-base64 option
- one public-key recipient profile for direct and small-group use
- self-describing encrypted message preambles using `[pact]:v1:<profile-id>:<remap-spec>:<encrypted-payload>`
- deterministic crypto fixtures for all stock profiles
- profile-specific extensions layered in after the core stock profiles are stable
