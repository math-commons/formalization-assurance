# Fidelity review — vetting that a formalized statement matches its source

The forward-chaining analog of axiom soundness vetting (`VETTING.md`). In
backward chaining the question is *"is this assumed axiom true?"*; in forward
chaining the proofs are real, so the question is **"does this Lean statement
*faithfully* capture the informal source?"** — the dominant forward risk, because a
`proved`, axiom-free theorem can still be the *wrong statement*.

## The failure modes (what a fidelity review checks)

These are the ways a translation goes wrong while still type-checking and proving:

1. **Dropped / weakened hypothesis** — the source requires `Re s > 1`, `A` invertible,
   `f ∈ L¹`; the Lean statement omits it (often making the theorem *easier* and
   *false* as a rendering of the source).
2. **Narrowed or wrong domain** — stated for `ℝ` when the source is `ℂ`; for a special
   case presented as the general one.
3. **Branch / sign / normalization convention** — `log`, `√`, Fourier-transform
   constant, principal value, orientation. Silent and pervasive in special-function
   and matrix-identity corpora.
4. **Convergence / regularity glossed** — `tsum`/`∫` written without the summability
   / integrability the source assumes (or relies on Lean's junk-value convention).
5. **Edge / degenerate cases** — `n = 0`, empty product, singular matrix, poles — does
   the Lean statement say the right thing there, or quietly the wrong thing?
6. **Variable/indexing mismatch** — off-by-one, transposed roles, summation range.

A faithful render gets all of these right; the review's job is to confirm each, per
entry, against the verbatim source.

## Tools (in increasing strength)

- **Numerical / differential cross-check (cheapest, first)** — evaluate the
  formalized expression and diff against an independent reference (mpmath for DLMF,
  numpy on random matrices for the Matrix Cookbook). Catches sign/branch/index/
  normalization errors that no proof will. Record as the `numeric` column in the
  correspondence index. *A numeric pass is necessary, not sufficient* (it doesn't
  catch a dropped hypothesis that happens to hold on the test points).
- **Cross-model statement review** — give a model the **verbatim source** + the Lean
  statement and ask: are the hypotheses, domain, conventions, and edge cases an exact
  match? (Your `latex-code-verification` workflow is this.) Mandatory for any entry
  the numeric check can't fully exercise, and for every `flagged` row.
- **Self-audit against the source** — human/agent line-by-line against the pinned
  corpus text.

## Capture (same discipline as axiom vetting)

For any entry whose fidelity is non-obvious or `flagged`, write a record
`audit/fidelity/<source_id>.md` (template mirrors `templates/vetting-entry.md`), with
YAML front-matter:

```yaml
---
source_id: DLMF:25.2.2
lean_decl: Flmf.Zeta.zeta_eq_eulerProduct
model: gemini-3.1-pro-preview
date: 2026-06-14
checks: [hypotheses, domain, conventions, edge_cases]
numeric: pass@mpmath           # ref + result
verdict: flagged               # faithful | flagged | partial-domain
---
```
Body, verbatim: the **source statement**, the **Lean statement**, the **prompt**, the
**reply**, a one-line digest, and the resolution (what was fixed, or the accepted
restriction). The verdict mirrors into the correspondence index's `fidelity` column.
Trivially-faithful entries (numeric-pass + obvious) can live as an index row with no
separate record.

## Strictness ladder — a per-project knob

Calibrated to contributor sophistication and stakes, like `VETTING.md`'s L0–L3.
Declared in `audit/fidelity/policy.yml` (`fidelity_strictness: F2`); CI reads it.

| Level | CI behavior | Fits |
|---|---|---|
| **F0 — none** | index maintained by hand; no checks | exploratory |
| **F1 — numeric** | every formula-shaped `proved` entry must carry a `numeric: pass@<ref>` (or justified `n/a`); fail otherwise | any computational corpus (special functions, identities) |
| **F2 — numeric + review on gaps** | F1, **plus** every entry the numeric check can't fully exercise, and every `flagged`, needs a `audit/fidelity/` record | multi-contributor; high-fidelity corpora |
| **F3 — full** | F2, plus a fidelity record for **every** in-scope entry; `statement_hash`-style freshness so a re-stated theorem re-opens its review | reference-grade libraries (FLMF aspiration) |

Mechanics that make F1/F2 cheap: a numeric harness emits the `numeric` column; the
correspondence index gives coverage = set-difference, so CI computes
"`proved` entries missing a required numeric/record" automatically — no one
anticipates anything (`FORMALIZATION_YAML.md` "generate, don't hand-author").

## Reviewer-facing affordances (put these in the human digest)

The index + records above are the *machine/audit* side. A human reviewer also needs two
things in the project's curated digest (its `FAITHFULNESS.md` / reviewer guide), because the
kernel cannot catch misformalization and a reviewer's time is the scarce resource. Prior art:
[`rkirov/jacobian-claude`](https://github.com/rkirov/jacobian-claude)'s `docs/REVIEW.md`.

**1. A "subtleties to check" crosswalk — tell the reviewer where to *doubt*.** The usual
crosswalk says "this Lean decl ⟷ this informal claim, and here's the proof idea" — i.e. what's
*true*. Add a column/line per headline naming the **specific place misformalization could hide**
— the failure mode (from the list above) that actually threatens *this* statement:

| Lean | Informal claim | **Subtlety to check** |
|---|---|---|
| `genus_eq_zero_iff_homeo` | genus 0 ⟺ X ≅ S² | **homeomorphism, not biholomorphism**; the metric S² |
| `exists_riemannRoch_divisor` | `l(D) − l(K−D) = deg D + 1 − g` | the germ-quotient `lDim` kills *only* junk |
| `abelJacobi_twoPoint_ne_zero` | AJ(P−Q) ≠ 0, g ≥ 1 | `abelJacobi` is the *honest* path-integral map |

This is the difference between "here's why it's true" and "here's where to look if it isn't" —
the latter is the actual job of a faithfulness review.

**2. A "deliberate design choices (not bugs)" section — tell the reviewer what *not* to flag.**
Formalizations carry intentional conventions that make a statement *look* weaker or stranger
than the textbook's; without a pre-emptive list, a reviewer wastes time re-deriving that each is
fine (or worse, files a non-bug). Enumerate them with the "check it is *equivalent*, not broken"
framing. Recurring kinds:

- **Junk-value conventions** — germs / `toFun` carry arbitrary values at poles/off-domain;
  statements quotient by a "zero" relation or use order/germ language. When a statement looks
  weaker, this is usually why.
- **Carrier/encoding wrappers** — `ULift` to hit a universe-polymorphic signature; a bundled
  hom type; a pair-representation standing in for a missing bundle type.
- **Scalar / instance diamonds** — e.g. `SMul ℝ ℂ` / `restrictScalars` at the manifold layer,
  handled by an elaboration flag (`backward.isDefEq.respectTransparency false`) or explicit
  routing; **elaboration-only, the kernel rechecks**.
- **Typeclass-vs-hypothesis choices** — a side condition carried as `[Fact …]` so downstream
  synthesis fires (a Lean-mechanics choice, not a math restriction).
- **Definitional substitutes** — discrete continuation standing in for an integral; a chosen
  representative for an object defined up to a lattice/equivalence.
- **Axioms as documented inputs, not gaps** — remaining axioms are off the headline closure and
  textbook-standard (vetted), not `sorry`s; "axiom-clean" means `#print axioms` = standard-3.

Both belong in the *curated digest* (bounded, high-signal), not the exhaustive index. Reference
implementation: `mrdouglasny/jacobian-challenge`'s `docs/FAITHFULNESS.md`.

## One-line summary

**Backward chaining's spine is "are the assumptions true?" (`VETTING.md`); forward
chaining's spine is "are the statements faithful, and is the corpus covered?"
(`CORRESPONDENCE_INDEX.md` + this).** Proved ≠ faithful; numeric-pass ≠ faithful;
faithful is what makes the library trustworthy.
