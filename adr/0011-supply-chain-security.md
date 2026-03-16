# ADR-0011: Supply Chain Security

- **Date:** 2026-02-26
- **Status:** Draft

## Context

epm runs install hooks — shell scripts authored by package maintainers — as part of every
install. This is the same threat model as Homebrew (which runs Ruby DSL + shell during
install) and npm (whose `postinstall` lifecycle scripts are the exact mechanism behind
attacks like event-stream). It is a well-understood class of risk, not a unique one.

This ADR defines the threat model and the mitigations epm commits to. It does not aim to
eliminate all risk — that is impossible for a system that runs user-authored scripts. It
aims to make attacks difficult, detectable, and limited in blast radius.

## Threat Model

| Threat | Description |
|--------|-------------|
| **Source redirect** | Author moves a git tag or rewrites history after publish; new installs get different code than what was reviewed |
| **Account takeover** | Attacker compromises author's GitHub account; publishes malicious `@1.0.1`; users who run `epm upgrade` execute the payload |
| **Malicious publish** | Author intentionally publishes a harmful version, then yanks it hoping the window goes unnoticed |
| **Repo disappearance** | Author deletes their GitHub repo; the registry points to nothing; installs break (the left-pad problem) |
| **Typosquatting** | `tech_ta1ker`, `tech_talker_`, etc. lure users into installing a look-alike malicious package |
| **Rug pull** | Author replaces published code after users have adopted it (code equivalent of the license rug pull addressed in ADR-0007) |

## Questions To Resolve (Draft)

- Should hook transparency be mandatory (always show + confirm) or opt-out for trusted authors?
- Should 2FA be mandatory for publish, or strongly encouraged with a warning?
- What is the security disclosure process? (responsible disclosure, CVE assignment, etc.)
- When a compromised account publishes a malicious version, what is the remediation process?
- How long are audit logs retained?

## Proposed Decision

### 1. Commit SHA Pinning (already decided in ADR-0009)

The registry resolves package versions to exact commit SHAs at publish time. `epm install`
fetches by SHA, not by tag or branch name. This kills the source redirect attack entirely:
even if an author rewrites history or moves a tag, the registry's record of what
`tech_talker@1.0.0` points to does not change.

This is the most important single mitigation and it costs nothing — it falls out naturally
from the CAS install architecture.

### 2. Registry Mirrors Source Code

The registry does not only store metadata pointing to a git URL. At publish time, `epm publish`
uploads a content-addressed archive of the package source to the registry's own storage.

This means:
- The registry holds a copy of every published version's source
- If the author's GitHub repo is deleted or made private, installs still work
- The registry's copy is the authoritative source for install; the git URL is metadata only

This solves the left-pad problem and prevents repo deletion from becoming a supply chain
attack vector.

### 3. Content Hashing

At publish time, the registry computes a SHA-256 hash of the package archive and stores it
in the package index. At install time, `epm` downloads the archive and verifies the hash
before extracting or running any hooks.

A hash mismatch is a hard error — epm refuses to proceed and prints:

```
error: hash mismatch for tech_talker@1.0.0
       expected: sha256:abc123...
       got:      sha256:def456...
       This package may have been tampered with. Aborting.
```

The hash is also displayed on the registry package page, allowing users to independently
verify a package before installing.

### 4. Immutable Published Versions

Once `tech_talker@1.0.0` is published, its contents are frozen. The registry will refuse
to accept a re-publish of the same name + version, even from the package owner.

Authors who need to correct a mistake must publish a new version. This mirrors crates.io's
model.

**Yanking:** Authors may yank a version to signal "do not use this." A yanked version:
- Remains in the registry and remains installable (with `--allow-yanked`)
- Is excluded from version resolution by default (`epm install` will not resolve to it)
- Displays a warning if explicitly installed
- Is never deleted — the audit trail must be preserved

Yanking is not a remediation for a compromised version. It is a signal, not a removal.

### 5. Hook Transparency

Before executing any hook, `epm` displays its full contents and prompts for confirmation:

```
tech_talker@1.0.0 install hook (Scripts/install.sh):
────────────────────────────────────────────────────
#!/bin/bash
brew install cmake libomp
...
────────────────────────────────────────────────────
Run this script? [y/N]
```

This is not opt-out. Every install requires explicit confirmation of the hook contents.
The intent is that users — or the LLMs acting on their behalf — read what is about to run.

A `--yes` flag exists for non-interactive use (CI, agentic workflows) but its use is
logged in the audit trail.

### 6. Audit Log

Every publish event is recorded in a public, append-only audit log:

- Timestamp
- Package name + version
- Author GitHub username
- Commit SHA
- Content hash
- Whether the publish was from a 2FA-verified session

The audit log is publicly readable. Anyone can monitor it for suspicious activity (new
versions of popular packages, unusual publish patterns, etc.). This is modelled on
certificate transparency logs.

### 7. Typosquatting Mitigation

At publish time, the registry checks the new package name against existing names using
edit distance. If a new name is within edit distance 2 of an existing package name, publish
is blocked with:

```
error: "tech_ta1ker" is too similar to existing package "tech_talker".
       If this is intentional, contact the registry maintainers.
```

This is a blunt heuristic and will produce false positives for legitimate packages with
similar names. An override process exists but requires manual review.

## What This Does Not Solve

- **A malicious author** who publishes a harmful package under a name that passes all checks.
  Hook transparency is the primary (weak) mitigation; users must read what they run.
- **A compromised author account** that publishes a plausible-looking `@1.0.1` before anyone
  notices. The audit log makes this detectable; there is no automated prevention.
- **Social engineering** of the registry maintainers into accepting a malicious name override.

These are hard problems with no clean technical solution. They are documented here for
honesty, not because they are acceptable.

## Consequences

**Positive:**
- Commit SHA pinning + content hashing + registry mirroring together eliminate the most
  common passive supply chain attacks (source redirects, repo deletion, tampering)
- Immutable versions prevent code rug pulls
- Public audit log enables community monitoring
- Hook transparency puts the user in the loop before anything executes

**Negative:**
- Registry mirroring means the registry stores source code, not just metadata — significant
  storage and infrastructure cost compared to a pure metadata registry
- Mandatory hook confirmation adds friction to every install; agentic workflows will
  routinely pass `--yes`, which somewhat defeats the purpose
- Typosquatting heuristic will block legitimate packages; manual override process adds
  operational burden
- None of this prevents a determined malicious author; it only raises the cost of attack
