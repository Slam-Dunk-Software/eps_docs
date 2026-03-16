# ADR-0009: Install Lifecycle

- **Date:** 2026-02-26
- **Status:** Draft

## Context

ADR-0003 defines the `eps.toml` manifest and its hooks fields. ADR-0006 defines platform
compatibility enforcement. Neither defines what `epm install` actually does end-to-end —
the stages, the directory layout, what hooks receive, how state is tracked, or how the
local package store is structured.

This ADR defines the full install contract.

## Questions To Resolve (Draft)

- Does epm manage system dependencies (e.g. `brew install cmake`) or just warn about them?
- Should installs be atomic? (install to a temp dir, then rename into place)
- What is the rollback behavior if an install hook fails?
- Do we support APFS clonefiles on macOS and hardlinks on Linux, with a copy fallback?

## Proposed Decision

### Directory Layout

```
~/.epm/
  cache/
    git/
      <repo-host>/<owner>/<repo>/<commit-sha>/   # bare git clones, keyed by commit
    builds/
      <source-hash>-<platform-triple>/           # cached build artifacts
  installs/
    <package-name>/
      <version>/                                 # active install of a specific version
  state.toml                                     # installed package registry (see below)
```

The cache is global and shared. The installs directory is where packages live after
installation. Multiple versions of a package can coexist under `installs/<name>/`.

### Stage 1: Resolution

`epm install tech_talker` or `epm install tech_talker@1.0.0`

- Query the registry for package metadata: git URL, commit SHA for the requested version,
  declared platforms, license.
- If no version specified, resolve to the latest published version.
- **Platform check** (ADR-0006): compare current platform triple against declared platforms.
  Hard-block on mismatch unless `--allow-unsupported-platform` is passed.
- **License check** (ADR-0007): if the package is GPL/AGPL, display a notice before
  proceeding. Proprietary or source-available licenses are a hard error.

### Stage 2: Fetch

- Cache key: `(git_url, commit_sha)`
- If `~/.epm/cache/git/<repo>/<commit-sha>/` exists: skip fetch entirely.
- Otherwise: `git clone --depth 1 <git_url>` at the resolved commit SHA into the cache.

Git's object store is itself content-addressed, so the cache inherits integrity guarantees
from git: the commit SHA is both the identifier and the integrity check.

### Stage 3: Build (optional)

Some EPSs require a build step before the install hook runs (e.g. compiling a binary,
running `./run.sh build`). If `eps.toml` declares a `build` hook:

- Cache key: `(source_hash, platform_triple)` where `source_hash` is derived from the
  contents of the package at that commit.
- If a matching build cache entry exists: skip the build step.
- Otherwise: run the build hook, then cache the resulting artifacts.

Build caching is the primary performance win for EPSs with expensive compilation steps.

### Stage 4: Install

- Source: `~/.epm/cache/git/<repo>/<commit-sha>/` (plus any cached build artifacts)
- Destination: `~/.epm/installs/<name>/<version>/`
- Copy strategy (in order of preference):
  1. **APFS clonefiles** (macOS) — zero-overhead copy-on-write
  2. **Hardlinks** (Linux, APFS fallback) — no data duplication
  3. **Full copy** — fallback for filesystems that support neither

This mirrors the approach used by uv (Python) and zerobrew (Homebrew): the store and the
install are separate, and moving from store to install is as cheap as the filesystem allows.

### Stage 5: Hook Execution

After files are in place, epm runs the `install` hook declared in `eps.toml`.

**Working directory:** `~/.epm/installs/<name>/<version>/`

**Environment variables passed to all hooks:**

| Variable              | Value                                      |
|-----------------------|--------------------------------------------|
| `EPM_PACKAGE_NAME`    | e.g. `tech_talker`                         |
| `EPM_PACKAGE_VERSION` | e.g. `1.0.0`                               |
| `EPM_INSTALL_DIR`     | `~/.epm/installs/tech_talker/1.0.0/`       |
| `EPM_CACHE_DIR`       | `~/.epm/cache/`                            |
| `EPM_PLATFORM`        | current Rust target triple                 |

Hooks are run with a clean environment plus these variables. They do not inherit the user's
full shell environment by default. Authors who need PATH entries (e.g. to invoke `brew` or
`cargo`) must declare that dependency explicitly (see open questions re: system deps).

### Stage 6: State Recording

On successful completion of the install hook, epm records the installation in
`~/.epm/state.toml`:

```toml
[[installed]]
name       = "tech_talker"
version    = "1.0.0"
platform   = "aarch64-apple-darwin"
source_sha = "abc123..."
install_dir = "/Users/nick/.epm/installs/tech_talker/1.0.0"
installed_at = "2026-02-26T14:00:00Z"
```

This file is the source of truth for `epm list`, `epm upgrade`, and `epm uninstall`.

## Consequences

**Positive:**
- CAS-based cache means repeat installs of the same version are instant
- Build caching means expensive compile steps are skipped on reinstall
- Clean hook environment is reproducible and debuggable
- State file gives `epm list` and `epm upgrade` a reliable source of truth
- Multiple versions can coexist cleanly under `installs/<name>/`

**Negative:**
- System dependencies (brew, apt, cargo) are not yet handled — hooks that need them will
  fail silently if they're missing unless the author writes defensive checks
- APFS clonefile / hardlink strategy requires platform-specific code in epm
- No rollback defined yet — a failed install hook leaves a partial install in place
- Clean hook environment may surprise authors who expect `$HOME/.cargo/bin` in PATH
