# ADR-0002: Centralized Registry Model

- **Date:** 2026-02-26
- **Status:** Accepted

## Context

`epm` needs a way to resolve short package names to installable EPS packages. For example:

```
epm install tech_talker
```

There are three common models for this:

1. **Decentralized (git URLs)** — packages are identified by their git repository URL; no central server. Example: Go modules.
2. **Centralized index** — a single registry holds package metadata and maps short names to sources. Example: crates.io, npm, Homebrew.
3. **Hybrid (taps/channels)** — a centralized index for discovery, but packages live in user-controlled git repos. Example: Homebrew taps.

Key considerations:

- **Discoverability** — users should be able to search for EPSs by name, category, or keyword without knowing the author's git URL
- **Trust and curation** — the marketplace has an opinionated identity; a central index allows for curation and quality signals
- **EPS conventions enforcement** — the registry can validate that submitted packages meet EPS structural requirements (see ADR-0005)
- **Versioning** — a central registry can enforce semantic versioning and maintain an immutable version history
- **Simplicity for users** — `epm install tech_talker` is a better UX than `epm install github.com/nickagliano/tech_talker`

## Decision

`epm` will use a **centralized registry** hosted at a single authoritative URL (e.g., `registry.eps.dev` or similar). The registry:

- Stores package metadata (name, versions, description, author, source URL, manifest hash)
- Does **not** store package source code or binaries — it stores a pointer to the canonical source (git URL + tag/commit)
- Exposes a REST API consumed by the `epm` CLI
- Is the single source of truth for package name → source resolution
- Is built in Ruby on Rails with PostgreSQL (see ADR-0001)

The registry does **not** need to host binaries. It holds metadata and points to git sources; `epm` clones/downloads from those sources directly. This keeps hosting costs low and keeps authors in control of their own code.

## Consequences

**Positive:**
- Simple, memorable install commands (`epm install <name>`)
- Enables search, trending, and curation features
- Central point for enforcing EPS conventions at publish time
- Immutable version history prevents supply-chain attacks via tag mutation
- Users don't need to know or trust arbitrary URLs

**Negative:**
- Registry is a single point of failure — if it goes down, `epm install` fails (mitigated: local cache)
- Requires hosting infrastructure and ongoing maintenance
- Package authors must publish to the registry; can't install arbitrary git repos without going through it (may add `epm install --git <url>` as an escape hatch later)
- Namespace squatting is a risk (mitigated: scoped names like `@nickagliano/tech_talker` if needed)
