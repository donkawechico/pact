# PACT sync metadata

This folder defines downstream implementation targets that should be notified
when PACT's normative contract changes.

## Normative surfaces

Changes to these paths should be treated as contract changes for downstream
implementations:

- `SPEC.md`
- `fixtures/config/**`
- `fixtures/message/**`
- `fixtures/crypto/**`

## Non-normative surfaces

Changes to these paths should not trigger downstream implementation sync on
their own:

- `README.md`
- `CHANGELOG.md`
- `examples/**`

## Current stage

Current stage is intentionally minimal:

- `pact` detects normative changes
- `pact` triggers `pact-python`
- downstream implementation logic lives in `pact-python`, not here

## Design rule

This repo is the language-neutral source of truth for:

- the PACT spec
- normative fixtures
- downstream notification metadata

This repo should not contain language-specific implementation logic, code
generation, or library repair prompts.