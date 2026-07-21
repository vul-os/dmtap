# Changelog

All notable changes to the DMTAP specification are documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versions follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Added

- **§25 DMTAP-PUBSUB: a signed `Subscription`/`SubscriptionRevoke`, topic addressing, and
  push-hint delivery, layered on DMTAP-PUB (§22).** Closes the gap that §22's author feeds (public,
  pull-only), MLS channels (private, closed-membership, TreeKEM-churn-bound) and JMAP push
  (client-to-own-node only) leave open: machine-oriented event distribution with a real, revocable
  subscription object. Four additions, all additive and capability-negotiated (`pubsub-1`): a
  signed, mandatorily-expiring `Subscription` (§25.4) and same-author-only `SubscriptionRevoke`
  (§25.5); topic addressing at the serving/locator layer only, zero wire-object change (§25.3); new
  entries pushed as ordinary sealed MOTEs (`FeedHint`, kind `0x41`) riding the existing §2.6
  deliver/ack/retry path — no new reliability machinery — with `seq`/`tip` advisory-only and never a
  substitute for verified pull (§25.6); and fan-out explicitly governed by §9.9's existing
  group-address rules rather than a new anti-abuse model (§25.7). Default is pull-with-push-hint,
  not true push, because a publisher tracking per-subscriber delivery state is exactly the durable
  middle-state §0.5's architecture exists to avoid; a bounded inline-announce optimization is the
  one deliberately-scoped exception (§25.6.3). Stated honestly: encrypted broadcast to a large open
  subscriber set remains out of scope for v1 (§25.11 item 1) — MLS gives confidentiality with known
  membership, §22/§25 give scale with plaintext, and wanting both at once is unsolved, not
  overlooked. No core wire change: no existing object gains a field, no `Envelope.v`/DNS `v=` bump,
  no flag day. New allocations: message kinds `0x41`–`0x43` (§21.16), capability `pubsub-1`
  (§21.22), six error codes `0x090E`–`0x0913` *within* the existing DMTAP-PUB subsystem byte `0x09`
  (§21.24d — an extension of an extension, not a new subsystem), and two DS-tags. Conformance: 15
  new `PUBSUB` cases (328 → **343**; partition 56 + 6 + 263 + 18 = 343); no new vectors
  (`conformance/vectors/` unchanged).
- **Fixed a latent one-case undercount that had propagated into five prose statements.** The
  `SUITE` family's own coverage-table row (`conformance/SUITE.md`) had not been updated when
  `DMTAP-SUITE-11` was added in the previous commit, undercounting the catalog by exactly one
  vectored case everywhere that row's total fed into: `SUITE.md`'s own `Total` row and two
  paragraphs (327/55/61/68/42/43 → 328/56/62/69/43/44), and `conformance/README.md`'s
  byte-runnable-count sentence (which additionally already disagreed with its *own* next
  paragraph two lines down before this fix). `conformance/suite.json`'s top-level
  `vectors_count`/`referenced_vectors_count` fields carried the same stale 68/42. All now agree
  with the ground truth in `vectors.json`/`suite.json` (69 vectors, 43 driven by cases). Also
  softened the unreproducible "157 of the 183 reject cases..." breakdown in `README.md` to a
  claim actually verifiable from `suite.json` — the original split predates this fix and could not
  be reconciled with either the pre- or post-fix case set under any counting rule tried.
- **Conformance: 22 cases for the new normative requirements** (172 → **194**), closing the gap the
  hardening and one-binary-with-roles commits left open — a MUST with no case is unenforceable, and
  §10.3 makes the suite the operational definition of compatibility. New families: `MIXPROF`
  (§4.4.10a Bootstrap-profile anti-drift constraints), `FLEET` (§4.4.2 derived fleet view), `GUARD`
  (§4.4.8 persistent guard sample + ASN/attested-operator diversity), `LOC` (§4.2 per-epoch
  `peer_id`, §4.2.1 resolution order), `FLOOR` (§9.7a zero-relationship delivery floor, §9.4.1
  memory-hard-PoW floor), `FAILCLASS` (§10.7.0 failure classes) and `GWROLE` (§7.11.4/§9.11
  authorize-never-classify, §7.1b privilege separation). Partition: 46 vectored + 6 self-contained
  + 137 construction-todo + 5 manual-attestation.
- **§21.10 `0x070F` `ERR_POLICY_BELOW_FLOOR`** — referenced by §9.7a since the hardening pass but
  never allocated. The one code in the anti-abuse block whose fault is the recipient's *own* policy
  (`N_floor = 0`, or a VDF-only cold-contact requirement) rather than an inbound object. Registry:
  140 → 141 codes.

### Changed

- **Bucket ladder floor 8 KiB → 16 KiB; inline attachment cap 64 KiB → 48 KiB** (§4.4.1, §16.3,
  §16.4, §2.5, §5.5.1). The 8 KiB floor was arithmetically unsound: it was sized against *one*
  ML-DSA-65 signature and *one* public key, but a MOTE carries **two** of each (`Envelope.sender_sig`
  + `Payload.sig`, `sender_key` + `Payload.from`) plus the X-Wing encapsulated key, so the minimum
  conformant suite-`0x02` MOTE is **11 967 B** with an empty body — 3 775 B *over* the rung it was
  supposed to fit in. §4.4.1 now states the byte arithmetic explicitly, from the §18.2 lengths, so
  it cannot drift again. Two rungs are kept (a third would take the per-message size leak a pinned
  guard observes from 1 bit to log₂3 ≈ 1.58); anchor-suite (`0x04`) announcements are ordinary
  **top-rung** MOTEs at ≈ 26 kB and are *not* excluded from the inline path. The inline attachment
  cap follows: 64 KiB top rung − 11 967 B envelope ⇒ 48 KiB of content.
- **VDF demoted SHOULD → MAY** (§9.4.1, §16.5, `DMTAP-FLOOR-03`). Memory-hard PoW remains the
  interoperable MUST floor and VDF-only remains non-conformant, both unchanged. Three disclosures
  added: sequentiality is a **conjecture** defined only *relatively*, against a `p(t)`-processor
  bound (the foundational definition permits `Eval` up to poly log(t) parallelism); a VDF bounds
  **aggregate parallelism only**, leaving a **10–100×** per-gate latency advantage; and it is
  **not post-quantum** — a quantum adversary computes the group order and collapses the delay.
  The asymmetry that makes this tolerable is stated rather than hidden: a broken VDF is a *future
  spam-cost* problem, repairable locally, not a retroactive confidentiality loss like a broken KEM.
- **X-Wing's standing described accurately** (§1.3, §11.1, §11.3, §15, §16.7, README). It is
  `draft-connolly-cfrg-xwing-kem-10` on the **Independent Submission** stream, **not CFRG-adopted**,
  and **FIPS 203 standardizes no combiner**, warning that a combined KEM containing ML-KEM "might
  not meet IND-CCA2 security" and deferring to SP 800-227. Still pinned — on analysis and a fixed
  HPKE code point, not on standing. `draft-yun-privacypass-arc` likewise relabelled an individual
  draft rather than WG work (§9.3, §11.1).
- **Hybrid signatures: composite message representative, and the exact assurance level** (§1.3,
  §18.1.6, §10.7.1). AND-composition stands, but the components do **not** independently sign the
  object preimage: following the IETF LAMPS composite PQ/T construction both sign
  `M' = DS-tag ‖ 0x00 ‖ suite ‖ body`, which is what makes a component non-separable from the
  composite. Assurance stated as **EUF-CMA, not SUF-CMA** — no composite variant achieves strong
  unforgeability against a quantum adversary — with the note that DMTAP derives no identifier from
  a signature (`Envelope.id` is the content address of `ciphertext`), so it never needed it.
  Suite `0x01` signing is unchanged; the frozen vectors are all `0x01` and are byte-identical.
- **§16.7 gains the `0x04` row and §18.2 the `0x04` lengths** (`sig-val` 7 920 B, `ik-pub` 64 B) —
  the anchor suite was normative in §1.1/§1.2.0 but absent from both length registries, which is
  where the ladder arithmetic reads its numbers from.
- **§4.4.2a's growth argument labelled a design bet**, not a result: volunteer take-up of the mix
  role at scale is unmeasured, and §4.4.10a/§11.3 are what make being wrong about it survivable.

### Fixed

- The **class-group immaturity argument is removed** from §9.4.1/§16.5. It did not survive
  scrutiny — 2018/2019 silence on class-group performance is not evidence about 2026 — and the
  trusted-setup objection is weaker than its strong form (the literature offers a sufficiently
  large random `N`, at a disclosed cost, and class groups). What keeps a VDF out of the floor is
  the absence of a standard, an interoperable parameter set and a pinned proof encoding.
- `0x0311` (`ERR_MIX_DIRECTORY_STALE`) is **FAIL-QUEUED** per §10.7.0/§10.7.2, not
  `FAIL_CLOSED_BLOCK` — the registry still carried the pre-reclassification disposition, which is
  the exact "liveness failure handed a denial-of-service surface" error §10.7.0 exists to forbid.
- `0x030D` (`ERR_MIX_PATH_UNBUILDABLE`) now names the diversity-unmet case, not only the
  empty-layer one, and is scoped to the in-force profile's bar.
- Catalog rows that outlived their clauses: `DMTAP-PRIV-01` still declared the `{2,8,32,64}` KiB
  bucket ladder (cut to `{8,64}`), `DMTAP-PRIV-02` and the §21.12 condition matrix still spoke of a
  mix "directory authority" (deleted — the fleet view is derived).
- `conformance/README.md` stated 157 cases / 104 construction-todo, two waves behind.

## [0.1.0] — 2026-07-21

First versioned cut of the DMTAP specification — sovereign, end-to-end-encrypted, metadata-private mail/chat/files/identity over a peer-to-peer mesh. 22 numbered sections plus conformance vectors. Spec text is CC BY 4.0.
