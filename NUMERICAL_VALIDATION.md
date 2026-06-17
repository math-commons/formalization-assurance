# Numerical validation by parameter sampling

*A general assurance technique. When a theorem states a **parametric** fact — an
identity, equation, or inequality with free variables ranging over a domain — sample
concrete points, evaluate the statement with an **independent** computation
(numerics / CAS / decision procedure / Monte Carlo), and check it holds. Disagreement
exposes a faithfulness bug the kernel cannot see. This generalizes FLMF's mpmath
fidelity oracle from special functions to **many** projects — not all, but any whose
theorems include parametric facts you can evaluate.*

## Why it works

The kernel certifies the *proof*, not that the *statement* says what you meant (the
F1/F2 failures in [`VERIFICATION_VALIDATION.md`](VERIFICATION_VALIDATION.md)). A wrong
constant, a flipped sign, the wrong normalization convention, or a hypothesis that is
silently too strong or too weak **compiles and proves exactly like the truth**. An
independent numerical evaluation at sampled parameters catches these — precisely
*because* the evaluator shares no code, language, or author with the Lean statement:
agreement to many digits is strong evidence of faithfulness, and a single disagreement
is a definite bug on our side (the statement or its transcription).

This is the `numeric` column of [`CORRESPONDENCE_INDEX.md`](CORRESPONDENCE_INDEX.md) and
the numeric tier of [`FIDELITY_REVIEW.md`](FIDELITY_REVIEW.md), lifted into a reusable,
project-agnostic technique.

## When it applies — and when it doesn't

**Applies** whenever the statement has *evaluable* content at concrete parameters:

- special-function identities / values / asymptotics (FLMF vs mpmath — the original);
- matrix identities (sample random matrices; evaluate both sides);
- combinatorial / number-theoretic identities (sample integer parameters);
- analytic inequalities and bounds (sample points in the stated domain);
- probability / statistics identities (exact evaluation or Monte Carlo at sampled parameters);
- any `LHS(x) = RHS(x)` or predicate `P(x)` whose sides you can compute at a sampled `x`.

**Does not apply** — use the other F-ladder tiers / the acceptance ladder instead — to
purely structural statements with no numeric content: existence / uniqueness, universal
properties, statements about non-computable objects, topological or categorical facts.
These are the `numeric: n/a` rows.

> So: **not every project, but many.** The test is simply *"can I evaluate this
> statement at a sampled parameter point?"* If yes, sample it.

## Mechanics (generalized from FLMF)

1. **Transpile the conclusion to a computable claim.** Mechanically turn the theorem's
   conclusion into a numeric/symbolic assertion (`|LHS − RHS| < ε`, `bound holds`, …) —
   *reviewable code, no model in the loop*. (FLMF's `fidelity_extract.py` does this over
   its `-- source`-tagged theorems.)
2. **Sample the free variables in their *stated* domain.** Read the hypotheses and sample
   *inside* them — sampling where a hypothesis fails is a spurious failure, not a bug.
   Cover the domain (real/complex grids, random matrices, integer ranges), edges included.
3. **Evaluate independently and gate.** Compute both sides with a high-precision or
   independent evaluator (mpmath, numpy/scipy, a CAS, exact arithmetic, Monte Carlo) and
   check agreement to a tolerance (30+ digits where exact).
4. **Bind by anchoring, not by name.** The one subtle step is deciding *which* numeric
   object a Lean symbol denotes. Pin it at a value where rival conventions **disagree**
   (probabilists' `He₃(2) = 2`, never the physicists' `40`; elliptic modulus `k` vs
   parameter `m`), so a convention mismatch surfaces as a numeric discrepancy instead of
   hiding as a silent factor. (FLMF: `oracle_bindings.verify()`.)
5. **Run it in CI, continuously**, so it cannot silently go stale; record outcomes in the
   `numeric` column.

## What a pass means — and doesn't

- **Necessary, not sufficient.** A number cannot see quantifier scope or vacuity: a
  strictly weaker `∀ x>1` returns the same value at a sampled `x=5` as the faithful
  `∀ x>0`. Sampling validates the *formula*; hypothesis strength and vacuity need the
  other tiers (witnesses / back-translation).
- **Trusts the evaluator + the transcription.** Mitigated by the evaluator's independence
  and by keeping the transcription reviewable code (or, for the irregular tail, a *logged,
  decorrelated-model* transcription the numeric judge still gates — the model parses,
  the evaluator judges).
- **Sample the stated domain.** In practice most bring-up "failures" are sampling outside
  a hypothesis or a transcription bug — fix the harness, not the theorem. (FLMF's
  experience: the sweep's defects were all tool/transcription, surfaced and fixed.)
- **Asymptotics / limits** are checked at leading order on a sampled range, not at the
  limit.

## Adoption

A project adds a `numeric` harness with three parts: a **transpiler** for its statement
shapes, an **evaluator**, and its **oracle bindings** (the anchored symbol↔evaluator map).
The shapes and evaluator are project-specific — matrices ≠ special functions ≠
combinatorics — but the *pattern* is shared: **transpile → sample-in-domain →
independent-evaluate → anchor → gate-in-CI.** Lift FLMF's `fidelity_extract.py` /
`oracle_bindings.py` as the reference implementation. Results feed the `numeric` column
([`CORRESPONDENCE_INDEX.md`](CORRESPONDENCE_INDEX.md)) and the F-ladder
([`FIDELITY_REVIEW.md`](FIDELITY_REVIEW.md)).

> **Reusable target (future).** A shared *transpile + sample + compare* driver with
> pluggable evaluators and per-project bindings, so a project supplies only its
> statement-shape handlers and its oracle map — the generalization of FLMF's extractor
> into hub infrastructure, alongside the assurance CI gate.
