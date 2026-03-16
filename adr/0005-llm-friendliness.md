# ADR-0005: LLM-Friendliness as a First-Class EPS Requirement

- **Date:** 2026-02-26
- **Status:** Accepted

## Context

One of the core value propositions stated in `PLAN.md` is:

> EPSs are easy for LLMs to grep, which encourages more people to build on top of them, and more people to use them.

This is not just a nice-to-have — it's a fundamental design principle that distinguishes the EPS ecosystem from traditional software. The reasoning:

- LLMs and agentic AI have eroded the moat of feature-rich, opaque applications
- EPSs compete on customizability, not features; that customizability must be *discoverable*
- A user asking their AI assistant "how do I make tech_talker use a different hotkey?" should get a useful, accurate answer by the LLM simply reading the EPS's source structure
- Agentic workflows (an AI that installs, configures, and customizes EPSs on behalf of a user) require machine-readable customization surfaces

Without an explicit standard, LLM-friendliness will be inconsistent across EPSs, breaking agentic use cases.

## Decision

LLM-friendliness is a **first-class requirement** for EPSs published to the marketplace. The following structural conventions are **required** to pass registry validation at publish time:

### 1. `CUSTOMIZE.md` at the repository root

Every EPS must include a `CUSTOMIZE.md` file that documents, in plain human-readable language:

- What the EPS does at a high level
- Every configurable option, setting, or hook — with name, type, default, and description
- Examples of common customizations
- How to extend the EPS (e.g., adding new scripts, swapping components)

This file is the primary surface LLMs read when asked "how do I customize this EPS?"

### 2. `eps.toml` presence (see ADR-0003)

The manifest itself is structured and machine-readable. Its `customization_guide` field points to `CUSTOMIZE.md`.

### 3. Predictable hook structure

If an EPS defines hooks, they must live in the directory declared in `eps.toml` under `hooks_dir`. Hook scripts must include a comment block at the top describing their purpose and any environment variables they accept.

### 4. Flat, navigable source structure

EPSs should avoid deeply nested directory structures. Key customization files should be reachable within 2 directory levels of the root. This is a convention, not a hard validation rule, but it is documented as a strong recommendation.

---

The `epm publish` command validates that `CUSTOMIZE.md` exists and that `eps.toml` references it. A missing `CUSTOMIZE.md` is a **publish error**, not a warning.

## Consequences

**Positive:**
- Every published EPS is guaranteed to have a human+LLM-readable customization guide
- Agentic workflows can reliably fetch `CUSTOMIZE.md` to understand how to configure an EPS
- Raises the quality floor of the ecosystem — authors are forced to document their customization surface
- Differentiates the marketplace from generic package managers where documentation is optional

**Negative:**
- Higher barrier to publish — authors must write `CUSTOMIZE.md`
- Quality of `CUSTOMIZE.md` varies; the requirement enforces existence but not quality
- May need an LLM-assisted `epm init` scaffold to help authors write `CUSTOMIZE.md` (future work)
