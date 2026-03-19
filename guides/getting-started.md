# Getting Started

This guide walks you through installing `epm` and `epc`, then spinning up `shell` — a
web-based terminal with a mobile command palette that runs on your own machine. You'll
have it running in your browser in about 10 minutes. Getting it on your phone takes a few
more.

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

## 3. Get shell

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

### Setup

**Set up SSH access for the terminal:**

`shell` works by having the Node server SSH into your own machine to create a terminal
session — that's how it gets a real PTY without needing root or a native daemon. It needs
a key it can use to authenticate without a password prompt.

**On macOS**, make sure Remote Login is enabled first:
**System Settings → General → Sharing → Remote Login → On**

On Linux, ensure sshd is running: `sudo systemctl enable ssh && sudo systemctl start ssh`

The simplest approach is a dedicated key:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/shell_key -N ""
cat ~/.ssh/shell_key.pub >> ~/.ssh/authorized_keys
```

If you already have an SSH key in `~/.ssh/authorized_keys`, you can skip keygen and set
`SSH_KEY_PATH` to point at it in your `.env` instead.

**Install dependencies:**

`shell` is a Node.js app, so you need npm. If you don't have it, install
[Node.js](https://nodejs.org) first, then:

```bash
npm install
```

> **Don't skip this.** If you run `epc serve` before `npm install`, the service will
> crash on startup. `epc` will show you the error logs, but save yourself the detour —
> run `npm install` first.

**Create a `.env` file with your PIN:**

```
SHELL_TOKEN=1234
```

This is the PIN on the lock screen. Change it to something only you know.

---

## 4. Serve

```bash
epc serve
```

`epc` reads `eps.toml`, starts the server, and registers the service. Check it:

```bash
epc ps
```

```
NAME    PORT   STATUS
shell   4444   running
```

Open `http://localhost:4444` in your browser. Enter your PIN. You're in.

<details>
<summary>Having issues? Expand troubleshooting</summary>
<p>Start by checking the logs:</p>
<pre><code>epc logs shell   # or whatever you named it</code></pre>
<p>Most issues are obvious from the first few lines.</p>

<h4>Setup screen instead of PIN pad</h4>
<p>You haven't set <code>SHELL_TOKEN</code> in your <code>.env</code> yet. Add it:</p>
<pre><code>SHELL_TOKEN=1234</code></pre>
<p>Then <code>epc restart shell</code>.</p>

<h4>"SSH key not found" on startup</h4>
<p>The server can't find your SSH key. Make sure the path in <code>.env</code> matches what you created:</p>
<pre><code>SSH_KEY_PATH=~/.ssh/shell_key</code></pre>
<p>If you're using an existing key, point <code>SSH_KEY_PATH</code> at it and confirm it's in <code>~/.ssh/authorized_keys</code>.</p>

<h4>tmux not found</h4>
<p>Install it: <code>brew install tmux</code> (Mac) or <code>sudo apt install tmux</code> (Linux), then <code>epc restart shell</code>.</p>

<h4>Garbled or missing unicode characters</h4>
<p>tmux needs the right locale. Add to your <code>.env</code>:</p>
<pre><code>LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8</code></pre>
<p>If still broken, add to <code>~/.tmux.conf</code>:</p>
<pre><code>set -g default-terminal "xterm-256color"</code></pre>
<p>Then <code>epc restart shell</code>.</p>

<h4>Port already in use</h4>
<p><code>epc serve</code> will automatically pick the next available port if 4444 is taken and update <code>eps.toml</code>. Check <code>epc ps</code> to see which port your instance landed on.</p>
</details>

---

## 5. Access from your phone

Shell is running on your machine. To reach it from your phone you need a way to connect
— Tailscale is the easiest option, but there are others.

### Option A: Tailscale (recommended)

Tailscale creates a private encrypted network between your devices in about two minutes —
no port forwarding, no firewall config.

**On your Mac:**

The easiest way is the [Tailscale macOS app](https://tailscale.com/download/mac) — it runs
as a menu bar app and manages the daemon automatically. Sign in and you're done.

If you prefer the terminal:

```bash
brew install tailscale
sudo tailscaled &   # start the daemon
tailscale up        # authenticate (opens browser)
```

**Find your Mac's Tailscale IP:**

```bash
tailscale ip -4
```

You'll get something like `100.x.x.x`. EPS will use this automatically on the next deploy.

**On your iPhone:**

Install [Tailscale from the App Store](https://apps.apple.com/us/app/tailscale/id1470499037),
sign in with the **same account**, and tap **Connect**.

Then restart shell so it binds to your Tailscale IP:

```bash
epc restart shell
```

Open `http://<your-tailscale-ip>:4444` on your phone. You're in your shell from anywhere
on your tailnet.

---

### Option B: SSH app (Termius, JuiceSSH, etc.)

If you'd rather use a dedicated SSH client than a browser-based terminal:

1. **Generate an SSH key on your Mac** (if you don't have one):
   ```bash
   ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
   ```

2. **Add it to authorized keys:**
   ```bash
   cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
   ```

3. **Copy the private key to your phone** and add your Mac's LAN IP
   (`ipconfig getifaddr en0`) to your SSH app.

Enable SSH in **System Settings → General → Sharing → Remote Login**.

> You can still deploy EPS harnesses alongside this — `epm new` and `epc serve` work
> the same either way.

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

On **macOS** this installs a LaunchAgent. On **Linux** it installs a systemd user unit (`~/.config/systemd/user/epc-startup.service`). Individual services opt in via `startup = true` in their `eps.toml` — all official EPS harnesses include this by default.

---

## 8. Add observatory

`observatory` watches all your running services and sends you an SMS if something goes down.

```bash
epm runtime install observatory
cd ~/observatory
epc serve
```

```bash
epc ps
```

```
NAME          PORT   STATUS
shell         4444   running
observatory   9090   running
```

Open `http://localhost:9090` (or `http://<your-tailscale-ip>:9090`) to see your dashboard.

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
