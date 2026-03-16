# CUSTOMIZE.md

Every EPS must include a `CUSTOMIZE.md` at its repository root. It is the single required
artifact of the EPS contract.

## What it is

`CUSTOMIZE.md` is a flat Markdown file that documents every port in the EPS — the named
extension points where users make the harness their own. It's written for the person who
just cloned the repo and wants to know: *what do I actually change?*

It must be:

- **Complete** — every port is listed
- **Flat** — one file, readable in one fetch (for both humans and LLMs)
- **Actionable** — tells the user *how* to customize, not just *what* can be customized

## Required structure

```markdown
# <package-name> — Customization Guide

Brief description of what the harness does and who it's for.

## Ports

### `PORT_NAME`

**What it does:** Description.
**How to customize:** Instructions.

### `ANOTHER_PORT`

...

## Getting Started

How to clone, configure, and run.
```

`epm init` scaffolds this structure. The registry enforces that the file exists at publish
time.

## CUSTOMIZE.md vs developer docs

`CUSTOMIZE.md` is for **users** — people installing and personalizing the EPS.

If your EPS has an extension API, plugin system, or internal architecture worth
documenting for developers who want to build on it, that belongs in an mdBook (see
[ADR-0012](../adr/0012-mdbook-documentation.md)). `CUSTOMIZE.md` can be included as a
chapter in the book, but must also remain as a standalone file at the root.

## LLM-friendliness

One flat file, one fetch. An LLM answering "how do I change the transcription model in
tech_talker?" can fetch `CUSTOMIZE.md` directly and get the answer without navigating a
doc site. This is a design constraint, not a coincidence.
