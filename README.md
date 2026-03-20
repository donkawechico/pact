# PACT

PACT stands for **Portable Application-layer Cryptography Template**.

PACT is a protocol and configuration spec for sharing application-layer encryption settings between tools that wrap or overlay existing host applications. A PACT string names a fixed protocol profile and carries only the non-secret public parameters that profile needs.

## Goals

- Make encryption profile exchange easy across independent apps.
- Keep the number of stock profiles small and opinionated.
- Keep the wire format versioned and deterministic.
- Support interoperable application-layer encryption overlays over existing host apps.
- Provide conformance fixtures so multiple implementations can verify compatibility.

## Non-goals

- Carrying passwords, raw keys, or any other secret material.
- Defining transport security such as TLS.
- Standardizing UI, key exchange, or trust establishment.

## Repository Layout

- `SPEC.md`: normative PACT v1 spec draft.
- `fixtures/`: conformance fixtures for valid/invalid config strings and crypto vectors.
- `examples/`: human-readable usage examples.
- `CHANGELOG.md`: spec evolution notes.

## Implementation Model

The intended project shape is:

- `pact`: the language-neutral spec and fixture repo
- `pact-kotlin`: a Kotlin/JVM reference implementation
- future implementations such as `pact-python` or `pact-js` consuming the same fixtures

## Status

PACT is currently an early public draft. The current direction for `PACT v1` is:

- one shared-secret profile for pairwise and group use
- one public-key recipient profile for direct and small-group use
- profile-specific extensions and richer envelope details layered in after the core profile model is stable
