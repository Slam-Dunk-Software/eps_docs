# EPS Seasonings

A **seasoning** is a self-contained, LLM-ready implementation pattern for EPS apps.

Seasonings fill the gap between "install this" and "write this yourself." They are
documented patterns — not code, not packages — that describe a proven capability and
tell an agent or human exactly how to apply it to any compatible EPS app.

## The analogy

If an EPS harness is a motherboard, a seasoning is a wiring diagram. It doesn't ship
as a component you slot in. It tells you how to wire something correctly, once, based
on what's already worked.

## Properties

- **Not a package** — no `eps.toml`, no `epm install`. Copy the markdown.
- **Stack-agnostic** — written for any EPS web app regardless of framework or language.
- **LLM-targeted** — structured so an agent can apply it to a codebase without guessing.
- **Human-benefitting** — like all EPS primitives, the point is what it does for a person.

## Repository

All seasonings live at: `github.com/nickagliano/eps_seasonings`

Each seasoning is a single markdown file in `seasonings/`.

## How to apply a seasoning

**With an agent:**
> "Apply the pin_gate seasoning to this project. Here's the seasoning: [paste contents]"

Or, if the agent has local file access:
> "Apply the seasonings/pin_gate.md seasoning to palantir."

**Without an agent:**
Read the Implementation section and follow the steps. All code blocks are complete and copy-pasteable.

## Available seasonings

| Seasoning | Description |
|-----------|-------------|
| `haptics` | iOS Taptic Engine feedback via hidden switch input |
| `pin_gate` | 4-digit PIN entry with on-screen numpad and Fibonacci lockout |

## See also

- ADR-0015 — formal definition and rationale for the seasoning primitive
- `CUSTOMIZE.md` — the per-harness extension point documentation that seasonings complement
