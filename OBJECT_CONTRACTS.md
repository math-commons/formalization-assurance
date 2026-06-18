# Object contracts — auditing definitions without reading proofs

A proof assistant certifies that a *theorem's proof* is valid; it does **not** certify that a
**definition is the right object** — a development can be `sorry`-free and axiom-clean while
`genus` secretly computes the wrong invariant, or `cutNorm` is silently `|∫∫W|` instead of the
sup over rectangles. **Object contracts** close that gap on the *definition* side, the way
[`VETTING.md`](VETTING.md) + [`AXIOM_AUDIT_FORMAT.md`](AXIOM_AUDIT_FORMAT.md) close it on the
*axiom* side.

A contract is **one card per constructed object** (not per theorem). It lets a mathematician
judge whether the object is the right, non-degenerate one — and *where it is proven vs
asserted* — **without reading any Lean proof**. The mechanism is **differential testing for
mathematics**: evaluate the object at concrete instances and check the results against
independently-known values, with each value's status read from the kernel.

> **Objects vs axioms — use the right artifact.** Object contracts are for *constructed
> objects you can evaluate at instances*. An **axiom** is an assumed `Prop` with nothing to
> evaluate, so it gets a **vetting record** (`VETTING.md`) + closure status
> (`AXIOM_AUDIT_FORMAT.md`), **not** a `known_values` matrix. Putting a `known_values` table on
> an axiom is a category error.

Cards live at `audit/contracts/<Object>.md` in an adopting project, indexed by
`audit/contracts/README.md`.

## Card format

A markdown file with a YAML front-block and a short prose reader's guide. Fields:

| Field | Meaning |
|-------|---------|
| `object` | the Lean declaration name (short) |
| `informal` | one-paragraph plain-English description in textbook language |
| `sources` | textbook citations with chapter / section / theorem numbers |
| `lean` | the Lean `name`, `signature`, and a `body` sketch — **signature only**, not the proof |
| `characterization` | the defining/known properties as `id`'d claims, including at least one **anti-degeneracy** clause (what would make it the *wrong / hack* object) |
| `known_values` | the **test matrix**: each row is `instance → expected → theorem → status → note`. Differential testing for mathematics. |
| `well_definedness` | the instance/fact the definition silently relies on (e.g. integrability, finite-dimensionality) |
| `anti_degeneracy` | `history` of any real degeneracy bug + the `current_guard` that excludes it |
| `status` | one-line summary of where the object is validated |

The **anti-degeneracy clause** is the load-bearing part: it names the property a plausible
look-alike would *fail*, and a `known_values` row witnesses it (e.g. "`t(F, step G)` equals the
finite homomorphism density" rules out a definition that sums instead of multiplies over edges).

## The status vocabulary (the honesty surface)

Every `known_values` cell — and the object overall — is in exactly one state. These are
computed from `#print axioms` on the cell's witnessing theorem, so they **cannot be fudged**:

- **`PROVEN_CORE_AXIOMS`** — depends only on Lean's `propext`, `Classical.choice`, `Quot.sound`.
  Fully from the kernel + the standard library. The gold standard: it validates the *definition
  mechanism*, not merely the value.
- **`proven_via_axiom`** — the value is correct but obtained by *assuming* a named axiom (e.g.
  the value 0 via a high-level axiom rather than by computing it). Validates API-consistency, not
  the definition.
- **`proven_mod_axioms`** — correct value, reduced to a named axiom set. Read as "reduced to
  those inputs".
- **`sorry`** — stated, not proven.

The invariant a validation harness enforces: **no cell is silently in a fifth state** —
"asserted, looks fine, never checked".

### How a cell's status is checked

The witnessing theorem's `#print axioms` is the source of truth (the same generated
`audit/axiom-report.txt` used for the axiom audit; see
[`FORMALIZATION_YAML.md`](FORMALIZATION_YAML.md)). `sorryAx` in the trace ⇒ a hidden `sorry`
(fail). An empty axiom list ⇒ `PROVEN_CORE_AXIOMS`. A non-empty named list ⇒
`proven_mod_axioms` / `proven_via_axiom` (the axiom is named in the cell's `note`). Small
cases are typically pinned inline with `#guard_msgs in #print axioms` so a build fails on drift.

## Mixed status is normal — state it, don't hide it

An object's *structure* can be `PROVEN_CORE_AXIOMS` while one defining property leans on a
vetted axiom. Record both per row rather than collapsing to a single headline — the same
discipline as the **closure-status axis** in `AXIOM_AUDIT_FORMAT.md` (`declared-on-closure`
etc.). A card that shows, say, the quotient/metric rows axiom-clean and the
separation/injectivity row `proven_mod_axioms` is *more* trustworthy, not less.

## Relation to the other artifacts

- **`VERIFICATION_VALIDATION.md`** — object contracts are the per-object instrument of the
  **validation** layer (faithfulness + characterization); this doc is the format.
- **`AXIOM_AUDIT_FORMAT.md` / `VETTING.md`** — the axiom-side analogue. An object whose
  `known_values` row is `proven_mod_axioms` points at an axiom that must have a vetting record.
- **`NUMERICAL_VALIDATION.md`** — a `known_values` row may also be checked against an
  *independent* oracle (numeric/CAS/SMT), not only a Lean theorem; that is the numeric tier of
  the same test matrix.

## Worked example

[`math-commons/graphons`](https://github.com/math-commons/graphons) `audit/contracts/` carries
cards for `homDensity`, `Graphon`/`SymmKernel`, `cutNorm`, `cutDist`, and `GraphonSpace`. Each
has a `known_values` matrix backed by proved theorems (small cases pinned in `AxiomGuard.lean`);
the `GraphonSpace` card demonstrates **mixed status** — quotient/metric/continuity rows
`PROVEN_CORE_AXIOMS`, the separation/moment-injectivity row `proven_mod_axioms` via a vetted
inverse-counting axiom.

A template card lives in [`templates/object-contract.md`](templates/object-contract.md).
