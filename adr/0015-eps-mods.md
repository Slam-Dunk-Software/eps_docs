# ADR-0015 — EPS Mods

**Status:** Accepted

---

## Context

As the EPS ecosystem matures, a recurring need has emerged: proven implementation
patterns that don't belong in any one harness, but are useful across many. These
patterns — a PIN gate, haptic feedback, safe-area layout fixes, Fibonacci lockout — are
too small to be full EPS packages and too specific to be generic libraries. They live
in the gap between "install this" and "write this yourself."

The existing primitives don't cover this:

- **EPS harnesses** are full runnable apps. A single-feature pattern is not a harness.
- **EPM packages** are installable units with `eps.toml` manifests. A markdown snippet
  doesn't need a package manager.
- **CUSTOMIZE.md ports** are extension points within a single harness. Cross-harness
  patterns are out of scope.

There is also an LLM-friendliness angle. EPS is designed to be customized with agent
assistance. An agent that can read a structured pattern document and apply it correctly
to any EPS web app is significantly more useful than one reasoning from first principles.
Mods are that document.

---

## Decision

We define a new primitive: the **mod**.

A mod is a **self-contained, LLM-ready implementation pattern** for EPS apps.
It is not code — it is instructions. A mod describes a capability, explains why
it works, and tells an agent (or human) exactly how to apply it to any compatible EPS.

Mods live in the `eps_mods` repository. Each mod is a single markdown
file following a standard structure (see below). They are versioned as a collection,
not individually.

### What a mod is

- A documented, proven implementation pattern
- Stack-agnostic (written for any EPS web app regardless of language or framework)
- Self-contained (everything needed to apply it is in the file)
- LLM-targeted (structured so an agent can apply it without guessing)

### What a mod is not

- Not an EPS harness (not installable via `epm install`)
- Not a library (no build step, no import, no dependency)
- Not a config template (it describes behavior, not settings)
- Not agent infrastructure (like all EPS primitives, the beneficiary is a human)

### Required structure for every mod

```
# Mod: <Name>

## What it does
One paragraph. What capability this adds.

## Why this works
The mechanism. Why this approach rather than the obvious one.

## When to apply
The right context for this mod. What kind of EPS app benefits.

## Implementation
Numbered steps with code blocks. Complete and copy-pasteable.

## Customization
Named extension points. What the applier is expected to change.

## Guidelines
Rules of thumb. What to avoid. What invariants to maintain.

## Notes
Edge cases, browser/OS quirks, version requirements, reference implementations.
```

### Distribution

Mods are consumed by copying or referencing the markdown file. There is no
install command. The standard usage pattern is:

```
"Apply the pin_gate mod to this project. Here's the mod: [paste contents]"
```

Or, if the agent has local file access:

```
"Apply the mods/pin_gate.md mod to palantir."
```

### Relationship to EPM

Mods are adjacent to the EPM ecosystem but not part of it. EPM distributes
harnesses. Mods distribute knowledge. The two can reference each other — a
harness CUSTOMIZE.md might say "apply the pin_gate mod for auth" — but they
are distinct primitives with distinct distribution models.

---

## Consequences

- `eps_mods` becomes a first-class EPS repository alongside `epm` and
  `epm_registry`
- The ecosystem gains a lightweight, no-install distribution primitive for patterns
  that don't warrant full packages
- Agents working on EPS apps have a named, browsable library of proven patterns to
  draw from
- The bar for adding a mod is intentionally low: a pattern that has worked in
  at least one real EPS app and is worth preserving

---

## Current mods

| Mod | Description |
|-----------|-------------|
| `haptics` | iOS Taptic Engine feedback via hidden switch input |
| `pin_gate` | 4-digit PIN entry screen with on-screen numpad and Fibonacci lockout |
