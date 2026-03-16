# ADR-0003: EPS Package Manifest Format (`eps.toml`)

- **Date:** 2026-02-26
- **Status:** Accepted

## Context

Every EPS in the marketplace must declare its identity, version, and metadata in a machine-readable manifest file. This manifest is:

- Read by `epm` during install, update, and publish
- Parsed by the registry server to validate and index packages
- Intended to be easily understood by LLMs (see ADR-0005)
- Committed to the root of the EPS source repository

Format options considered:

| Format | Notes |
|--------|-------|
| TOML   | Human-readable, minimal syntax, familiar from `Cargo.toml`; excellent `serde` support in Rust |
| JSON   | Machine-friendly but noisy for humans; no comments |
| YAML   | Human-readable but whitespace-sensitive; surprising edge cases |
| Custom | Unnecessary complexity |

TOML is the natural choice for the CLI — it mirrors `Cargo.toml` conventions that Rust developers already know, and the `toml` crate handles parsing. The Rails registry parses it as well using the `toml-rb` gem.

## Decision

Every EPS must include an `eps.toml` file at its repository root. The schema is:

```toml
[package]
name        = "tech_talker"
version     = "1.0.0"          # semver
description = "Local-first audio transcription for macOS using Whisper"
authors     = ["nickagliano"]
license     = "MIT"
platform    = ["aarch64-apple-darwin"]  # Rust target triples (see ADR-0006)
homepage    = "https://github.com/nickagliano/tech_talker"
repository  = "https://github.com/nickagliano/tech_talker"

[eps]
# Declares that this is an EPS (required — not all repos are EPSs)
# Describes the "harness" surface: what customization points exist
customization_guide = "CUSTOMIZE.md"  # relative path to a human+LLM-readable guide
hooks_dir           = "Scripts/"      # directory containing install/configure/update hooks

[hooks]
install   = "Scripts/install.sh"
configure = "Scripts/configure.sh"   # optional
update    = "Scripts/update.sh"      # optional
uninstall = "Scripts/uninstall.sh"   # optional
```

**Required fields:** `[package]` block with `name`, `version`, `description`, `platform`; `[eps]` block presence (signals this is an EPS).

**Optional fields:** hooks, homepage, license, authors, `customization_guide`.

The manifest is parsed by `epm` (Rust, `toml` crate) and independently by the registry server (Rails, `toml-rb` gem). There is no shared parsing library — the schema is the contract.

## Consequences

**Positive:**
- TOML is readable by humans and LLMs alike
- Mirrors `Cargo.toml` — zero new syntax to learn for Rust users
- The `[eps]` block acts as an explicit declaration that distinguishes EPSs from regular software
- `customization_guide` field makes the LLM-friendliness contract explicit and discoverable
- Schema-as-contract: the `eps.toml` format is the single source of truth; both implementations parse it independently

**Negative:**
- TOML is less universally known than JSON outside the Rust ecosystem
- Adding required fields raises the barrier to publish; authors must write a manifest
- Schema will evolve — versioning the manifest format will be needed eventually
