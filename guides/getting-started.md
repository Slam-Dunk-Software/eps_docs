# Getting Started

This guide walks you through installing `epm`, then spinning up `shell` — a
web-based terminal with a mobile command palette that runs on your own machine. You'll
have it running in your browser in about 10 minutes. Getting it on your phone takes a few
more.

---

## 1. Install epm

```bash
curl -fsSL https://raw.githubusercontent.com/Slam-Dunk-Software/epm/main/install.sh | sh
```

Installs a pre-built binary to `/usr/local/bin`. Supports macOS (Intel + Apple Silicon) and Linux (x86_64 + ARM64 / Raspberry Pi).

> **Windows:** Native Windows is not supported. Install [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install) and run the installer from there.

Verify:

```bash
epm --version
```

---

## 2. Get shell

```bash
epm new shell
cd ~/eps/shell
```

`epm new` scaffolds into `~/eps/` by default — all your EPS projects live together in one place. If you'd rather call it something personal — `terminal`, `cockpit`, whatever feels right — just pass a name:

```bash
epm new shell seeing_stone
cd ~/eps/seeing_stone
```

The directory name is yours. The harness is yours. EPS doesn't care what you call it.

### Setup

**Set up SSH access for the terminal:**

`shell` works by having the Node server SSH into your own machine to create a terminal
session — that's how it gets a real PTY without needing root or a native daemon. It needs
a key it can use to authenticate without a password prompt.

First, make sure SSH is enabled on your machine:

- **macOS:** System Settings → General → Sharing → Remote Login → **On**
- **Linux:** `sudo systemctl enable ssh && sudo systemctl start ssh`

Then create a dedicated key for shell:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/shell_key -N ""
cat ~/.ssh/shell_key.pub >> ~/.ssh/authorized_keys
```

If you already have a key in `~/.ssh/authorized_keys`, you can skip keygen and point
`SSH_KEY_PATH` at it in your `.env` instead.

**Install dependencies:**

`shell` is a Node.js app. You need **Node 18 or later** — if you're not sure what you have, run `node --version`. If you don't have Node, install it from [nodejs.org](https://nodejs.org) first, then:

```bash
npm install
```

> **Don't skip this.** If you run `epm services start` before `npm install`, the service will
> crash on startup. `epm services logs shell` will show you the error, but save yourself
> the detour — run `npm install` first.

**Create a `.env` file with your PIN:**

```
SHELL_TOKEN=1234
```

This is the PIN on the lock screen. Change it to something only you know.

---

## 3. Serve

```bash
epm services start
```

`epm services` reads `eps.toml`, starts the server, and registers the service. Check it:

```bash
epm services ps
```

```
NAME    PORT   STATUS
shell   4444   running
```

Open `http://localhost:4444` in your browser. Enter your PIN. You're in.

<details>
<summary>Having issues? Expand troubleshooting</summary>
<p>Start by checking the logs:</p>
<pre><code>epm services logs shell   # or whatever you named it</code></pre>
<p>Most issues are obvious from the first few lines.</p>

<h4>Setup screen instead of PIN pad</h4>
<p>You haven't set <code>SHELL_TOKEN</code> in your <code>.env</code> yet. Add it:</p>
<pre><code>SHELL_TOKEN=1234</code></pre>
<p>Then <code>epm services restart shell</code>.</p>

<h4>"SSH key not found" on startup</h4>
<p>The server can't find your SSH key. Make sure the path in <code>.env</code> matches what you created:</p>
<pre><code>SSH_KEY_PATH=~/.ssh/shell_key</code></pre>
<p>If you're using an existing key, point <code>SSH_KEY_PATH</code> at it and confirm it's in <code>~/.ssh/authorized_keys</code>.</p>

<h4>tmux not found</h4>
<p>Install it: <code>brew install tmux</code> (Mac) or <code>sudo apt install tmux</code> (Linux), then <code>epm services restart shell</code>.</p>

<h4>Garbled or missing unicode characters</h4>
<p>tmux needs the right locale. Add to your <code>.env</code>:</p>
<pre><code>LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8</code></pre>
<p>If still broken, add to <code>~/.tmux.conf</code>:</p>
<pre><code>set -g default-terminal "xterm-256color"</code></pre>
<p>Then <code>epm services restart shell</code>.</p>

<h4>Port already in use</h4>
<p><code>epm services start</code> will automatically pick the next available port if 4444 is taken and update <code>eps.toml</code>. Check <code>epm services ps</code> to see which port your instance landed on.</p>
</details>

---

## 4. Access from your phone

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
epm services restart shell
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

> You can still deploy EPS harnesses alongside this — `epm new` and `epm services start`
> work the same either way.

---

## 5. Make it yours

Open `CUSTOMIZE.md` in your `shell` directory. Key things to personalize:

- **`palette.json`** — add your own command shortcuts to the palette (the buttons on the
  side panel / mobile bottom sheet)
- **`SHELL_TOKEN`** — change the PIN in `.env`
- **`TMUX_SESSION`** — attach to a specific tmux session by name
- **`public/index.html`** — the whole frontend is a single file, edit it freely

After any change:

```bash
epm services restart shell
```

---

## 6. Start on login

To have `epm services` launch all your services automatically when you log in:

```bash
epm services install-startup
```

On **macOS** this installs a LaunchAgent. On **Linux** it installs a systemd user unit (`~/.config/systemd/user/epm-startup.service`). Individual services opt in via `startup = true` in their `eps.toml` — all official EPS harnesses include this by default.

---

## 7. Add observatory

`observatory` watches all your running services and sends you an SMS if something goes down.

```bash
epm new observatory
cd ~/eps/observatory
epm services start
```

```bash
epm services ps
```

```
NAME          PORT   STATUS
shell         4444   running
observatory   9090   running
```

Open `http://localhost:9090` (or `http://<your-tailscale-ip>:9090`) to see your dashboard.

---

## 8. Feeling ambitious? Add HTTPS

If you've got Tailscale running, you can get a real HTTPS cert for your machine in about two minutes — no reverse proxy, no Let's Encrypt, no port 80.

Tailscale issues certificates for your MagicDNS hostname (e.g. `my-mac.tail1234.ts.net`) via its built-in CA. Enable it first:

1. In the [Tailscale admin console](https://login.tailscale.com/admin/dns), turn on **MagicDNS** and **HTTPS certificates**.

2. On your Mac, fetch the cert:

```bash
tailscale cert $(tailscale whois $(tailscale ip -4) | grep Name: | awk '{print $2}')
```

This writes two files to the current directory: `<hostname>.ts.net.crt` and `<hostname>.ts.net.key`.

3. Add them to your `.env`:

```
TLS_CERT=/path/to/<hostname>.ts.net.crt
TLS_KEY=/path/to/<hostname>.ts.net.key
```

4. Restart:

```bash
epm services restart shell
```

You can now open `https://<hostname>.ts.net:4444` from any device on your tailnet. Note: once TLS is enabled you'll need to use the hostname — `https://<hostname>.ts.net:4444`. The raw IP still works over plain HTTP if you need it.

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
