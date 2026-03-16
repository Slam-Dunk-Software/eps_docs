# ADR-0012: mdBook as Recommended EPS Documentation Format

- **Date:** 2026-02-26
- **Status:** Draft

## Context

ADR-0005 requires every EPS to include a `CUSTOMIZE.md` at its root — a single flat
Markdown file documenting the EPS's customization surface. This is the hard minimum.

But there are two distinct documentation audiences for an EPS, and a flat file serves
only one of them well:

- **Users** — people who install the EPS and want to configure and personalize it.
  `CUSTOMIZE.md` is written for them. It answers: "how do I change this setting, swap
  this component, or add this behaviour?"

- **Developers** — people who want to build on top of the EPS: writing extensions,
  registering new tools, forking, or contributing. For an EPS like pi, the developer
  docs — extension API, session model, how to register tools — are arguably the primary
  artifact. None of that belongs in `CUSTOMIZE.md`.

The motherboard analogy holds: `CUSTOMIZE.md` is the quick-start guide that ships in the
box. Developer docs are the hardware developer reference for people building peripherals.

mdBook is the documentation framework used by the Rust Book and much of the Rust
ecosystem. It builds a navigable, structured site from plain Markdown files — readable
by both humans and LLMs at the source level, not just the rendered output.

The parallel to Rust's ecosystem is natural: crates.io + docs.rs means every published
crate automatically gets documentation hosted at `docs.rs/<crate>/<version>`. The EPS
registry can offer the same.

## Decision

### `CUSTOMIZE.md` remains the hard requirement

Nothing in this ADR changes ADR-0005. `CUSTOMIZE.md` must still exist as a standalone
flat file at the repository root. It is user-facing — always present, always readable in
one fetch, always sufficient for a basic customization query.

### mdBook is the recommended home for developer documentation

EPSs that expose an extension surface — tools, plugins, hooks, APIs — are strongly
encouraged to document that surface in an mdBook. This is the developer reference: how to
build on top of the harness, not just how to configure it.

The recommended convention:

```
<repo-root>/
  CUSTOMIZE.md         # required — user-facing customization guide
  book.toml            # optional — signals a full mdBook developer doc site
  docs/
    SUMMARY.md         # mdBook table of contents
    introduction.md    # what the EPS is and who the developer docs are for
    architecture.md    # how the harness is structured internally
    extensions.md      # how to write and register extensions
    api.md             # extension API reference
    examples/
      ...
```

`CUSTOMIZE.md` may be included in the mdBook as an early chapter (e.g. "For Users") to
keep both audiences in one place, but it must also remain as a standalone file at the
root.

### Registry auto-builds and hosts mdBook docs

At publish time, if `book.toml` is present, the registry builds the mdBook and hosts it
at:

```
docs.epm.sh/<package-name>/<version>/
```

This mirrors docs.rs. The build uses the source at the published commit SHA (consistent
with ADR-0009's CAS model), so hosted docs are always pinned to that version's source.

If no `book.toml` is present, the registry renders `CUSTOMIZE.md` as a simple page at
the same URL. Every EPS gets a docs page; ones with developer surfaces get a richer one.

### LLM-friendliness

The two-tier structure gives LLMs complementary entry points:

- `CUSTOMIZE.md` — one flat file, one fetch, sufficient for user-level customization queries
- `SUMMARY.md` — a predictable table of contents an LLM can fetch first to understand the
  doc structure, then drill into the specific chapter it needs
- Raw Markdown source is always available via the registry's source mirror (ADR-0011),
  so LLMs can read source rather than rendered HTML

An agent answering "how do I write an extension for pi?" fetches `SUMMARY.md`, finds
`extensions.md`, fetches that. It does not need to parse a monolithic flat file.

## Consequences

**Positive:**
- Makes the user/developer documentation distinction explicit and structural
- Gives EPS authors building extension surfaces a clear, recommended home for those docs
- Every EPS gets a hosted docs page — zero extra work at the `CUSTOMIZE.md` minimum
- mdBook is familiar to the Rust-adjacent audience most likely to build EPSs
- `SUMMARY.md` gives LLMs navigable structure rather than a flat file to parse
- Consistent URL structure makes docs discoverable by humans and agents alike

**Negative:**
- Registry must run mdBook builds at publish time — adds build infrastructure
- Authors must keep `CUSTOMIZE.md` and the mdBook in sync if both exist; they can drift
- mdBook is a Rust tool; authors in other ecosystems may be unfamiliar with it
- Hosting at `docs.epm.sh` requires a subdomain and CDN — cost scales with published versions
