# ADR-0008: The Harness Definition — What Makes an EPS an EPS

- **Date:** 2026-02-26
- **Status:** Draft

## Context

ADR-0006 describes EPS acceptance criteria but its philosophical definition of a "harness" is
negative: *not a finished product, not opaque*. That tells authors what to avoid but not what
to aim for.

This ADR provides the positive definition. It is the canonical answer to "is this an EPS?"
and should be referenced wherever that question needs to be answered — by authors before
submitting, by reviewers during evaluation, and by tooling in the future.

## Decision

### The Definition

An EPS is software that is **functional by default but deliberately incomplete**. The default
behavior is a starting point, not a destination. The author consciously withheld features —
not because they couldn't build them, but because the extension surface is where the value
is meant to come from.

### The Motherboard Analogy

Think of an EPS as a motherboard, not a device. A motherboard powers on without RAM. The
BIOS runs. It does things. It is not useless. But nobody would ship it to a customer and
call it a computer — because the author made a deliberate choice not to complete it. The
ports are the product.

This distinguishes an EPS from regular configurable software:

- **VS Code** has an extension system. It is not an EPS. The extensions are a bonus; the
  editor is complete and useful without them.
- **pi** (the agent harness powering OpenClaw) has an extension system. It *is* an EPS.
  It ships with four tools and a short system prompt. The author deliberately chose not to
  add more — because the extension surface is the point. The restraint is the signal.

### The Three Properties

An EPS has all three of the following:

1. **Intentional ports.** Extension points are named, documented, and designed in from the
   start — not config files bolted on later. The author can point to them and say "this is
   where you plug things in."

2. **Functional but deliberately incomplete defaults.** The software works out of the box,
   but the default state is clearly a starting point. The author made a conscious choice to
   leave room rather than ship a complete feature set.

3. **The interface is the value.** The software's worth is in what it enables you to build,
   not primarily in what it does out of the box.

### The Litmus Test

> **Did the author deliberately leave room, or did they just add a config file?**

A finished product with a settings menu is complete software with options. An EPS is
incomplete software by design, where the incompleteness is the feature.

A secondary check: can the author point to specific, named extension points and explain
what is meant to plug into each one? If the answer is vague or the "extension points" are
just environment variables, it is probably not an EPS.

## Relationship to Other ADRs

- **ADR-0005** (LLM-Friendliness) requires `CUSTOMIZE.md` to document the extension
  surface. That requirement is grounded here: if the ports are the product, they must be
  documented as such.
- **ADR-0006** (Acceptance Standards) lists philosophical criteria including "the package
  must be a harness, not a finished product." This ADR is the canonical definition of that
  criterion. ADR-0006 should link here.

## Consequences

**Positive:**
- Authors have a positive definition to aim for, not just a negative one to avoid
- Reviewers have a concrete framework for borderline cases
- The motherboard analogy is memorable and maps cleanly onto the technical requirements

**Negative:**
- "Deliberate" intent is not machine-checkable — enforcement still requires human judgment
  for ambiguous cases
- The line between "functional but incomplete" and "minimal but complete" is a judgment call;
  reasonable people will disagree on edge cases
