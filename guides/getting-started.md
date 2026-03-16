# Getting Started

This guide will walk you through installing the `epm` CLI, spinning up your first personal
app (the `todo` harness), and getting it running as a persistent service with `epc`. By the
end you'll also have `observatory` watching your stack.

For networking, we'll use Tailscale — it's a nifty setup that makes your personal services
reachable from any of your devices with zero configuration. That said, do whatever works for
you. Run it on localhost, use your LAN, use a different VPN. That's the EPS ethos.

The whole thing takes about 15 minutes.

---

## 1. Install epm

The quickest way:

```bash
curl -fsSL https://raw.githubusercontent.com/Slam-Dunk-Software/epm/main/install.sh | sh
```

Downloads the right pre-built binary for your OS and architecture from
[GitHub Releases](https://github.com/Slam-Dunk-Software/epm/releases) and installs it
to `/usr/local/bin`.

Verify it worked:

```bash
epm --version
```

---

## 2. Get the todo harness

```bash
epm new todo
cd todo
```

This clones the `todo` harness, strips the upstream git history, and gives you a fresh repo
that's entirely yours. You own it from the first commit.

Open `CUSTOMIZE.md` — this is the harness author's note to you. It documents the extension
points: what's designed to be changed, what the defaults are, and how to make it your own.
Read it before you do anything else.

---

## 3. Set up Tailscale (recommended)

> **Note:** You can run `todo` however you like — bind it to localhost, expose it on your
> LAN, use a VPN you already have. Tailscale is just what we recommend because it's the
> easiest way to securely reach your personal services from any of your devices with zero
> config.

[Install Tailscale](https://tailscale.com/download) and sign in. Once you're on a tailnet,
every device you add gets a stable private IP (something like `100.x.x.x`) that only
your devices can see.

Find your Tailscale IP:

```bash
tailscale ip -4
```

You'll use this IP as the `HOST` when starting services. The `todo` harness already expects
this — check the `start` command in its `eps.toml`:

```toml
[service]
start = "HOST=$(tailscale ip -4) cargo run --release"
port  = 8765
```

This means when `epc` starts `todo`, it binds to your Tailscale IP. Open
`http://<your-tailscale-ip>:8765` on your phone or any device on your tailnet — no port
forwarding, no firewall rules.

---

## 4. Install epc

`epc` is the process supervisor for your personal cloud. It deploys your EPS harnesses as
persistent background services and keeps them running.

```bash
cargo install --git https://github.com/nickagliano/epc
```

Verify:

```bash
epc --version
```

---

## 5. Deploy todo

From inside your `todo` directory:

```bash
epc deploy .
```

`epc` reads the `eps.toml`, starts the service using the `start` command, and registers it
in `~/.epc/services.toml`. The first run compiles the release binary — this takes a minute.

Check that it's running:

```bash
epc ps
```

You should see something like:

```
NAME   PORT   STATUS
todo   8765   running
```

Open `http://<your-tailscale-ip>:8765` in your browser. Your todo app is live.

---

## 6. Make it yours

Now's the time to go back to `CUSTOMIZE.md` and actually customize. Common starting points:

- **Theme** — change `APP_COLOR` in your environment or `.env` file
- **Title** — update the app name in the UI
- **Features** — add, remove, or rewire anything. You own the source.

After any change that requires a rebuild:

```bash
epc restart todo
```

---

## 7. Get observatory running

`observatory` gives you a web dashboard (and TUI) to monitor all your running services in
one place. It'll show you health status, ports, and send you an SMS if something goes down.

```bash
cd ~
epm new observatory
cd observatory
epc deploy .
```

Once it's running:

```bash
epc ps
```

```
NAME          PORT   STATUS
todo          8765   running
observatory   9090   running
```

Open `http://<your-tailscale-ip>:9090`. You'll see `todo` listed and being monitored.

To configure SMS alerts, see `CUSTOMIZE.md` inside your `observatory` directory — it walks
through wiring in `txtme` or any webhook-compatible notification service.

---

## What's next

- Browse [packages on epm.dev](https://epm.dev/packages) for more harnesses to add to your stack
- Read [What is EPS?](/docs/concepts/what-is-eps) for the philosophy behind how these pieces fit together
- Check the [CUSTOMIZE.md concept doc](/docs/concepts/customize-md) to understand how harness authors design extension points
