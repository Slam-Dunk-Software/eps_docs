# ADR-0013: System Dependency Declaration

**Status:** Accepted
**Date:** 2026-02-27

---

## Problem

Harnesses like `tech_talker` require system tools (`cmake`, `libomp`, `xcpretty`) before they
can be built or used. There is currently no way to declare these requirements in `eps.toml`, no
way for the registry to store them, and no way for `epm install` to warn the user before they
end up with a broken install.

---

## Decision

Add an optional `[system-dependencies]` section to `eps.toml` that declares required system
tools grouped by package manager. The registry stores these declarations as part of the version
record (immutable per ADR-0007). `epm install` checks all declared dependencies before
completing installation and hard-blocks with an actionable error message if any are missing.

### eps.toml format

```toml
[system-dependencies]
brew  = ["cmake", "libomp"]
gem   = ["xcpretty"]
# cargo = ["sccache"]
```

The section is optional. Missing = `{}` (no system dependencies required).

---

## Supported managers

| Manager | Detection method                                           |
|---------|------------------------------------------------------------|
| `brew`  | `brew list --formula <pkg>` or `brew list --cask <pkg>`   |
| `cargo` | `which <pkg>` (cargo-installed binaries land in `$PATH`)  |
| `gem`   | `gem list -i <pkg>`                                        |

`apt` and other Linux package managers are **deferred** (see Consequences).

---

## Behavior at install time

1. After `git clone` + `git checkout` succeed, `epm install` calls `check_system_deps`.
2. For each declared package, the appropriate detection command is run.
3. If all packages are present, installation completes normally.
4. If any packages are missing, installation is **hard-blocked**: the process exits non-zero
   and prints exactly what commands to run to resolve the issue.

```
error: missing system dependencies — run:
  brew install cmake
  brew install libomp
  gem install xcpretty
```

`epm` **never auto-installs** system dependencies. This is intentional: system package
managers have side effects (modifying `/usr/local`, cask GUI apps, etc.) that should only
happen with explicit user consent.

---

## Storage

System dependencies are stored as a JSON text column (`system_deps`) on the `versions` table,
using the same pattern as `authors` and `platforms`:

```
{"brew":["cmake","libomp"],"gem":["xcpretty"]}
```

Default value is `'{}'` (empty JSON object). The column is added via migration
`002_add_system_deps.sql`.

---

## Immutability

System dependencies are part of the version record and are locked at publish time, consistent
with ADR-0007 (immutable versions). A published version's system dependencies cannot be
changed. If requirements change, publish a new version.

---

## Consequences

**Positive:**
- Users get a clear, actionable error instead of a cryptic build failure.
- Registry stores the full dependency picture alongside the version.
- `epm info` can surface system dependencies before install.

**Negative / deferred:**
- `apt` and other Linux package managers are not yet supported. The detection approach
  (`dpkg -l`, `apt list --installed`) varies by distro and is harder to test portably.
  This can be added in a future ADR once there is a concrete harness requiring it.
- No version constraints on system dependencies (e.g. `cmake >= 3.20`). Declared as a plain
  list of package names for simplicity. Version-constrained deps are a future extension.
- Windows package managers (`winget`, `choco`, `scoop`) are not supported. EPS is currently
  macOS/Linux focused (ADR-0006).
