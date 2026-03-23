# EPS Mods

A **mod** is a self-contained, LLM-ready implementation pattern for EPS apps.

Mods fill the gap between "install this" and "write this yourself." They are
documented patterns — not code, not packages — that describe a proven capability and
tell an agent or human exactly how to apply it to any compatible EPS app.

## The analogy

If an EPS harness is a motherboard, a mod is a wiring diagram. It doesn't ship
as a component you slot in. It tells you how to wire something correctly, once, based
on what's already worked.

## Properties

- **Not a package** — no `eps.toml`, no `epm install`. Copy the markdown.
- **Stack-agnostic** — written for any EPS web app regardless of framework or language.
- **LLM-targeted** — structured so an agent can apply it to a codebase without guessing.
- **Human-benefitting** — like all EPS primitives, the point is what it does for a person.

## Repository

All mods live at: `github.com/nickagliano/eps_mods`

Each mod is a single markdown file in `mods/`.

## How to apply a mod

**With an agent:**
> "Apply the pin_gate mod to this project. Here's the mod: [paste contents]"

Or, if the agent has local file access:
> "Apply the mods/pin_gate.md mod to palantir."

**Without an agent:**
Read the Implementation section and follow the steps. All code blocks are complete and copy-pasteable.

## Available mods

| Mod | Description |
|-----------|-------------|
| `haptics` | iOS Taptic Engine feedback via hidden switch input |
| `pin_gate` | 4-digit PIN entry with on-screen numpad and Fibonacci lockout |

## See also

- ADR-0015 — formal definition and rationale for the mod primitive
- `CUSTOMIZE.md` — the per-harness extension point documentation that mods complement
