# Getting Started

This guide walks you through installing `epm` and `epc`, then spinning up `shell` — a
web-based terminal with a mobile command palette that runs on your own machine. Once it's
running, you can access your full shell from any device on your tailnet: your phone, a
tablet, another laptop.

From there you're hooked in — run more EPS commands, deploy more harnesses, build your
stack. The whole thing takes about 15 minutes.

---

## 1. Install epm

```bash
curl -fsSL https://raw.githubusercontent.com/Slam-Dunk-Software/epm/main/install.sh | sh
```

Installs a pre-built binary to `/usr/local/bin`. Verify:

```bash
epm --version
```

---

## 2. Install epc

The `epm` installer will ask if you want to install `epc` — say yes. `epc` is the process
supervisor that runs your EPS services as persistent background processes, and you'll need
it for everything that follows.

If you skipped it or installed `epm` non-interactively, run:

```bash
epm runtime install epc
```

Verify:

```bash
epc --version
```

---

## 3. Connect your phone

There are two ways to get phone access to your Mac. Pick one.

### Option A: Tailscale + web terminal (recommended)

Tailscale creates a private encrypted network between your devices in about two minutes — no
port forwarding, no firewall config. The `shell` harness you'll deploy in step 4 is a
web-based terminal that runs in your phone's browser over Tailscale.

**On your Mac:**

The easiest way is the [Tailscale macOS app](https://tailscale.com/download/mac) — it runs
as a menu bar app and manages the daemon automatically. Sign in and you're done.

If you prefer the terminal:

```bash
brew install tailscale
sudo tailscaled &   # start the daemon
tailscale up        # authenticate (opens browser)
```

Either way, a **tailnet** is created automatically — a private network that only your
devices can see.

**On your iPhone:**

Install [Tailscale from the App Store](https://apps.apple.com/us/app/tailscale/id1470499037),
sign in with the **same account**, and tap **Connect**. Your phone is now on the same
tailnet as your Mac.

**Find your Mac's Tailscale IP:**

```bash
tailscale ip -4
```

You'll get something like `100.x.x.x`. Every EPS service binds to this address by default,
so anything you deploy is immediately reachable from your phone at
`http://<tailscale-ip>:<port>`.

Continue to step 4 to deploy the web terminal.

---

### Option B: SSH app (Termius, JuiceSSH, etc.)

If you'd rather use a dedicated SSH client than a browser-based terminal, skip steps 4–8
and connect directly:

1. **Generate an SSH key on your Mac** (if you don't have one):
   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
   ```

2. **Add it to authorized keys:**
   ```bash
   cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
   ```

3. **Copy the private key to your phone** and add your Mac's IP to your SSH app. Use your
   LAN IP (`ipconfig getifaddr en0`) or Tailscale IP if you've installed it.

This approach requires no EPS harness — just sshd, which is already running on macOS.
Enable it in **System Settings → General → Sharing → Remote Login**.

> If you go this route, you can still come back to deploy EPS harnesses later. `epm new`
> and `epc deploy` work the same either way.

---

## 4. Get shell

```bash
epm new shell
cd shell
```

This clones the harness into a directory called `shell/`. If you'd rather call it something
personal — `terminal`, `cockpit`, whatever feels right — just pass a name:

```bash
epm new shell seeing_stone
cd seeing_stone
```

The directory name is yours. The harness is yours. EPS doesn't care what you call it.

Read `CUSTOMIZE.md` — it covers every config knob before you deploy.

### Required setup

**Generate a dedicated SSH key for shell:**

The web terminal works by having the Node server SSH into your own machine to create a
terminal session. It needs its own key so it can do that without a password prompt.

```bash
ssh-keygen -t ed25519 -f ~/.ssh/shell_key -N ""
cat ~/.ssh/shell_key.pub >> ~/.ssh/authorized_keys
```

**Install dependencies:**

`shell` is a Node.js app, so you need npm. If you don't have it, install
[Node.js](https://nodejs.org) first, then:

```bash
npm install
```

> **Don't skip this.** If you run `epc deploy` before `npm install`, the service will
> crash on startup. `epc` will show you the error logs, but save yourself the detour —
> run `npm install` first.

**Create a `.env` file with your PIN:**

```
SHELL_TOKEN=1234
```

This is the PIN on the lock screen. 4 digits recommended — change it to something only you
know.

---

## 5. Deploy

```bash
epc deploy
```

`epc` reads `eps.toml`, starts `node server.js` bound to your Tailscale IP, and registers
the service. Check it:

```bash
epc ps
```

```
NAME    PORT   STATUS
shell   4444   running
```

Open `http://<your-tailscale-ip>:4444` in your browser — or on your phone. Enter your PIN.
You're in your shell from anywhere on your tailnet.

---

## 6. Make it yours

Open `CUSTOMIZE.md` in your `shell` directory. Key things to personalize:

- **`palette.json`** — add your own command shortcuts to the palette (the buttons on the
  side panel / mobile bottom sheet)
- **`SHELL_TOKEN`** — change the PIN in `.env`
- **`TMUX_SESSION`** — attach to a specific tmux session by name
- **`public/index.html`** — the whole frontend is a single file, edit it freely

After any change:

```bash
epc restart shell
```

---

## 7. Start on login

To have `epc` launch all your services automatically when you log in:

```bash
epc install-startup
```

This installs a LaunchAgent that runs `epc startup` at login. Individual services opt in
via `startup = true` in their `eps.toml` — all official EPS harnesses include this by
default.

---

## 8. Add observatory

`observatory` watches all your running services and sends you an SMS if something goes down.

```bash
epm runtime install observatory
cd ~/observatory
epc deploy
```

```bash
epc ps
```

```
NAME          PORT   STATUS
shell         4444   running
observatory   9090   running
```

Open `http://<your-tailscale-ip>:9090` to see your dashboard. From your phone, on your
tailnet, you now have a terminal and a monitoring dashboard for your personal cloud.

See `CUSTOMIZE.md` in your `observatory` directory to wire up SMS alerts via `txtme`.

---

## What's next

- Browse [packages on epm.dev](https://epm.dev/packages) — todo lists, CRMs, note apps,
  SMS endpoints, and more
- Open any package page in your browser with `epm open <name>`
- Read [What is EPS?](/docs/concepts/what-is-eps) for the philosophy behind the stack
- Check the [CUSTOMIZE.md concept doc](/docs/concepts/customize-md) to understand how
  harness authors design extension points
- Use `epm adopt <name>` to pull a harness into your repo as first-class source code, and
  `epm sync <name>` to check for upstream changes later
