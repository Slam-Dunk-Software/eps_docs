# ADR-0001: Use Rust as the Primary Implementation Language

- **Date:** 2026-02-26
- **Status:** Accepted

## Context

The Extremely Personal Marketplace requires two main software components:

1. **`epm` CLI** — installed on a user's machine; resolves, downloads, and manages EPS packages
2. **Registry server** — hosts the centralized package index and serves metadata/artifacts

We need to choose an implementation language. Key constraints:

- The CLI must ship as a single, self-contained binary with no runtime dependency (users should not need to install Node, Python, etc.)
- Performance and memory footprint matter for the CLI since it runs on user devices
- The registry server needs to be reliable, fast, and easy to deploy
- The broader project philosophy encourages local-first, lean software — the tooling should embody those values
- The team has a preference for Rust

Alternatives considered:

| Language | Notes |
|----------|-------|
| Go       | Strong CLI ecosystem, single binary, but weaker type safety and less ergonomic error handling |
| Python   | Rapid prototyping, but requires a runtime; poor single-binary story |
| Node.js  | Large ecosystem, but heavy runtime; not aligned with local-first philosophy |
| Rust     | Single binary, zero runtime, best-in-class memory safety, excellent CLI crates (clap, indicatif), strong async story (tokio, axum) |

## Decision

The implementation language is **Rust for the `epm` CLI** and **Ruby on Rails for the registry server**.

The original intent was to use Rust for both, but the registry is fundamentally a CRUD app — it needs a database, a web UI, auth, background jobs, and admin tooling. Rails provides all of that out of the box. Building it in axum would mean reimplementing a web framework. Rails is the right tool.

The CLI stays in Rust: it must ship as a single self-contained binary with no runtime dependency on the user's machine.

Platform-native code (e.g., macOS Swift for UI in EPSs themselves) is explicitly out of scope — EPSs are built by their authors in whatever language makes sense. This ADR only governs the marketplace tooling.

## Consequences

**Positive:**
- Single-binary distribution for `epm` — no install prerequisites beyond the binary itself
- Rails gives the registry a web UI, auth, background jobs, admin, and migrations for free
- Right tool for each job: Rust for the CLI, Rails for the server
- Aligns with the EPS ethos: lean, fast, reliable

**Negative:**
- Two languages means no shared code between CLI and server (manifest parsing is independent in each)
- Registry requires Ruby + bundler on the server; not a single binary
- Longer compile times for the CLI during development
