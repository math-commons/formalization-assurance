# Adopting these conventions in a project

A project references this hub and declares its **local settings**; it does not copy
the generic content. Keep per-project docs thin.

## 1. Reference the hub

In the project's `README.md` (and/or a short `docs/CONVENTIONS.md`), add:

> Assurance conventions: this project follows
> [`math-commons/formalization-assurance`](https://github.com/math-commons/formalization-assurance)
> (verification / validation / faithfulness, axiom vetting, `formalization.yaml`,
> comparator). Local settings below.

## 2. Declare local settings

| Setting | Where | Notes |
|---|---|---|
| Vetting strictness level | `docs/vetting/policy.yml` (`vetting_strictness: L1`) | L0–L3 ladder in `VETTING.md`; start L1, raise as you mature |
| Kernel-authoritative axiom set | generated `docs/axiom-report.txt` | the single source of truth for axioms/counts |
| Per-axiom vetting records | `docs/vetting/<AxiomName>.md` | template in this hub's `templates/` |
| Axiom audit | top-level `AXIOM_AUDIT.md` | format in `AXIOM_AUDIT_FORMAT.md` |
| Faithfulness map | `docs/FAITHFULNESS.md` | informal↔formal (a *validation* doc — not "VERIFICATION.md") |
| Acceptance / characterization | `docs/VALIDATION.md` | the ladder up to the categorical certificate |
| Project card | top-level `formalization.yaml` | per `FORMALIZATION_YAML.md` |

## 3. Stand up the machine gates

- **Golden `#print axioms` trace** generated + CI-diffed (fail-with-regenerate).
- **Axiom set + counts generated** from the same source; YAML/README counts derived
  or CI-checked — never hand-authored (`FORMALIZATION_YAML.md`).
- **Vetting coverage** enforced at the declared strictness level (`VETTING.md`).
- **Spec conformance** (if the project targets an external spec): restate the spec
  signatures as `example`s discharged by the project's decls.
- **Comparator** (optional, for released/headline results): `COMPARATOR.md`.

## 4. Naming hygiene (common pitfalls)

- The informal↔formal doc is **`FAITHFULNESS.md`**, not `VERIFICATION.md`
  (faithfulness is validation; verification is the kernel/axiom check).
- Don't ask PRs to *update axiom counts by hand* — that's the generator's job.
- A new or **strengthened** axiom needs a captured vetting record **before** it is
  relied upon downstream.

## Minimal checklist for a new project

1. `formalization.yaml` + top-level `AXIOM_AUDIT.md`.
2. Generated `docs/axiom-report.txt` + CI diff.
3. `docs/vetting/` with `policy.yml` (start `L1`) and the entry template.
4. `docs/FAITHFULNESS.md` + `docs/VALIDATION.md`.
5. README pointer to this hub + the local-settings table.
