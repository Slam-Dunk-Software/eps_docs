# ADR-0004: CLI Tool Named `epm`

- **Date:** 2026-02-26
- **Status:** Accepted

## Context

The command-line interface for the Extremely Personal Marketplace needs a name — the binary users will install and invoke. This name appears in:

- Shell invocations (`epm install tech_talker`)
- Documentation and tutorials
- PATH and system package managers when distributing `epm` itself
- Error messages and help text

Candidates considered:

| Name          | Notes |
|---------------|-------|
| `epm`         | Extremely Personal Manager — short, mirrors `npm`/`gem`/`pip` convention |
| `eps`         | Category name itself; could be confused with the format |
| `marketplace` | Descriptive but verbose; awkward to type repeatedly |
| `xpm`         | Shorter but opaque; `x` doesn't communicate meaning |

## Decision

The CLI binary is named **`epm`** (Extremely Personal Manager).

The command surface follows conventions established by `npm`, `cargo`, and `brew`:

```
epm install <package>         # install an EPS
epm uninstall <package>       # remove an EPS
epm update [<package>]        # update one or all installed EPSs
epm search <query>            # search the registry
epm publish                   # publish current directory as an EPS (reads eps.toml)
epm info <package>            # show package metadata
epm list                      # list installed EPSs
epm configure <package>       # run the configure hook for an installed EPS
```

## Consequences

**Positive:**
- Short and easy to type
- "epm" is a natural acronym — self-documenting once you know what EPS means
- Mirrors well-known tools; muscle memory transfers from `npm`/`gem` usage
- Clean namespace — `epm` is not a widely-used binary name

**Negative:**
- `epm` is not universally recognized without context (unlike `brew` which has brand recognition)
- If the project pivots away from "Extremely Personal" branding, the acronym ages poorly
