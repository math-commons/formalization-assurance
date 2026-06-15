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

## One-line summary

**Backward chaining's spine is "are the assumptions true?" (`VETTING.md`); forward
chaining's spine is "are the statements faithful, and is the corpus covered?"
(`CORRESPONDENCE_INDEX.md` + this).** Proved ≠ faithful; numeric-pass ≠ faithful;
faithful is what makes the library trustworthy.
