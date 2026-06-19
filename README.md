# formalization-assurance

These are conventions for **assuring** Lean 4 formalization projects:
how we establish, and how a reader can check, that a formalization is correct.

A proof assistant's kernel certifies that **proofs** are valid. It does **not**
certify that a *definition means what its name claims*, that a stated theorem is
*non-vacuous*, or that an assumed `axiom` is *true*. A development can be
`sorry`-free, axiom-clean, and green in CI while still formalizing the **wrong
object** or resting on a **false axiom**. These conventions close that gap with
artifacts a mathematician can audit **without reading the proofs**.

> **New here?** This page is the map. For the conceptual model, read
> [`VERIFICATION_VALIDATION.md`](VERIFICATION_VALIDATION.md). To wire the conventions
> into a project, go to [`ADOPTION.md`](ADOPTION.md). The per-document index is the
> **Documents** table below.

## The assurance model — three layers

1. **Verification** — *are the proofs valid relative to explicit assumptions?*
   The kernel (`lake build`) + the axiom certificate (`#print axioms`, CI-pinned).
   In a proof assistant this is **largely automatic**.
2. **Assumption review** — *are the assumed axioms true?* The kernel cannot do
   this. Established by **soundness review** (literature citations + cross-model 
   vetting + self-audit), captured as durable per-axiom records.
3. **Validation** — *did we formalize the right object?* Two sub-layers:
   **(a) faithfulness** (the informal↔formal correspondence) and
   **(b) the acceptance ladder** (theorems that pin the definitions, up to a
   categorical "the specification has exactly one model" certificate — see the
   ladder in [`VERIFICATION_VALIDATION.md`](VERIFICATION_VALIDATION.md)).

## Two modes — forward and backward chaining

Projects sit on a spectrum, and the dominant risk — hence the **assurance spine**, the
documents that carry the assurance weight for that project — differs:

- **Forward-chaining** — formalize a large external *corpus* bottom-up (a classic
  reference or textbook such as Rudin's *Principles of Mathematical Analysis*,
  Abramowitz & Stegun's *Handbook of Mathematical Functions* / NIST's **DLMF**, or
  Ireland & Rosen's *A Classical Introduction to Modern Number Theory*): build up
  from foundations, ≈ no axioms.
  Central risk = **unfaithful statements** + **incomplete coverage**. Spine:
  `CORRESPONDENCE_INDEX` + `FIDELITY_REVIEW`.
- **Backward-chaining** — a posed *target spec* (e.g. a challenge): stub unknowns as
  `axiom`s, discharge top-down. Central risk = **unsound axioms**. Spine:
  `AXIOM_AUDIT_FORMAT` + `VETTING` (assumption review) + the categorical certificate
  in `VERIFICATION_VALIDATION`.

Both share verification (kernel + axiom certificate), faithfulness,
`formalization.yaml`, and the comparator; they differ in which review layer carries
the weight. Many projects mix both modes.

> **Spine in one line:** forward chaining asks *"are the statements faithful, and is
> the corpus covered?"*; backward chaining asks *"are the assumptions true?"*
> Note `proved ≠ faithful`: a sorry-free, axiom-free theorem can still be the wrong
> statement.

## Documents

| File | What it covers |
|---|---|
| [`VERIFICATION_VALIDATION.md`](VERIFICATION_VALIDATION.md) | the V&V terminology, the faithfulness/characterization layers, operational-vs-categorical specs |
| [`VETTING.md`](VETTING.md) | *(backward)* capturing axiom **soundness reviews** as durable records; the strictness ladder **L0–L3**; CI policy |
| [`AXIOM_AUDIT_FORMAT.md`](AXIOM_AUDIT_FORMAT.md) | *(backward)* the per-project `AXIOM_AUDIT.md` format (ratings, source codes, discharge plans) |
| [`CORRESPONDENCE_INDEX.md`](CORRESPONDENCE_INDEX.md) | *(forward)* the machine-readable source↔Lean coverage + faithfulness index (corpus-scale); schema + FLMF/DLMF example |
| [`FIDELITY_REVIEW.md`](FIDELITY_REVIEW.md) | *(forward)* vetting that a formalized statement **matches its source**; numeric cross-checks; strictness ladder **F0–F3** |
| [`NUMERICAL_VALIDATION.md`](NUMERICAL_VALIDATION.md) | **independent-oracle cross-validation** — check a statement against an independent oracle (numeric sampling, CAS, SMT, exact); catches value / sign / constant / normalization bugs the kernel can't see, **blind** to quantifier / hypothesis / vacuity. A *partial* tier; generalizes FLMF's mpmath oracle |
| [`OBJECT_CONTRACTS.md`](OBJECT_CONTRACTS.md) | *(validation)* per-**definition** contract cards — informal↔Lean signature, an **anti-degeneracy** clause, and a `known_values` **test matrix** (instance → expected → theorem → status); differential testing for definitions, status read from `#print axioms`. The definition-side analogue of axiom vetting |
| [`FORMALIZATION_YAML.md`](FORMALIZATION_YAML.md) | the Mathlib-Initiative `formalization.yaml` project card + the "generate, don't hand-author" rule |
| [`COMPARATOR.md`](COMPARATOR.md) | external kernel-replay verification (Lean FRO comparator) protocol + registry |
| [`ADOPTION.md`](ADOPTION.md) | how a project adopts these conventions and declares its local settings |
| [`OPEN_QUESTIONS.md`](OPEN_QUESTIONS.md) | known design tensions and limitations, with who raised them and where we stand |
| [`templates/`](templates/) | copy-in templates: the assurance CI **caller workflow** + **sorry-allowlist**, the strictness **`policy.yml`**, the **vetting entry**, the **object-contract card**, and the **`AXIOM_AUDIT.md` skeleton** |

## Core principles

1. **Generate machine facts; never hand-author them.** Axiom lists, axiom/sorry
   counts are *kernel facts* — generate them from `#print axioms`, enforce in CI,
   and never ask a contributor to *anticipate* a post-merge count. (Hand-authored
   counts drift the moment two PRs land concurrently.)
2. **Capture evidence, not just verdicts.** A soundness review saves the verbatim
   model + version + prompt + reply, not a paraphrase — so it is reproducible.
3. **Calibrate strictness to contributors.** Enforcement is a per-project knob
   (L0–L3 in `VETTING.md`), not a one-size mandate; tighten it as a project's
   contributor base and stakes grow.
4. **Characterize, don't just accumulate.** The strongest validation is a
   *categorical* certificate ("the spec has exactly one model"), not the longest
   checklist; an operational API is typically non-categorical.

## Relationship to other sources

- The **methodology paper** (narrative + justification) is the publication; this
  repo is the **operational spec** projects implement.
- An agent session's global config can carry a one-line pointer here so these
  conventions are auto-consulted.
- Lean *code* conventions (naming, style, tactics) live elsewhere; this repo is
  about *project-level assurance*, not code style.

## Status

v1 scaffold. First adopter:
[`mrdouglasny/jacobian-challenge`](https://github.com/mrdouglasny/jacobian-challenge).

## License

Apache License 2.0 (see [`LICENSE`](LICENSE) and [`NOTICE`](NOTICE)). The conventions,
templates, and CI here are free to adopt, modify, and redistribute under those terms.
