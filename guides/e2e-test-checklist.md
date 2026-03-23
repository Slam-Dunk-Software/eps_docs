# E2E Getting-Started Test Checklist

This is a **human walkthrough** to run on a fresh machine (or fresh user account) before a public release. It validates the complete getting-started path from zero to running EPS services.

---

## Prerequisites

- A machine with **Tailscale** installed and logged in
- A GitHub account
- A phone/second device on the same Tailnet

---

## Install

- [ ] `curl -fsSL https://epm.dev/install.sh | sh` completes without error
- [ ] `epm --version` shows the expected version
- [ ] `epm` is on `$PATH` in a new shell

## Auth

- [ ] `epm login` opens a browser to GitHub OAuth
- [ ] After authenticating, `~/.epm/credentials` is created
- [ ] `cat ~/.epm/credentials` shows the token (mode 0600)

## Scaffold a new project

- [ ] `epm new shell` creates a `~/eps/shell/` directory
- [ ] `~/eps/shell/eps.toml` exists and has correct `[package]` and `[service]` sections
- [ ] `~/eps/shell/CUSTOMIZE.md` is present and readable
- [ ] `cd ~/eps/shell && npm install` succeeds

## Configure

- [ ] `.env` created with `SHELL_TOKEN=<random>` as documented
- [ ] SSH key steps work (or skip if already have `~/.ssh/palantir_key`)

## Deploy

- [ ] `epm services start` (from inside `shell/`) starts the service
- [ ] `epm services ps` shows `shell` with status `running`
- [ ] `http://<tailscale-ip>:4444` loads in browser on local machine
- [ ] PIN screen appears; correct PIN is accepted
- [ ] Terminal is functional — can type commands, see output
- [ ] Phone can reach the URL over Tailscale
- [ ] `epm services logs shell` shows recent output

## Auto-start

- [ ] `epm services install-startup` installs LaunchAgent (macOS) or systemd unit (Linux)
- [ ] After logout and login, `epm services ps` shows `shell` running again

## Observatory (optional but recommended)

- [ ] `epm new observatory` + `epm services start` deploys observatory
- [ ] Observatory dashboard at `http://<tailscale-ip>:9090` shows `shell` as running
- [ ] `epm login` token works for publishing a test package

## Regression

- [ ] `epm services stop shell` stops the service cleanly
- [ ] `epm services ps` shows `shell` as stopped (not in list)
- [ ] `epm services restart shell` brings it back
- [ ] `epm services remove shell` removes it from services.toml

---

## After completing this checklist

Check the following are in good shape before announcing launch:

- [ ] `epm.dev` loads with correct getting-started docs
- [ ] `epm.dev/docs/guides/skills-setup` renders correctly
- [ ] At least 3 quality packages are visible on `epm.dev/packages`
- [ ] GitHub OAuth flow works end-to-end on production (`https://epm.dev`)
