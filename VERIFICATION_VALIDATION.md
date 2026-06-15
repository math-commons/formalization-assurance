# Verification & Validation — terminology and the layers

The standard V&V split (Boehm; IEEE 1012), specialized to a proof assistant.

## The two questions

- **Verification — "did we build it *right*?"** The proofs are valid: the kernel
  certifies each theorem against its stated form, relative to an explicit list of
  assumptions. In Lean this is *largely automatic* — `lake build` + a
  `#print axioms` audit (see `AXIOM_AUDIT_FORMAT.md`, `FORMALIZATION_YAML.md`).
- **Validation — "did we build the *right thing*?"** The development captures the
  informal mathematical intent. This carries the whole intellectual burden.

> In formal mathematics, **verification is nearly free; the residual is all
> validation.** A fully *verified* development can still be *invalid* — e.g. a
> theorem about the wrong object, a true-but-vacuous statement, or a **false
> axiom** (which type-checks and is caught only by *assumption review*, not the
> kernel).

### Two failure modes a green, axiom-clean build does NOT catch

Both are **validation** failures:

- **(F1) Wrong object.** `Foo X` type-checks and carries all the advertised
  theorems but is secretly a degenerate or wrong-dimensional look-alike. Every
  theorem is true *of that object*; the object just isn't the intended one.
- **(F2) Vacuous / mis-stated statement.** A theorem is true but does not *say*
  what its name claims (a mis-stated invariant, a hypothesis that can never be
  met, the wrong regularity class).

### A third concern, often miscounted as verification

- **Assumption review** — *are the assumed axioms true?* The axiom *certificate*
  (which axioms are assumed) is verification; vetting whether each axiom is *true*
  is a soundness activity the kernel cannot perform. It is its own layer; see
  `VETTING.md`.

## Naming guidance

The **informal↔formal correspondence is a *validation* activity** (it concerns
*meaning*, not proof-validity). Do **not** file it under "verification." If a
project keeps a dedicated correspondence document, name it `FAITHFULNESS.md`, not
`VERIFICATION.md`, and reserve "verification" for the kernel/axiom check.

## The validation ladder

A tiered list of acceptance theorems such that, once proved for a project's
definitions, the definitions are certified to capture the intended theory rather
than a look-alike. Ascending strength:

- **Faithfulness (layer a) — the informal↔formal map.** For each *primary
  object*, the textbook definition beside its exact Lean form; for each *headline
  statement*, the informal claim beside its Lean theorem, a one-line proof idea,
  and a status. Closes (F2) at the level of *reading the statements*; does not, on
  its own, close (F1).
- **Tier A — encoding sanity.** Cheap theorems that catch wrong definitions, ideally
  on *explicit instances* with independently-known answers.
- **Tier B — structural interactions.** The objects fit together.
- **Tier C — characterizing / universal properties.** The named theorems of the
  field.
- **Tier D — downstream consequences that were never design targets.** An
  *independent* axis: Tiers A–C were written by the same process that wrote the
  definitions and can share a blind spot; a Tier-D result (e.g. a named inequality
  proved through the public API) *fails or yields wrong constants* under an
  encoding bug. Cross-check against an independent numerical reference.
- **Tier E — the independent axiomatic specification.** State, auditable against a
  textbook *without reading the implementation*, what "a theory of X" is, then
  prove **existence** (our construction is a model) and **uniqueness** (any two
  models are canonically isomorphic). Together: *the specification has exactly one
  model, and it is our construction* — the definitions are **forced**. This closes
  (F1) outright.

## Operational vs categorical specifications (the central point)

An **operational** specification — a list of properties an object must satisfy (an
API) — is typically **non-categorical**: it accumulates necessary conditions and
leaves the reader to judge whether the list is *complete enough*; that residual
doubt never fully closes. A **categorical** characterization (a universal property,
an initial-object statement, a uniqueness-of-completion theorem) pins the object up
to unique isomorphism in a single statement. **Efficiency of validation comes from
categoricity, not length.**

This does not make the API worthless. A finished formalization should *exceed* the
categorical minimum, because (a) **validation ≠ library** — the universal property
certifies correctness but is hard to *use*; the "redundant" API theorems are what
downstream users cite; (b) **concrete breadth cross-validates the primitives** — an
abstract statement is faithful only relative to its own primitives being right, and
a spread of Tier-A/D consequences triangulates them; (c) **it is nearly free** —
proving the universal property requires most of the API as scaffolding anyway. The
principle: **characterization (the certificate) + a usable API + enough concrete
cross-checks to triangulate the primitives** — the first minimal and categorical,
the rest deliberately broad.

A useful methodology move: when the natural operational spec is non-categorical,
**exhibit a machine-checked counterexample** (a wrong object that passes the API).
It both proves non-categoricity and identifies the load-bearing acceptance tier —
then add the categorical characterization.
