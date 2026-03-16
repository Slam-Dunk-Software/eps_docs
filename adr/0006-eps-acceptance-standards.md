# ADR-0006: EPS Acceptance Standards and Platform Compatibility

- **Date:** 2026-02-26
- **Status:** Draft

## Context

Not every software project qualifies as an EPS, and not every EPS runs on every platform. The
registry needs a clear, enforceable definition of what it will and won't accept, for two reasons:

1. **Platform safety** — `tech_talker` is macOS/Apple Silicon only. A Windows user running
   `epm install tech_talker` should get a clear, early failure (or ideally, be prevented from
   attempting it at all), not a cryptic build error halfway through.

2. **EPS identity** — the marketplace has an opinionated philosophy. A feature-complete,
   opaque application that happens to have a config file is not an EPS. The registry should be
   able to reject submissions that don't meet the spirit of the format.

These two concerns are related: both are about ensuring the registry only contains software
that will behave predictably and correctly for its intended users.

## Questions To Resolve (Draft)

This ADR is in **Draft** status because the following questions are not yet settled:

- Should `epm install` hard-block on platform mismatch, or warn and allow override with a flag?
- Who enforces EPS identity — automated validation, human review, or both?
- Should there be tiers of acceptance (e.g. "verified EPS" vs. "listed")?

## Proposed Decision

### Platform Identifiers

Platform identifiers use **Rust target triples** as the canonical format. This is a natural fit
for a Rust-first project: `epm` itself is compiled with a target triple baked in at build time
(`std::env::consts::ARCH` + `std::env::consts::OS`, or the full triple via `rustc --print cfg`),
so platform detection requires no custom logic.

Authors declare platforms using exact target triples. There are no fuzzy shorthands like `macos`
— specificity is required so that authors must consciously decide whether they support Intel Mac
vs. Apple Silicon vs. both.

Commonly used triples and their meaning:

| Triple                        | Meaning                              |
|-------------------------------|--------------------------------------|
| `aarch64-apple-darwin`        | macOS on Apple Silicon (M1/M2/M3/M4) |
| `x86_64-apple-darwin`         | macOS on Intel                       |
| `x86_64-unknown-linux-gnu`    | Linux on x86_64 (glibc)              |
| `aarch64-unknown-linux-gnu`   | Linux on ARM64 (e.g. Raspberry Pi)   |
| `x86_64-pc-windows-msvc`      | Windows on x86_64 (MSVC toolchain)   |
| `any`                         | Cross-platform; author asserts it works everywhere |

`tech_talker`, for example, would declare:

```toml
[package]
platform = ["aarch64-apple-darwin"]
```

This is unambiguous: it does not run on Intel Macs, Linux, or Windows — by design.

The registry maintains a whitelist of recognized triples. Submitting an unrecognized triple is
a publish error. New triples can be added to the whitelist as the ecosystem grows.

### Platform Compatibility Enforcement

At install time, `epm` resolves the current platform and compares it against the package's
declared list. If there is no match:

- **Hard block by default.** `epm install tech_talker` on an Intel Mac prints an error and exits:
  ```
  error: tech_talker does not support your platform (x86_64-apple-darwin).
         Supported: aarch64-apple-darwin
  ```
- An `--allow-unsupported-platform` flag exists as an escape hatch for power users who know
  what they're doing (e.g. running under Rosetta, WSL, or a VM).

The registry also surfaces platform filters in search results and package info pages, so users
on a given platform only see compatible packages by default.

### EPS Acceptance Criteria

To be listed in the registry, a package must satisfy all of the following:

#### Hard requirements (enforced by `epm publish`, blocking):
1. Valid `eps.toml` with all required fields (see ADR-0003)
2. `[eps]` block present — explicit declaration that this is an EPS
3. `CUSTOMIZE.md` present at root (see ADR-0005)
4. At least one declared platform
5. A valid semver version string
6. A license declared using a valid SPDX identifier from the approved allowlist (see ADR-0007)

#### Soft requirements (checked, but produce warnings not errors):
- `CUSTOMIZE.md` is non-trivial (> 100 words) — guards against empty placeholder files
- At least one hook defined (`install` at minimum)
- `repository` field points to a publicly accessible git URL

#### Philosophical criteria (not automatically enforced — subject to human review or future tooling):
- The package must be a *harness*, not a finished product. It should ship with minimal
  default behavior and expose clear customization points.
- It should not require a cloud account or external service to function in its base state
  (local-first principle).
- It should not phone home or collect telemetry without explicit user opt-in.

The philosophical criteria are documented here for transparency but are not currently
enforced automatically. A future review process or automated heuristics (e.g., scanning for
telemetry endpoints) may address this.

## Consequences

**Positive:**
- Platform mismatch is caught early and clearly, not buried in a build failure
- Clear acceptance bar gives authors a checklist and gives users confidence in listed packages
- Platform metadata enables registry-side filtering ("show me macOS packages only")
- Documenting philosophical criteria even before enforcement sets expectations for authors

**Negative:**
- Hard-blocking on platform mismatch may frustrate power users (mitigated by escape hatch flag)
- Philosophical criteria without enforcement are aspirational — registry could drift from the
  EPS ethos over time if volume grows and review doesn't scale
- `any` platform declaration is self-reported and unverified; a bad actor could lie
- License requirement excludes personal/private EPSs that authors don't want to open-source
  (may need a "private registry" or "unlisted" concept later)
