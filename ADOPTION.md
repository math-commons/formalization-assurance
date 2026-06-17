# Adopting these conventions in a project

A project references this hub and declares its **local settings**; it does not copy
the generic content. Keep per-project docs thin.

## 1. Reference the hub

In the project's `README.md` (and/or a short `audit/CONVENTIONS.md`), add:

> Assurance conventions: this project follows
> [`math-commons/formalization-assurance`](https://github.com/math-commons/formalization-assurance)
> (verification / validation / faithfulness, axiom vetting, `formalization.yaml`,
> comparator). Local settings below.

## 2. Declare local settings

| Setting | Where | Notes |
|---|---|---|
| Vetting strictness level | `audit/vetting/policy.yml` (`vetting_strictness: L1`) | L0–L3 ladder in `VETTING.md`; start L1, raise as you mature |
| Kernel-authoritative axiom set | generated `audit/axiom-report.txt` | the single source of truth for axioms/counts |
| Per-axiom vetting records | `audit/vetting/<AxiomName>.md` | template in this hub's `templates/` |
| Axiom audit | top-level `AXIOM_AUDIT.md` | format in `AXIOM_AUDIT_FORMAT.md` |
| Faithfulness map | `audit/FAITHFULNESS.md` | informal↔formal (a *validation* doc — not "VERIFICATION.md") |
| Acceptance / characterization | `audit/VALIDATION.md` | the ladder up to the categorical certificate |
| Project card | top-level `formalization.yaml` | per `FORMALIZATION_YAML.md` |

> **Why `audit/` and not `docs/`?** The evidence artifacts go in a dedicated
> top-level **`audit/`** directory — *not* `docs/`. In most Lean projects `docs/`
> is the GitHub Pages / leanblueprint **site root**: a Pages pipeline (e.g.
> `leanprover-community/docgen-action`) detects a top-level `docs/` and Jekyll-builds
> it, so dropping bare `.md`/`.txt`/`.yml` evidence files into a `docs/` with no
> `Gemfile` **breaks the Pages deploy**. `audit/` is a neutral home that never
> collides and keeps the evidence off the published site. The two project *cards* —
> `formalization.yaml` and `AXIOM_AUDIT.md` — stay at the **repo root** (the
> comparator / lean-eval flow reads `formalization.yaml` from root, and the audit is
> the top-level human entry point). If your `docs/` is already a real Jekyll site
> (with a `Gemfile`) you *may* keep artifacts there, but `audit/` is recommended so
> the evidence isn't published.

## 3. Stand up the machine gates

**The fast path — the shared reusable workflow.** Build + axiom-report-in-sync +
sorry-confinement are implemented once in this hub's
[`.github/workflows/assure.yml`](.github/workflows/assure.yml). Adopt them by copying
[`templates/assurance.caller.yml`](templates/assurance.caller.yml) into your repo as
`.github/workflows/assurance.yml` — a one-line `uses:` of the shared workflow. It reads
`audit/vetting/policy.yml` for strictness, so the gates **warn at L0/L1 and enforce at
L2/L3**: adopt at L1 to see your real state, raise to L2 to lock it down. Drop in
[`templates/sorry-allowlist.txt`](templates/sorry-allowlist.txt) as `audit/sorry-allowlist.txt`
and provide your project's `#print axioms` generator at `audit/axiom_report.lean` (+ the
committed `audit/axiom-report.txt` golden trace). *(Private hub: enable Settings → Actions →
"Allow … access to components in private repositories" so other repos can call it.)*

What the gates establish (and where the convention lives):

- **Golden `#print axioms` trace** generated + CI-diffed (fail-with-regenerate) — *in the
  reusable workflow.*
- **Axiom set + counts generated** from the same source; YAML/README counts derived or
  CI-checked — never hand-authored (`FORMALIZATION_YAML.md`).
- **Sorry-confinement** — only allowlisted files may carry `sorry`; the core stays
  sorry-free — *in the reusable workflow.*
- **Vetting coverage** enforced at the declared strictness level (`VETTING.md`). *(Coverage
  enforcement — every axiom has a vetting record — is not yet in the reusable workflow; for
  now it's the L2/L3 manual gate. Tracked as a follow-up.)*
- **Spec conformance** (if the project targets an external spec): restate the spec
  signatures as `example`s discharged by the project's decls.
- **Comparator** (optional, for released/headline results): `COMPARATOR.md`.

## 4. Naming hygiene (common pitfalls)

- The informal↔formal doc is **`FAITHFULNESS.md`**, not `VERIFICATION.md`
  (faithfulness is validation; verification is the kernel/axiom check).
- Don't ask PRs to *update axiom counts by hand* — that's the generator's job.
- A new or **strengthened** axiom needs a captured vetting record **before** it is
  relied upon downstream.
- Put evidence artifacts in **`audit/`**, never a bare top-level `docs/` — a
  Pages/blueprint pipeline will try to Jekyll-build `docs/` and fail without a
  `Gemfile` (see §2). Keep `formalization.yaml` and `AXIOM_AUDIT.md` at the repo root.

## Minimal checklist for a new project

1. `formalization.yaml` + top-level `AXIOM_AUDIT.md`.
2. Generated `audit/axiom-report.txt` + CI diff.
3. `audit/vetting/` with `policy.yml` (start `L1`) and the entry template.
4. `audit/FAITHFULNESS.md` + `audit/VALIDATION.md`.
5. README pointer to this hub + the local-settings table.
