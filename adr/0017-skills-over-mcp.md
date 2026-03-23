# ADR-0017: Claude Code Skills Over MCP for Ecosystem Knowledge

- **Date:** 2026-03-23
- **Status:** Accepted

## Context

Delivering ecosystem knowledge to an AI agent requires a delivery mechanism. There are two
main approaches in the Claude ecosystem:

**MCP servers** — a background process the agent connects to over a local socket. The agent
can call tools on the server at query time. For EPS, this meant `eps_mcp`: a Rust binary
that ran persistently, indexed docs, and answered questions via tool calls.

**Claude Code skills** — plain Markdown files in `~/.claude/commands/`. When a user invokes
`/eps`, Claude reads the file as a system prompt injection. No process, no socket, no
background daemon.

`eps_mcp` was the original approach. It worked, but it had problems:

- **Installation friction**: users had to install `epm`, then `epm mcp install eps_mcp`,
  then restart Claude Code. Three steps, and the third step (restart) was easy to forget
- **Maintenance overhead**: `eps_mcp` was a separate Rust binary with its own release
  cadence, its own dependencies, and its own crash surface. When the binary was out of
  date, knowledge was stale or missing
- **Wrong abstraction**: MCP servers shine when they provide *dynamic* capabilities —
  searching a live database, calling an API, checking runtime state. EPS ecosystem
  knowledge is *static* documentation. A static MCP server is overhead for no benefit
- **Process management complexity**: running `eps_mcp` required a persistent process. On
  macOS this meant managing a socket file. On login it might not start. It could crash
  silently
- **The "token burn" claim was backwards**: MCP servers were sold as avoiding token burn
  by only fetching relevant docs. In practice, the model still processes the full context
  anyway, and a well-written Markdown skill file is smaller than the overhead of a tool
  call roundtrip

The EPS philosophy prefers simple, debuggable, inspectable solutions. A Markdown file
sitting in `~/.claude/commands/` is all three. An MCP binary is none of them.

## Decision

EPS ecosystem knowledge is delivered via **Claude Code skills** installed by `epm skills
install eps_skills`. Each skill is a plain Markdown file:

| Skill | What it does |
|---|---|
| `/eps` | EPS overview — what it is, philosophy, getting started |
| `/eps-adr <n>` | Read an Architecture Decision Record |
| `/eps-toml` | Full eps.toml manifest reference |
| `/eps-dev` | TDD guidelines for developing EPS services and packages |

Skills are:
- **Zero-process**: no daemon, no socket, no restart required after install
- **Inspectable**: `cat ~/.claude/commands/eps.md` shows exactly what the agent reads
- **Updateable**: `epm skills install eps_skills` fetches the latest version from the
  registry, same as any other package
- **Composable**: users can add their own skills alongside ecosystem skills without
  any configuration

`eps_mcp` is retired. `epm mcp` command is removed from the CLI. MCP server management
is not part of the EPS ecosystem's core value proposition.

## Consequences

**Positive:**
- Install is one command: `epm install eps_skills`
- No process to manage, no socket to debug, no restart to forget
- Skills work offline — they're just files
- Users can read, fork, and customize skills without understanding MCP protocol
- Simpler `epm` CLI — no `mcp` subcommand to maintain

**Negative:**
- Skills cannot query live state (e.g., "what services are currently running?").
  This is acceptable because ecosystem knowledge is static documentation, not runtime state
- Skills are limited to the context window — very large doc sets cannot be delivered
  as a single skill. This is mitigated by splitting into focused, targeted skills
- Users who had `eps_mcp` installed retain the binary but it is no longer registered
  or maintained; `epm self-uninstall` cleans it up
