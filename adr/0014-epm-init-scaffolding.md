# ADR-0014: `epm init` — Package Scaffolding Command

- **Date:** 2026-02-27
- **Status:** Accepted

## Context

Authors creating a new EPS had to manually write `eps.toml`, `CUSTOMIZE.md`, and a run
script from scratch, with no canonical template and no validation until `epm publish`. This
created two problems:

1. **Friction** — getting the manifest format right requires reading ADR-0003 and looking at
   an existing package. There's no "just start here" path.
2. **Late failure** — invalid package names (violating ADR-0010 name rules) weren't caught
   until publish time, potentially after the author had done significant work under the wrong
   name.

## Decision

Add `epm init <name>` as a first-class CLI command that scaffolds a new EPS in a
`<name>/` directory.

### Name validation at init time

Before touching the filesystem, `epm init` validates the package name against the same rules
enforced by the registry at publish time (ADR-0010):

- 2–64 characters
- Must start with a lowercase letter (`[a-z]`)
- Remaining characters: lowercase letters, digits, underscores (`[a-z0-9_]`)
- Hyphens, uppercase, spaces, and other characters are rejected

If the name is invalid, the command fails immediately with a clear error that names the
offending character. No directory is created.

### Scaffolded files

`epm init` creates three files:

**`eps.toml`** — the package manifest (ADR-0003). Pre-filled with the package name, version
`0.1.0`, MIT license, and the author's name and email from `git config` (best-effort). The
`description` field defaults to a placeholder but can be set at init time with `--description`
/ `-d`.

**`CUSTOMIZE.md`** — the LLM-friendliness contract (ADR-0005). Pre-filled with the package
name and a structured Ports section template. Every published EPS must have this file; init
ensures authors start with the right structure.

**`run.sh`** — the harness entry point. A minimal executable shell script (`chmod 755`)
with `#!/usr/bin/env bash` + `set -euo pipefail` and a `TODO` placeholder. Scaffolding this
signals to the author that "runnable" is the baseline expectation — an EPS that can't be run
is not yet a harness.

### `git init`

By default, `epm init` runs `git init` in the new directory. EPS installs are git-based
(ADR-0009), so a git repository is a hard requirement before `epm publish`. Authors can
suppress this with `--no-git` if they intend to initialize the repository themselves.

### Flags

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--description` | `-d` | placeholder | Short description written into `eps.toml` |
| `--no-git` | — | false | Skip `git init` |

## Consequences

**Positive:**
- Authors have a working, valid skeleton in one command — `epm init my_pkg` → `epm publish`
  is now the full publish path
- Name validation fails fast with a clear message, before any filesystem state is created
- `CUSTOMIZE.md` is always scaffolded with the right structure, reducing the risk of
  technically-present-but-empty compliance
- `run.sh` being executable from day one reinforces the "functional by default" EPS contract
  (ADR-0008)

**Negative:**
- Name validation in the CLI and registry must stay in sync; a drift between the two would
  allow `epm init` to accept names that `epm publish` rejects (or vice versa)
- The scaffolded `eps.toml` still requires manual edits before publishing (`repository`,
  `description` if not passed, possibly `platforms`)
- No interactive prompt mode — authors who want to fill everything in at once must use flags
  or edit files manually (interactive mode is a future concern)
