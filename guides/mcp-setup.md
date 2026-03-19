# MCP Setup

**eps_mcp** is an [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) server that gives Claude Code deep knowledge of the EPS ecosystem — packages, ADRs, harness templates, and more.

Once installed, Claude can answer questions about EPS, help you design harnesses, and use live documentation instead of guessing.

---

## Install

```bash
epm runtime install eps_mcp
```

This installs the `eps_mcp` binary into `~/.epm/packages/eps_mcp/<version>/`.

---

## Register with Claude Code

Add the following to your `~/.claude.json` under `mcpServers`:

```json
{
  "mcpServers": {
    "eps_mcp": {
      "type": "stdio",
      "command": "/Users/<you>/.epm/packages/eps_mcp/<version>/eps_mcp"
    }
  }
}
```

Replace `<you>` with your username and `<version>` with the installed version (check with `epm list`).

Or use the shortcut:

```bash
epm mcp install eps_mcp
```

This registers it in `~/.claude.json` automatically.

---

## What eps_mcp knows

Once connected, Claude has access to:

| Tool | Description |
|------|-------------|
| `get_overview()` | High-level summary of the EPS ecosystem |
| `list_adrs()` | List all Architecture Decision Records |
| `get_adr(number)` | Full text of a specific ADR |
| `get_concept(name)` | Deep dive into a core concept (package, harness, port, etc.) |
| `get_example(name)` | Canonical example code for a pattern |
| `list_epc_docs()` | List available EPC documentation files |
| `get_epc_doc(path)` | Read a live EPC doc from the repo |

---

## Verify it's working

Start a new Claude Code session and ask:

```
What is an EPS harness?
```

If eps_mcp is connected, Claude will answer from the canonical EPS documentation rather than general knowledge.
