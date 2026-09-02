# Contributing

wasco.dev is a registry of generic WebAssembly components, built with `wash`/`wkg`/`wit-bindgen`
against WASI P2 (`wasm32-wasip2`), meant to be usable by any service that runs WASM components —
not tied to any one platform.

## What belongs here

A component belongs in this org if it's generic: no host-specific imports beyond standard WASI
interfaces (`wasi:http/outgoing-handler`, etc.), nothing that only makes sense inside one
platform's runtime. `datetime` and `string_json_functions` are examples of pure logic with no
external dependencies at all; the `*-api` components wrap third-party REST APIs but are still
generic in the same sense — usable by any WASM host, not tied to Betty Blocks or any other single
platform.

If your component only makes sense linked with another one first (e.g. `openai-api` needs
`wac plug`-ing with an HTTP-proxy component before it's a deployable standalone `.wasm`), say so
explicitly in the README, with the exact composition command.

### Wrapping a third-party API

If you're specifically wrapping a REST API, check the
[`openapi-to-wasm` skill](https://github.com/wasco-dev/skills) first — several existing
components (`openai-api`, `glyphic-api`, `heyreach-api`) were generated from it and follow the
same shape:

- Package naming: `{namespace}:{api-name}-api@{version}` (see any existing `*-api` component's
  `wit/world.wit`).
- Credentials are passed as WIT parameters, not read from the environment — components are
  stateless and isolated, so there's no process environment to read from at runtime.

## Toolchain

- Rust targeting `wasm32-wasip2`.
- [`wash`](https://wasmcloud.com/docs/installation) 2.0.0+, `wkg` (installed via
  `wasco-dev/workflows/.github/actions/install-wkg` in CI — same action, use it locally too).
- `wit-bindgen` for the component's WIT interface.

## Before opening a PR

CI (`wasco-dev/workflows/.github/workflows/ci.yml`) runs on every PR and enforces, all with
`RUSTFLAGS="--deny warnings"`:

- `cargo fmt --check`
- `cargo build --target=wasm32-wasip2`
- `cargo clippy --target=wasm32-wasip2`
- `cargo test`

Run these locally before pushing — CI will block the merge otherwise. There's no fixed unit-test
bar beyond "tests exist and pass"; match the coverage style of an existing component like
`datetime` if you're unsure how much to write.

## Commits and branches

- Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/):
  `type: short summary` in the imperative mood. Common types in this org's history: `feat`,
  `fix`, `docs`, `chore`.
- Branch naming isn't strictly enforced; `<type>/<short-description>` (e.g.
  `feat/add-timezone-offset`) matches existing history.

## Opening a PR

If you're the only one working on a given component repo, pushing straight to `main` is fine —
that's how several of these repos got started. Once more than one person is touching a repo,
open a PR instead so review actually happens before something lands. Either way, CI must pass.

PR description should cover: what the component does, which WIT interfaces/capabilities it
imports, and — if it wraps a third-party API — which endpoints/auth flow you tested it against.

## Publishing

Merges to `main` trigger `wasco-dev/workflows/.github/workflows/cd.yml`, which publishes to
`ghcr.io/wasco-dev/<component-name>` and the internal dev/prod Azure registries. **The version in
`wit/world.wit`'s `package` line is what controls this** — the publish step reads that line
directly and hard-fails if that exact version already exists in the registry, so bump it before
merging or the merge's publish step will error. (`Cargo.toml`'s version isn't read by this
pipeline at all and isn't necessarily kept in sync with the WIT version — don't rely on it.)

## Review

This doesn't currently define a required review process — CI passing is the only enforced gate
today. No branch protection exists on `main` in any repo in this org, and there's no CODEOWNERS
file anywhere. That's a decision this document is deliberately not making on its own: does every
PR need an approval before merge (human or otherwise), or is "CI green, maintainer merges their
own PR" still fine for single-maintainer components? If the answer is "yes, require review,"
GitHub's branch protection settings on `main` (required approvals, required status checks) are
the lever — currently unset everywhere. Worth deciding explicitly rather than defaulting into
whatever happens to occur once a component gets a second regular contributor.

## Questions

Open an issue on the relevant repo, or reach out to [Chris Obdam](https://github.com/chrisobdam)
directly — there's no dedicated support channel yet.
