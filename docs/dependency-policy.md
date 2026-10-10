# Dependency and Security Policy

Every dependency is supply-chain risk. This policy is the rule; automation
enforces it. `cargo-deny` runs `advisories`, `licenses`, `bans`, and
`sources` checks in CI (`.github/workflows/supply-chain.yml`), and
`cargo-audit` checks the lockfile against the RustSec advisory database.

## Adding a dependency

1. **Justify it.** One line in the PR: what it does and why the stdlib or
   an existing dependency does not cover it. Prefer stdlib and
   already-present dependencies over new ones.
2. **Pick maintained crates.** Check recent releases, open issues, and
   `cargo audit` / RustSec status. Abandoned crates are rejected.
3. **Verify the license.** It must be on the allow-list in `deny.toml`.
   Anything else needs CTO sign-off before merge.
4. **Pin via the lockfile.** All crates come from crates.io only. No git
   or path dependencies on published crates without a documented exception
   in `deny.toml` (`[sources]`).

## Updating a dependency

- Routine bumps go through normal review with a green `cargo deny` /
  `cargo audit` run.
- Security advisories (RustSec): patch promptly; if no fix exists, open an
  issue to track removal or mitigation and document it in the PR.

## License policy

Workspace code is dual-licensed `MIT OR Apache-2.0`. Dependencies must use
a permissive license from the `deny.toml` allow-list. Copyleft (GPL/LGPL/
AGPL) and proprietary licenses are rejected by CI.

## `unsafe` code

`#![forbid(unsafe_code)]` is the default for every crate in this
workspace. The single exception is a small, dedicated module where
`unsafe` is unavoidable (FFI, platform intrinsics):

- the module, not the crate, opts in with a scoped `allow`;
- every `unsafe` block carries a `// SAFETY:` comment stating the
  invariant it upholds;
- the module is fuzz- or property-tested where it parses or transforms
  untrusted input, and Miri-clean where feasible;
- the exception is reviewed in the PR that introduces it — escalate to the
  CTO before merging a new `unsafe` module.

## When checks fail in CI

- **Advisory found:** update the affected crate; if impossible, document
  the risk and get CTO sign-off before ignoring it in `deny.toml`.
- **License rejected:** replace the dependency or request an allow-list
  exception with justification.
- **Duplicate/wildcard versions:** unify on one version before merging.
