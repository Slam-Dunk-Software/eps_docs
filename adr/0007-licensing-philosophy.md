# ADR-0007: Licensing Philosophy

- **Date:** 2026-02-26
- **Status:** Draft

## Context

Licensing is not a formality — it is a trust contract between authors, users, and the ecosystem.
The tldraw case study is instructive:

- **v1.x**: MIT. The community built on it freely.
- **v2.x**: Custom license. Mandatory "Made with tldraw" watermark; commercial use requires a
  paid license.
- **v4.0**: $6,000/year per team for production commercial use. Projects like BigBlueButton,
  which had embedded tldraw for years across thousands of self-hosted deployments, suddenly
  faced requiring every downstream user to buy an individual commercial license.

The problem was not that tldraw eventually charged money. The problem was that **existing
users had built hard dependencies on code that retroactively changed its terms**. The rug was
pulled.

The EPS ecosystem faces the same risk. If an EPS becomes foundational infrastructure for users
and then locks down, those users lose the ability to customize, fork, or even continue using
the software they've built their workflow around. This is antithetical to the EPS philosophy.

The questions this ADR must address:

1. What licenses may EPSs use?
2. How do we prevent retroactive license tightening?
3. What license do the marketplace tools (`epm`, registry server) use?

## Questions To Resolve (Draft)

- Should we allow *any* OSI-approved license, or only permissive ones (MIT, Apache-2.0, BSD)?
  AGPL is OSI-approved but has significant commercial implications.
- Should source-available licenses (BSL, SSPL, Elastic License) be allowed? These are often
  used as a stepping stone to full commercial lockdown.
- Should there be a distinction between "free for personal use" EPSs and ones that allow
  commercial use? Or do we require all EPSs to allow commercial use?

## Proposed Decision

### The Core Principle: License Immutability Per Version

Regardless of what licenses are allowed, **the license of a published version is immutable**.
Once `tech_talker@1.0.0` is published under MIT, it is MIT forever. The registry will refuse
to accept a re-publish of the same version under a different or more restrictive license.

Authors may release future versions under a different license. Users who depend on
`tech_talker@1.0.0` always know what terms they are on. This mirrors how crates.io handles
crate yanking — you can yank a version to signal "don't use this," but the version and its
license remain accessible.

This is the primary protection against the tldraw anti-pattern.

### Acceptable Licenses for EPSs

**Allowed (permissive):**
- MIT
- Apache-2.0
- BSD-2-Clause, BSD-3-Clause
- ISC
- MPL-2.0 (weak copyleft — only modifications to the licensed files must be shared)

**Allowed with a warning at install time:**
- GPL-2.0, GPL-3.0
- AGPL-3.0
  These are valid open-source licenses but carry copyleft obligations that may affect how
  users can build on top of the EPS. `epm install` will display a notice.

**Not allowed:**
- Proprietary / all-rights-reserved
- Source-available licenses (BSL, SSPL, Elastic License, tldraw License)
  These are explicitly excluded because they are the mechanism by which open-source-looking
  projects execute the tldraw anti-pattern. Source-available is not open-source.
- Any license that restricts use based on field-of-endeavor (commercial use, AI training, etc.)
  Even if the restriction seems benign, it is incompatible with the EPS philosophy of
  unconditional customizability.

License validation is enforced by `epm publish`. The registry checks the declared SPDX
identifier against the allowlist.

### License for `epm` and the Registry

The marketplace tooling itself — `epm` CLI and the registry server — is **proprietary / all rights reserved**. This is the pre-launch state. Licensing for the tooling will be revisited before public release.

The EPS ecosystem's licensing philosophy (open EPSs, no rug-pulls) applies to *packages published on the registry*, not to the registry infrastructure itself.

## Consequences

**Positive:**
- License immutability per version gives users a concrete, enforceable guarantee against rug-pulls
- Explicit exclusion of source-available licenses makes the "no tldraw anti-pattern" rule clear
- SPDX-based allowlist is machine-checkable at publish time — no ambiguity
- Tooling license is explicitly deferred until pre-launch

**Negative:**
- Excluding source-available licenses may deter authors who want to eventually monetize their
  EPS commercially — they cannot use BSL as a "open now, commercial later" path
- Copyleft warning at install time may confuse users unfamiliar with licensing
- Some niche but legitimate licenses (e.g. EUPL, CDDL) are not in the allowlist and would
  require a manual exception process
- "License immutability" means a mistake in the declared license cannot be silently corrected —
  authors would need to publish a new version with corrected metadata

## References

- [tldraw 4.0 licensing debate](https://biggo.com/news/202509190115_tldraw_SDK_4.0_Licensing_Debate)
- [SPDX License List](https://spdx.org/licenses/)
