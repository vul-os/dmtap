# Changelog

All notable changes to the WRAP specification are documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versions follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html) and
track **this document and its conformance vectors** — not the wire-level
`wrap_version` field defined by the protocol itself, which the spec pins at
`0` for the whole of this experimental phase (see §3, §14).

---

## [Unreleased]

No unreleased changes.

---

## [0.1.0] - 2026-07-20

Initial publication of WRAP v0 — the Work Request & Assignment Protocol —
plus its normative conformance vectors. Version 0: no compatibility
guarantee, per the spec's own status (`00-overview.md`, `meta.json`). A
specification with a single implementation is a description, not a standard;
this release does not claim otherwise.

### Added

- **The specification, 16 sections**: [Overview](00-overview.md),
  [Identity](01-identity.md), [Objects](02-objects.md) (WorkOrder, Offer,
  Bid, Assignment, Progress, Attestation), [Wire format](03-wire-format.md)
  (CBOR, versioning, forbidden keys), [Signing](04-signing.md) (sign the
  object, not the frame), [Lifecycle](05-lifecycle.md) (states, transitions,
  computed expiry), [Merge](06-merge.md) (CRDT algebra, HLC, tie-break),
  [Pools](07-pools.md) (discovery without privileged nodes),
  [Trust](08-trust.md) (Sybils, curated pools, inverted reputation),
  [Fulfilment](09-fulfilment.md) (handoff codes, honest weak proofs),
  [Transport](10-transport.md) (HTTP binding, DMTAP binding, USB sticks),
  [Profiles](11-profiles.md) (delivery and trades in v0; other domains are
  expressible), [Errors](12-errors.md), [Security](13-security.md) (including
  what it deliberately does not protect), [Conformance](14-conformance.md),
  and [References](15-references.md).
- **Core protocol design**: identity is an Ed25519 keypair; the issuer —
  already the natural authority for work they want done — makes the one
  contended decision (assignment), removing the need for consensus, leader
  election, or distributed locking. State is a set of signed,
  content-addressed objects that merge deterministically by a CRDT algebra
  (HLC-stamped, with an explicit tie-break), so participants converge
  offline, out of order, and with arbitrary delay.
- **Pools** as discovery infrastructure with no authority over outcomes —
  distribution only, replaceable, join-many/leave-any.
- **Two transport bindings**: plain HTTP and the DMTAP substrate, plus a
  described USB-stick / sneakernet path for full offline exchange.
- **Domain-neutral object model** with delivery and skilled-trades profiles
  shipped in v0; field service, mutual aid, municipal reporting, medical
  courier, and remote freelance work are stated as expressible without a
  core change.
- **PDF build** (`build/`) — Markdown → single HTML → PDF via markdown-it,
  markdown-it-anchor, highlight.js, mermaid, and headless Chrome
  (`puppeteer-core`), with no LaTeX dependency. Produces `wrap.pdf`,
  formatted as an Internet-Draft (`draft-wrap-00`) for rigour, though WRAP is
  explicitly stated not to be seeking RFC status yet.
- **CC BY 4.0 license** ([LICENSE.md](LICENSE.md)) for the specification
  text — implement, quote, and build on it freely, with attribution.
- **Conformance vectors** (`conformance/wrap_vectors.json`, §15.1) — 79
  vectors across all twelve required groups from §15.2 (`encode`, `id`,
  `sign`, `reject`, `authorship`, `hlc`, `tiebreak`, `merge`, `fold`,
  `expiry`, `forward`, `proof`), built around fixed, publicly-known Ed25519
  seeds and a typed-value convention (`conformance/README.md`) so every
  vector is reproducible byte-for-byte in any language, not just this
  repo's own tooling. Weighted toward the cases §15.3 flags as "most likely
  to be got wrong": the author-key tie-break, unknown-field preservation
  through re-encode, rejecting rather than repairing non-canonical
  encodings, pending-not-rejecting unknown referents, and computed expiry.
  Every id (BLAKE3) and signature (Ed25519) value was independently
  cross-checked against a second, non-Go implementation before being
  committed. Validated against a reference driver in
  `github.com/vul-os/propfix` (branch `fix/wrap-cbor`,
  `backend/internal/wrap/vectors_test.go`) — a narrow encode/sign/decode/
  authorship binding that does not cover the `hlc`/`merge`/`fold`/`expiry`/
  `proof` groups, which the conformance README states explicitly rather than
  passing over in silence.
- Along the way, the vector work surfaced a real spec ambiguity in §5.4
  step 4 (`ERR_BAD_ID` has no reachable path in the plain
  `[canonical_bytes, signature]` wire binding, since `canonical_bytes`
  excludes the id by construction) — recorded in
  [`conformance/README.md`](conformance/README.md) pending a spec fix.

### What this isn't (by design, stated in the spec itself)

Not a payment system (compensation terms only; settlement is out of band,
no escrow, no currency, no blockchain), not a mesh (point-to-point plus
pools, to avoid broadcasting customer addresses), not trustless (curated
pools make the trust source explicit rather than pretending it doesn't
exist), not anonymous (protects integrity and ownership of history, not
metadata privacy), and not a governance framework (how a pool admits members
is left to the people affected).
