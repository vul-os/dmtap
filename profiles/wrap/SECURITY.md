# Security Policy

WRAP is a cryptographic protocol specification, not a running service: there
is no WRAP server to attack here, and this repository ships no keys, no
user data, and no production deployment. That does not make security
reports out of scope — a flaw in the *design* (a signing gap, a merge rule
that lets a dishonest party rewrite history, a trust model that quietly
re-centralizes) would affect every future implementer who builds against
this document, silently and at once. Treat a spec-level finding with the
same seriousness as a code-level one.

The normative security discussion lives in the specification itself at
**[`13-security.md`](13-security.md)**, including what WRAP *deliberately does
not* protect (see also the README's "What it deliberately isn't"). Read that
first — a report that only restates a disclosed, intentional limitation is
not a new finding.

## Reporting a vulnerability

**Please do not open a public issue for security problems.**

- Preferred: [GitHub private vulnerability reporting](https://github.com/vul-os/wrap/security/advisories/new) on `vul-os/wrap`.
- Alternatively, email **vulosorg@gmail.com** with `[wrap security]` in the subject.

Include what you can: the affected section (identity, signing, lifecycle,
merge, pools, trust, fulfilment, transport, or the conformance vectors), a
concrete construction or trace showing the break, and which normative
requirement (a MUST/MUST NOT from §13, or an object rule from §2–§7) it
violates. You'll get an acknowledgement within **72 hours** and a status
update at least every **14 days** until resolution. Please give a reasonable
window before public disclosure — we'll credit you in the release notes
unless you'd rather stay anonymous.

## Scope

Especially interested in:

- **Signing and identity** ([01-identity.md](01-identity.md),
  [04-signing.md](04-signing.md)) — any construction that lets an object be
  attributed to a key that did not sign it, that lets a signature be
  replayed across objects or contexts it wasn't made for, or that weakens
  the key-name/Principal binding.
- **The merge algebra** ([06-merge.md](06-merge.md)) — any pair of valid,
  signed object sets that merge to different results depending on delivery
  order, or any construction that lets a participant retroactively rewrite
  history that has already been observed and merged elsewhere.
- **Lifecycle and expiry** ([05-lifecycle.md](05-lifecycle.md)) — an illegal
  state transition that the object model does not reject, or a computed
  expiry that can be gamed to extend or shorten a window unfairly.
- **Trust and pools** ([07-pools.md](07-pools.md),
  [08-trust.md](08-trust.md)) — any mechanism that lets a pool silently
  regain the privileged-arbiter role WRAP's whole design exists to avoid, or
  a Sybil construction cheaper than the spec's stated assumptions.
- **Fulfilment proofs** ([09-fulfilment.md](09-fulfilment.md)) — a handoff
  code or weak proof that is presented as stronger evidence than the spec
  claims for it.
- **The wire format** ([03-wire-format.md](03-wire-format.md)) — a
  non-canonical encoding that a conformant decoder is required to reject
  (§5.4) but that some accepted construction lets through, or a forbidden
  key that leaks in-band.
- **The conformance vectors** (`conformance/wrap_vectors.json`) — a vector
  whose `canonical_bytes_hex`, `id_hex`, or `signature_hex` does not
  actually reconstruct/verify as documented, since an implementer trusts
  this file byte-for-byte (see [`conformance/README.md`](conformance/README.md)).
  Note the fixed Ed25519 seeds in this file are intentionally public test
  keys, not a vulnerability — do not report "the private keys are
  guessable," that's the point of a reproducible vector set.

Out of scope: settlement/payment mechanics (WRAP explicitly carries
compensation *terms* only — no escrow, no currency, is out of scope by
design), metadata privacy (WRAP protects integrity and ownership of history,
not who-talks-to-whom), and pool governance policy (left to each pool by
design, §08-trust.md).

Implementation-level (not spec-level) vulnerabilities in a specific WRAP
implementation belong in that implementation's own repository — e.g. the
reference conformance driver in `github.com/vul-os/propfix`
(`backend/internal/wrap/vectors_test.go`) — though a finding that turns out
to stem from an ambiguous or under-specified requirement here is welcome
regardless of where it was first noticed.

## Supported versions

Pre-1.0: only the latest published revision of the specification (and the
`main`/`dev` branches) is current. WRAP is version 0 with no compatibility
guarantee (§14); there is no older version to backport a fix to.

## Build tooling

The Markdown → PDF build (`build/`, via `puppeteer-core` and headless
Chrome) runs locally at build time only, over this repository's own
Markdown — it fetches nothing external at build time. A finding there
(e.g. a dependency pulling in unexpected code, or a way for spec content to
break out of the rendered PDF sandbox) is welcome but is a much lower
severity class than a flaw in the protocol itself.
