# EPC — Extremely Personal Cloud

EPC is a personal cloud harness. It runs EPS packages as persistent services on your
own hardware and makes them accessible on all your devices via Tailscale.

It is itself an EPS: functional out of the box, but deliberately incomplete. The services
you deploy into it are the extension points. The default state — no services running — is
a starting point, not a destination.

## Who these docs are for

**For users** — people who install EPC and want to deploy and manage services:
start with the [Quickstart](users/quickstart.md).

**For developers** — people who want to build EPS packages that are deployable via EPC,
or who want to extend EPC itself: start with [Architecture](developers/architecture.md).

## The ecosystem

```
EPS   — the app format      (eps.toml, CUSTOMIZE.md, intentional ports)
EPM   — the package manager (install, search, publish)
EPC   — the runtime         (deploy, ps, logs, stop)
```

EPC is the third layer. It assumes EPM is installed and uses it for package installation.
Tailscale is a required dependency — see [ADR-0002](adr/0002-tailscale-as-networking-layer.md).
