# Quickstart

## Prerequisites

1. [Install Tailscale](https://tailscale.com/download) and log in
2. Install EPM:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/Slam-Dunk-Software/epm/main/install.sh | sh
   ```
3. Install EPC: `epm install epc`

## Deploy your first service

Install it via EPM, then deploy:

```bash
epm install tech_talker
epc deploy tech_talker
```

EPC will:
1. Read `[service]` from the installed `eps.toml`
2. Start it as a persistent background daemon
3. Register the port and pid in `~/.epc/services.toml`

You can also deploy from a local project directory:

```bash
epc deploy --local ./my-project
# or from inside the project:
epc deploy
```

## Check what's running

```bash
epc ps
```

```
NAME          PORT     PID  STATUS    URL
tech_talker   8080   12345  running   http://100.x.x.x:8080
```

Open that URL on your phone, laptop, or any device on your Tailnet. Bookmark it.

## Tail logs

```bash
epc logs tech_talker
```

## Stop a service

```bash
epc stop tech_talker
```

## Restart a service

After updating source code or configuration, restart to pick up the changes:

```bash
epc restart tech_talker
```

EPC will stop the old process, wait for it to exit, then start a fresh one from
the same directory — no need to remember the original deploy spec.
