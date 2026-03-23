# ADR-0016: Process Management Built Into epm (Not a Separate Binary)

- **Date:** 2026-03-23
- **Status:** Accepted

## Context

The EPS ecosystem needs a way to run harnesses as persistent background services — starting
them on login, restarting them after crashes, tailing logs, and checking status. Early in
the ecosystem's life this was handled by a separate tool called `epc` (Extremely Personal
Cloud), a standalone CLI binary installed alongside `epm`.

`epc` worked. But having two binaries created friction:

- New users had to install both `epm` and `epc`, and understand why two tools existed
- `epm install <harness>` was followed by `epc serve <harness>` — a two-step deploy with two
  different CLIs and two different mental models
- The boundary between "package management" (`epm`) and "process management" (`epc`) was
  artificial; both exist to support the same goal: getting an EPS running on your machine
- Keeping two binaries in sync meant two release cadences, two install steps in `install.sh`,
  and two sources of breakage
- `epc` had its own state directory (`~/.epc/`), its own registry format, and its own
  command vocabulary that had to be documented and maintained separately

The deeper issue: `epc` as a concept framed process management as a separate concern.
For personal software that runs on a single machine, there is no meaningful separation
between "installed" and "running." A harness that isn't running isn't useful.

## Decision

Process management is implemented as `epm services` — a subcommand of `epm`, not a
separate binary. `epm` is the single CLI for the EPS ecosystem.

```sh
epm services start <name>     # start a service
epm services stop <name>      # stop a service
epm services restart <name>   # restart a service
epm services ps               # list running services
epm services logs <name>      # tail logs
epm services startup          # restart all registered services (used at login)
epm services install-startup  # install LaunchAgent / systemd unit
epm services audit            # check for insecure network bindings
epm services prune            # remove stale services
epm services sync             # repair services.toml from the registry
```

State lives in `~/.epc/` (named for historical continuity; this directory predates the
merge and migrating it is tracked as future work). The services registry at
`~/.epc/services.toml` and the observatory database at `~/.epc/observatory.db` are the
sources of truth for what is running.

The `epm services startup` command is what the LaunchAgent/systemd unit calls at login,
replacing what was previously `epc startup`.

## Consequences

**Positive:**
- One binary to install, one binary to learn
- `epm new <harness> && epm services start` is a single mental context: install then run
- Single release cadence — process management improvements ship with `epm`
- Simpler install.sh: one binary, one PATH entry
- Existing `~/.epc/` state carries forward without migration

**Negative:**
- `epm services` is a longer command than the old `epc` aliases (`epc serve`, `epc ps`)
- `~/.epc/` directory name is now a misnomer; renaming it is a future migration
- The `epm` binary is larger than it would be without process management
