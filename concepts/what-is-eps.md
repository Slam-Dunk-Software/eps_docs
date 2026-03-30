# What is EPS?

**Extremely Personal Software** is software that is lightweight, functional, and designed from the start to be made yours. 

An Extremely Personal Software project starts from a package installed from [epm.dev](https://epm.dev/). 

The package is a starting point. It's a scaffold, or harness, or seed (or whatever you want to call it). The package you install from [epm.dev](https://epm.dev/) intentionally ships with minimal features. It's *intentionally open-ended*. From there, you can start using the minimal version to your heart's desires.

The core functionality of the app will work just fine in that minimal state; but, you might quickly find yourself thinking, "It's a bit empty here!".

Do not fret! That emptiness is by design. The `CUSTOMIZE.md` file in the project root will have some ideas for you on some high-value features to start implementing. These features slot into the "ports" of the EPS harness.

So get creative, and find ways you'd like to improve or change your EPS. It's yours.

With just a bit of iteration, you'll be surprised how quickly you'll be able to build your dream app, tailor-made to you. 

## The problem EPS solves

Most software ships as a finished product. It has too many opinions. It makes choices on your behalf.
Customization is an afterthought. A config file, or a plugin system bolted on later.

Recent improvements to LLMs and agentic harnesses (Claude Code, Codex, etc.) have dissolved the moat. Nearly any feature in any app can be reproduced, often in a few minutes, or for the bigger features it might take an afternoon.

What can't be reproduced is *fit*—software that works exactly the way you work, with exactly the integrations you need, and none of the ones you don't. Fit cannot be productized, nor built on an assembly line.

EPS is a bet that the right primitive isn't a finished app, it's a well-designed harness.

## The three properties

An EPS has three properties:

### 1. Intentional ports

A *port* is a named, designed-in extension point. Not a config key, but more of a structural seam where the author deliberately said "this is where you plug in your feature/version/integration/design."

Ports are documented in `CUSTOMIZE.md`. Sometimes they have names. Sometimes they have contracts. The author
thought about them.

### 2. Minimal defaults

The defaults are a demonstration. They show the harness working, prove it's functional, and
give you something to run on day one. But they're not trying to be the final answer.

An EPS that ships with twenty config options and sensible defaults for all of them is
probably not an EPS, it's just software with a lot of settings.

### 3. Interface is the value

What you're getting when you install an EPS is the architecture, not the implementation. The skeleton. The shape of the thing.

This is why EPS authors are secondary. The author gave it a shape, but you have to make it walk.

## Who EPS serves

EPS is software for **people**. Not agents, not pipelines. People.

This is more of a philosophical, or values statement, not a technical distinction.

There is a growing category of developer tooling described as "npm for agents": packages that give AI agents new capabilities, skill bundles that agents can compose, shared infrastructure for autonomous software. That is an interesting problem, but EPS is not solving it.

EPS is solving a different problem: most people can't get software that fits their personal or work life. They get software that fits the median user. EPS is a bet that the right answer to this is a harness. Not a mass-produced product, but a well-designed starting point you can make truly yours.

## Mods

Some implementation patterns are too small to be a harness but too useful to leave undocumented.
A **mod** is a self-contained, LLM-ready implementation pattern for EPS apps. They're not a package, not a library, just structured instructions an agent (or human) can apply to any app.

Mods might be useful to pass, almost like a skill, to an agent. But they might also be useful as inspiration. A growing collection of mods can be a place for knowledge sharing. So [take a look](https://epm.dev/mods) and see how others are getting value from plugging mods into their EPS stack.

See the [mods concept doc](mods.md) (and if you're really interested, see [ADR-0015](../adr/0015-eps-mods.md) for a bit of history).
