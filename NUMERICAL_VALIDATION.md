# Independent-oracle cross-validation

*Validate that a Lean **statement** says what you meant by checking it against an
**independent, mechanizable oracle** that shares no provenance with the formalization.
**Numeric sampling** — instantiate a parametric fact at sampled points and evaluate
both sides — is the most general instance, because numbers are universal; a CAS, an
SMT/decision procedure, or exact computation are others. Disagreement exposes a
faithfulness bug the kernel cannot see. This generalizes FLMF's mpmath fidelity oracle
from special functions to **many** projects — not all, and only as a **partial** check
(see the boundary below).*

## Why it works

The kernel certifies the *proof*, not that the *statement* is the one you intended (the
F1/F2 failures in [`VERIFICATION_VALIDATION.md`](VERIFICATION_VALIDATION.md)). A wrong
constant, a flipped sign, the wrong normalization convention, or a hypothesis that is
silently too strong or too weak **compiles and proves exactly like the truth**. An
independent oracle catches these *because* it shares no code, language, or author with
the Lean statement: agreement is strong evidence of faithfulness, and a single
disagreement is a definite bug on our side (the statement or its transcription).

This is the `numeric` column of [`CORRESPONDENCE_INDEX.md`](CORRESPONDENCE_INDEX.md) and
the numeric tier of [`FIDELITY_REVIEW.md`](FIDELITY_REVIEW.md), lifted into a reusable,
project-agnostic technique.

## The oracles

The general asset is not "sample and check numerically" — it is **find an independent,
mechanizable oracle for this statement's content**. Instances, widest first:

- **Numeric sampling** — instantiate free variables at sampled points and evaluate both
  sides with an independent high-precision/numerical engine (mpmath, numpy/scipy, LAPACK,
  Monte Carlo). The most general, because almost anything concrete reduces to numbers.
- **CAS** — symbolic identity checking (Sympy, Mathematica). Reaches algebraic identities
  sampling can over- or under-shoot.
- **SMT / decision procedure** — decidable fragments (linear arithmetic, real-closed
  fields via CAD, Presburger). Can discharge some `∀`-over-a-decidable-domain that
  sampling only gives evidence for.
- **Exact computation** — finite/rational arithmetic, finite groups, polynomial identities
  — when an approximate check would be ambiguous.

Each broadens the reachable slice a little, and each has its own decidability boundary.

## How general is it — the boundary

General over a precise slice — *statements whose truth is an independent computation* —
and **partial** even there. Five axes bound it:

1. **Subject matter.** Broad in **concrete** mathematics — special functions, linear
   algebra, combinatorics, number-theoretic identities, inequalities, probability,
   numerical analysis. Near-zero in **abstract/structural** mathematics — algebra,
   topology, geometry, category theory, logic. A Galois group, a cohomology class, a
   measure, a generic point have nothing to evaluate.
2. **Statement shape.** Works for `LHS(x) = RHS(x)`, `P(x)`, bounds — both sides
   computable at a sampled `x`. Fails for `∃`/`∀`-over-infinite and structural claims.
3. **Bug class (the sharpest limit).** **Catches** *transcription* bugs superbly: wrong
   constant, flipped sign, wrong normalization, dropped coefficient. **Blind to**
   *logical-structure* bugs: quantifier scope and hypothesis strength (`∀ x>1` and the
   faithful `∀ x>0` return the same value at a sampled `x=5`), **vacuity** (an
   unsatisfiable hypothesis), domain-of-validity, and almost-everywhere subtleties (the
   counterexample set is measure zero — sampling never lands on it). It validates
   *values*, not *logical faithfulness*.
4. **Oracle availability.** Strongest when a **separately-authored** reference engine
   already exists (mpmath, LAPACK, a CAS). For a **frontier/novel** result there may be
   no independent implementation; writing the evaluator from the same understanding that
   produced the Lean statement **weakens or destroys the independence** that is the whole
   source of the evidence.
5. **Exactness.** Cleanest for exact / finite-precision-decidable facts; degrades for
   limits, asymptotics, infinite sums and oscillatory integrals — checkable at leading
   order on a finite range, never the actual limit.

> **So: necessary, not sufficient — and only for the right kind of statement.** A green
> `numeric` column means *the values are right*; it says nothing about quantifiers,
> hypotheses, or vacuity. Use it as **one tier** of a layered defense, never the whole
> assurance.

## Mechanics (the numeric-sampling instance; the others are analogous)

1. **Transpile the conclusion to a checkable claim.** Mechanically turn the theorem's
   conclusion into the oracle's language (`|LHS − RHS| < ε`, a CAS `simplify(…)==0`, an
   SMT query) — *reviewable code, no model in the loop*. (FLMF's `fidelity_extract.py`.)
2. **Sample the free variables in their *stated* domain.** Read the hypotheses and sample
   *inside* them — sampling where a hypothesis fails is a spurious failure, not a bug.
   Cover the domain (real/complex grids, random matrices, integer ranges), edges included.
3. **Evaluate independently and gate** to a tolerance (30+ digits where exact).
4. **Bind by anchoring, not by name.** The one subtle step is deciding *which* object a
   Lean symbol denotes. Pin it at a value where rival conventions **disagree**
   (probabilists' `He₃(2) = 2`, never the physicists' `40`; elliptic modulus `k` vs
   parameter `m`), so a convention mismatch surfaces as a discrepancy rather than hiding.
   (FLMF: `oracle_bindings.verify()`.)
5. **Run it in CI, continuously**, so it cannot silently go stale; record outcomes in the
   `numeric` column.

In practice most bring-up "failures" are sampling outside a hypothesis or a transcription
bug — fix the harness, not the theorem. (FLMF's sweep defects were all tool/transcription.)

## Adoption

A project adds a harness with three parts: a **transpiler** for its statement shapes, an
**oracle** (numeric engine / CAS / SMT / exact), and its **bindings** (the anchored
symbol↔oracle map). Shapes and oracle are project-specific — matrices ≠ special functions
≠ combinatorics — but the *pattern* is shared: **transpile → constrain-to-domain →
independent-check → anchor → gate-in-CI.** Lift FLMF's `fidelity_extract.py` /
`oracle_bindings.py` as the reference implementation. Results feed the `numeric` column
([`CORRESPONDENCE_INDEX.md`](CORRESPONDENCE_INDEX.md)) and the F-ladder
([`FIDELITY_REVIEW.md`](FIDELITY_REVIEW.md)) — *as one tier among several*, never alone.

> **Reusable target (future).** A shared *transpile + check* driver with pluggable oracles
> (numeric / CAS / SMT) and per-project bindings, so a project supplies only its
> statement-shape handlers and its oracle map — the generalization of FLMF's extractor
> into hub infrastructure, alongside the assurance CI gate.
