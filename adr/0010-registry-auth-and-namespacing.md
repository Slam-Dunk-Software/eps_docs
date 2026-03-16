# ADR-0010: Registry Auth and Package Namespacing

- **Date:** 2026-02-26
- **Status:** Draft

## Context

The registry needs to answer two related questions before `epm publish` can work:

1. **Identity**: who are you, and how does the registry verify it?
2. **Namespacing**: what is your package called, and who owns that name?

These are coupled because the ownership model determines how names are claimed and
defended. Getting this wrong early is hard to fix — name ownership decisions are
effectively permanent once an ecosystem has users.

## Questions To Resolve (Draft)

- How are forks and derivatives handled? If someone forks `tech_talker` and wants to
  publish their version, what do they call it? (`tech_talker_nick`? `tech_talker-whisper`?)
- Should name ownership be transferable between authors?
- Should there be a dispute/reclaim process for abandoned packages?
- Do we want verified publisher badges (like npm's verified orgs)?

## Proposed Decision

### Auth: GitHub OAuth

Authors authenticate via GitHub OAuth. Reasons:

- EPSs are git-sourced; GitHub is already in the supply chain for most authors
- Mirrors the crates.io model, which is well-understood and low-friction
- GitHub identity provides a meaningful, human-verifiable author attribution
- No need to build a separate account/password system

On `epm publish`, the CLI exchanges a GitHub OAuth token for a registry API token stored
in `~/.epm/credentials`. The registry stores the author's GitHub username and uses it as
the canonical identity for package ownership.

Non-GitHub git hosts (GitLab, Sourcehut, self-hosted) are a future concern. For now,
GitHub OAuth is the only supported auth method.

### Namespacing: Flat Names

Package names are flat: `tech_talker`, not `nickagliano/tech_talker`.

This is a deliberate philosophical choice. The EPS ethos is that the software belongs to
the user, not to its author. `tech_talker` is a thing in the world; the author is
secondary. Scoped names (`@nickagliano/tech_talker`) make author attribution structural
and permanent — they imply that every install is of *someone's* version rather than *the*
version.

Flat names also feel right for the intended audience: people who want to say "install
tech_talker" not "install nickagliano's tech_talker."

The tradeoff is squatting risk and collision, addressed below.

### Name Ownership: First-Come-First-Served with Author Lock

The first author to publish a given name owns it, tied to their GitHub identity. This is
the crates.io model.

Specifically:
- `tech_talker` published by `@nickagliano` means `@nickagliano` owns `tech_talker`
- No other author can publish under `tech_talker` unless ownership is transferred
- Ownership is recorded in the registry and displayed on the package page

Names are locked to the GitHub account that first claimed them. The registry does not
have a human review step for name claims — it is purely first-come-first-served.

### Name Rules

- Must start with a lowercase letter: `[a-z][a-z0-9_]*`
- Remaining characters: lowercase letters, digits, underscores
- Underscores preferred over hyphens (consistent with Rust/Python conventions)
- Minimum 2 characters, maximum 64 characters
- Reserved names: `epm`, `eps`, `registry`, `std`, and other obvious conflicts
  (full list maintained in registry config)

These rules are enforced **client-side at `epm init` time** (ADR-0014) as well as
server-side at `epm publish` time. Both must stay in sync.

### API Tokens

After OAuth, the registry issues a scoped API token for `epm publish`. Tokens:
- Are stored in `~/.epm/credentials` (chmod 600)
- Can be revoked via the registry web UI
- Are scoped to publish-only (no admin operations)
- Expire after 90 days and must be refreshed

## Consequences

**Positive:**
- GitHub OAuth is zero-friction for the target audience
- Flat names are clean and consistent with the "software belongs to the user" ethos
- First-come-first-served is simple and requires no review infrastructure
- Author lock prevents silent takeovers

**Negative:**
- GitHub-only auth excludes authors who don't use GitHub (acceptable for now)
- Flat names with FCFS create squatting risk as the ecosystem grows
- No answer yet for forks/derivatives — a forked `tech_talker` needs a different name
  but the right convention isn't defined
- Name disputes (abandoned packages, trademark conflicts) have no resolution process
- Flat names mean `epm search whisper` returns all whisper-related EPSs with no
  author-scoping to disambiguate similar packages
