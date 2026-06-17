# Correspondence index — forward-chaining coverage + faithfulness at scale

For **forward-chaining** projects (formalizing a large external *corpus* —
DLMF/FLMF, the Matrix Cookbook, a textbook — bottom-up), the central assurance
artifact is not the axiom audit (axioms ≈ 0) but a **correspondence index**: a
comprehensive, machine-readable, **source-ID-keyed** table mapping every corpus
entry to its Lean declaration (or marking it absent).

It is three things at once:
- **coverage tracker** — what fraction of the corpus is done, and where the gaps are;
- **faithfulness map** — which Lean decl claims to be which informal statement;
- **provenance record** — the citable external source for each result.

The human-readable [`FAITHFULNESS.md`](VERIFICATION_VALIDATION.md) stays the
*curated digest* of the load-bearing correspondences; the index is the *exhaustive*
machine artifact. (In `formalization.yaml`, `alignment.statements` is the seed of
this — the index is its corpus-scale generalization, kept as its own file/DB.)

## Why "proved" is not the whole story

A result can be **proved (sorry-free, axiom-free) yet unfaithful** — the Lean
statement may drop a hypothesis, narrow a domain, or get a branch/sign wrong, so it
is a true theorem about the *wrong* statement. The index therefore tracks
**`status`** (is it proved?) and **`fidelity`** (does the statement match the
source?) as **separate columns**. Faithfulness review is `FIDELITY_REVIEW.md`.

## Schema (TSV / DB columns)

One row per corpus entry:

| Column | Meaning |
|---|---|
| `source_id` | canonical corpus ID — `DLMF:25.12.10`, `MatrixCookbook:eq43`, `Forster:Thm21.7` |
| `source_ref` | human citation + URL/version (the pinned corpus release) |
| `informal` | the source statement, verbatim/LaTeX (or a stable pointer to it) |
| `lean_decl` | fully-qualified Lean name, or empty if absent |
| `module` | `file:line` |
| `status` | `proved` \| `statement-only` \| `sorry` \| `partial` \| `absent` \| `n/a` |
| `fidelity` | `faithful` \| `flagged` \| `partial-domain` \| `unchecked` (+ link to the fidelity record) |
| `numeric` | `pass@<ref>` (cross-check ran and matched) \| `fail` (ran, mismatch) \| `none` (not run) \| `n/a` (no numeric content to check, e.g. an existence statement) |
| `deps` | key dependencies; **axioms used** (should be standard-3 only) |
| `notes` | free text |

**`status` taxonomy**
- `absent` — not started.
- `statement-only` — the Lean statement exists; proof is `sorry`/TODO.
- `sorry` — proof attempted, incomplete.
- `partial` — restricted case/domain only (note the restriction).
- `proved` — sorry-free (and ideally axiom-free).
- `n/a` — corpus entry not in scope / not a formalizable statement.

**Coverage** = `proved ∧ faithful` rows ÷ total in-scope corpus rows. Report it; never
let `proved` alone be read as "done" (see fidelity).

## What's generated vs hand-maintained

Same principle as the axiom counts (`FORMALIZATION_YAML.md`): **generate the machine
columns, hand-maintain only judgment.**
- *Generated* from the build/kernel + a test harness: `lean_decl`/`module` presence,
  `status` (sorry-free?), `deps`/axioms, `numeric` (from the cross-check harness).
- *Hand-maintained* (the corpus side + judgment): `source_id`, `source_ref`,
  `informal`, and `fidelity` (the verdict — backed by a `FIDELITY_REVIEW` record).

So coverage and axiom-cleanliness can't drift; only the corpus enumeration and the
fidelity verdicts are authored, and those change rarely.

## Worked example — FLMF / DLMF

The DLMF (Digital Library of Mathematical Functions) is the ideal forward target:
every entry already has a **canonical ID** (chapter.section.number) and a fixed
release version, and there is an independent reference implementation (mpmath) for
numerical cross-checks.

*Shown space-padded for readability; the real file is **tab-separated**. The trailing
`# …` on the second row is the `notes` column.*

```
source_id     source_ref                      informal              lean_decl                         module                 status  fidelity  numeric        deps   notes
DLMF:25.2.1   DLMF rel.1.2.x §25.2 (ζ series) ζ(s)=∑ n^{-s}, Re s>1 Flmf.Zeta.zeta_eq_tsum            Flmf/Zeta/Series.lean  proved  faithful  pass@mpmath    std-3
DLMF:25.2.2   DLMF §25.2                       ζ(s) Euler product    Flmf.Zeta.zeta_eq_eulerProduct    Flmf/Zeta/Euler.lean   proved  flagged   pass@mpmath    std-3   # domain Re s>1 missing? see fidelity
DLMF:25.4.1   DLMF §25.4                       ζ functional equation —                                 —                      absent  unchecked none           —
```

- `fidelity: flagged` on a `proved` row is the exact case the index exists to surface
  — a real theorem whose *statement* may not match the DLMF entry (here, a
  domain-hypothesis question), routed to `FIDELITY_REVIEW`.
- `numeric: pass@mpmath` is per-formula evidence the right function was formalized.
- The Matrix Cookbook analog keys `source_id` on its equation numbers and runs
  `numeric` against numpy on random matrices.

## Relationship to the other artifacts

- `FAITHFULNESS.md` — curated human digest of the headline correspondences.
- `FIDELITY_REVIEW.md` — the *evidence* behind each `fidelity` verdict (the forward
  analog of `VETTING.md`'s axiom records).
- `AXIOM_AUDIT.md` — degenerates to a tripwire here (corpus work should be axiom-free).
- `formalization.yaml › alignment.statements` — can be generated from the index.
