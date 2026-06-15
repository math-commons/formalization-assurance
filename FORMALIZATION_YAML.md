# `formalization.yaml` — the project card, and the generate-don't-author rule

The **Mathlib Initiative** standardized metadata schema
(<https://github.com/mathlib-initiative/formalization.yaml>) is a single
machine-readable card describing a formalization project; it is also the artifact
the **lean-eval / comparator** submission flow reads. Every `math-commons`
formalization project keeps one at its top level.

## Sections

- **`project`** — name, authors, license.
- **`sources`** — every input (the spec/challenge, prior formalizations, textbooks),
  each with `title / authors / id / type`; primary sources also carry
  `license / author_contacted / prior_work`.
- **`status`** — the machine accounting:
  - `scope`, `sorry_count`, `sorry_in_definitions`,
  - **`axioms`** — standard-3 + project axioms (annotated, critical vs not),
  - **`main_results`** — per flagship declaration: `file`, `sorry_count`, the exact
    `axioms` set, `literature_dependencies`, and (where applicable) `comparator_config`.
- **`automation`** — methods (models, framework, tool_setup, cost, prompting_notes),
  spend, and the AI-authorship disclosure.
- **`fidelity.divergences`** — how the formalization differs from the literature /
  from prior formalizations (representation choices; remaining literature deps).
- **`review`** — review status, reviewers, comparator notes; can aggregate the
  `audit/vetting/` front-matter.
- **`alignment.statements`** — a `source ↔ lean ↔ module ↔ status ↔ note` table
  (the machine-readable faithfulness map).
- **`acknowledgements`**.

So `formalization.yaml` is the standardized home for the **machine accounting**
(`status`, `main_results[].axioms`) and a **machine-readable faithfulness alignment**
(`alignment.statements`). The project's prose docs (`FAITHFULNESS.md`,
`VALIDATION.md`, `AXIOM_AUDIT.md`) are the human companions.

## The rule: generate machine facts, never hand-author them

The axiom *list*, axiom *counts*, and sorry *counts* are **kernel facts**. Asking
humans or agents to fill them by hand fails in any multi-contributor setting: no one
can *anticipate* the post-merge count when several PRs discharge axioms
concurrently, so the YAML, the README, and the audit drift apart.

**Therefore:**

1. **One source of truth — the kernel.** A generator (e.g. an `axiom_report.lean`
   driver over `#print axioms`) emits the golden per-headline trace **and** the
   distinct project-axiom set + counts in machine-readable form.
2. **Derive or CI-check the rest.** `status.axioms`, `main_results[].axioms`, and any
   README / `AXIOM_AUDIT` counts are either *generated from* that, or diffed against
   it and **failed-with-regenerate** in CI (the pattern that already works for the
   golden `#print axioms` trace).
3. **Take counts off the contributor.** The PR obligation shrinks to *vetting* any
   new/changed axiom (see `VETTING.md`); regeneration + CI own the numbers.

What stays hand-maintained is **judgment**, not facts: the per-axiom *vetting*
(is it true?), the *discharge plan*, the *fidelity divergences*, the *scope* prose.

> Symptom to watch for: `status.axioms` listing `noncomputable def`s as axioms, or
> omitting real ones, or a README count that disagrees with `audit/axiom-report.txt`.
> All are the same root cause — a kernel fact being hand-authored. Fix the
> generation, not the entry.
