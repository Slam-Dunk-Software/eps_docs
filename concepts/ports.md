# Ports

A *port* is a named, designed-in extension point in an EPS. It's where the author
deliberately made room for you to plug in your version of something.

## What makes something a port

A port has three characteristics:

- **Named** — it has an identifier. `MODEL`, `TRANSCRIPT_HANDLER`, `TOOL_REGISTRY`.
- **Documented** — it's described in `CUSTOMIZE.md` with what it does and how to change it.
- **Structural** — it's a seam in the architecture, not a config value.

The difference between a port and a setting: a setting lets you change a value, a port lets
you change behavior.

## Examples

**A port:**

```python
# PORT: TRANSCRIPT_HANDLER
# Replace this function to customize how transcripts are processed.
def handle_transcript(text: str) -> None:
    print(text)  # default: just print it
```

**Not a port (just a setting):**

```toml
[config]
model = "whisper-large-v3"
```

The config key lets you pick a different model. That's useful, but the author made the
choice that model selection is a value, not a behavior. A port would let you swap the
entire inference backend.

## Ports in CUSTOMIZE.md

Every port must be documented in `CUSTOMIZE.md`. The convention:

```markdown
### `PORT_NAME`

**What it does:** One sentence describing the extension point.
**How to customize:** What the user should edit, replace, or configure.
```

`epm init` scaffolds this structure. Fill it in as you build.

## The spectrum

Ports exist on a spectrum:

- **Config port** — environment variable or config file entry. Low friction, limited power.
- **Code port** — a function or class the user replaces. Higher friction, full power.
- **Plugin port** — a directory the harness scans and loads. Highest friction, most power.

All three are legitimate. The right choice depends on how much variation you expect and how
much you want to expose.
