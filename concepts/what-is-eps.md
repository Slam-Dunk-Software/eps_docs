# What is EPS?

**Extremely Personal Software** is software that is functional by default and designed from
the start to be made yours. Not unfinished — *intentionally open*.

## The problem EPS solves

Most software ships as a finished product. It has opinions. It makes choices on your behalf.
Customization is an afterthought — a config file here, a plugin system bolted on later.

LLMs have dissolved the moat. Any feature any app has can be reproduced in an afternoon.
What can't be reproduced is *fit* — software that works exactly the way you work, with
exactly the integrations you need, none of the ones you don't.

EPS is a bet that the right primitive isn't a finished app, it's a well-designed harness.

## The three properties

An EPS has three properties:

### 1. Intentional ports

A *port* is a named, designed-in extension point. Not a config key — a structural seam
where the author deliberately said "this is where you plug in your version."

Ports are documented in `CUSTOMIZE.md`. They have names. They have contracts. The author
thought about them. They're the product.

### 2. Minimal defaults

The defaults are a demonstration. They show the harness working, prove it's functional, and
give you something to run on day one. But they're not trying to be the final answer.

An EPS that ships with twenty config options and sensible defaults for all of them is
probably not an EPS — it's just software with a lot of settings.

### 3. Interface is the value

What you're getting when you install an EPS is the architecture, not the implementation.
The skeleton. The shape of the thing.

This is why EPS authors are secondary. `tech_talker` is a thing in the world. The author
gave it a shape. You give it a life.

## Who EPS serves

EPS is software for **people**. Not agents. Not pipelines. People.

This is not a technical distinction — it's a values distinction. There is a growing category
of developer tooling described as "npm for agents": packages that give AI agents new
capabilities, skill bundles that agents can compose, shared infrastructure for autonomous
software. That is a real and interesting problem. EPS is not solving it.

EPS is solving a different problem: most people can't get software that fits their life. They
get software that fits the median user. EPS is a bet that the right answer to this is a
harness — not a finished product, but a designed starting point that an agent (or anyone)
can help you make truly yours.

The agent is a tool. The human is the point.

When you install `tech_talker`, the goal is not to give an agent a transcription skill. The
goal is for *you* to have a transcription workflow that works exactly the way you work. The
agent might help you configure it. It doesn't benefit from it.

This distinction determines what gets built and why. An EPS that adds no user value but
makes agents more capable is not an EPS — it's agent infrastructure. The question is always:
*whose life does this improve?*

## What EPS is not

The ecosystem around AI agents is maturing fast. Skill bundles, MCP servers, sub-agent
packages, orchestration layers — there is real infrastructure being built to make agents
more capable, and EPM could easily drift into distributing that kind of thing.

We're naming this explicitly because the pull is strong. A package that gives an AI agent
a new capability *looks* like an EPS. It has a manifest. It gets installed. It might even
have a `CUSTOMIZE.md`. But if the primary beneficiary is the agent pipeline and not a
person's daily life, it isn't an EPS — and EPM shouldn't host it.

**EPM distributes harnesses, not agent skill packs.**

The test is the same as always: is there a human whose life is concretely better because
this is installed? If the honest answer is "well, it makes the agent more useful," that's
a different registry.

## The litmus test

*Did the author deliberately leave room, or did they just add a config file?*

A config file lets you change values. A port lets you change behavior. The difference is
whether the author designed for extension or just didn't finish.

## EPS deployment models

EPS have two structural archetypes (app harness vs framework harness), but they also
fall into three deployment models. The deployment model determines how an EPS runs, how
EPC manages it, and how it's accessed.

### Native harness

Runs as a desktop or system application — macOS GUI, global hotkeys, microphone, camera.
No network port. EPC cannot manage it as a daemon.

Distribution: installed locally, launched by the OS or the user directly. Extension
model: can have its own in-app plugin store (e.g. `tech_talker` language packs or model
variants).

Example: `tech_talker`

### Service harness

Binds a TCP port and serves HTTP or WebSocket. EPC deploys it as a persistent daemon and
the dashboard discovers it automatically. Accessible from any device on the tailnet via
a browser — no installation on the client side.

Extension model: CUSTOMIZE.md ports, environment variables, and request middleware.

Examples: `notes`, `todo`, `chat`, `eps-dashboard`

### Tool harness

A CLI invoked on demand. No port, no GUI. EPC doesn't manage it (no `[service]` block).
Composes naturally with other tools; often a building block rather than an end product.

Example: `pi` (OpenClaw)

---

All three are valid EPS. The question is always the same: did the author leave room?

## Seasonings

Some implementation patterns are too small to be a harness but too useful to leave undocumented.
A **seasoning** is a self-contained, LLM-ready implementation pattern for EPS apps — not a
package, not a library, just structured instructions an agent (or human) can apply to any
compatible EPS app.

Seasonings live in the `eps_seasonings` repository. They cover things like: PIN gate auth,
haptic feedback, safe-area fixes. Hand one to an agent alongside your project and it knows
exactly how to wire it in.

See the [seasonings concept doc](seasonings.md) and [ADR-0015](../adr/0015-eps-seasonings.md).

---

## A note on plugin marketplaces

Some native harnesses have enough complexity — language packs, model variants, voice
profiles — to warrant their own plugin distribution layer. That layer is **not** an EPS.
It's infrastructure that supports an EPS.

The distinction matters: EPM distributes harnesses. A plugin marketplace for `tech_talker`
would distribute bundles *for* a harness. Conflating the two would blur what EPM is for.
If a native harness needs a plugin ecosystem, it builds one internally. EPM stays focused
on harness distribution.
