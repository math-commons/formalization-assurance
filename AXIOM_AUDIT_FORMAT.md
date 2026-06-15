# Axiom audit — format convention

Conventions for the per-project axiom audit at `<project>/AXIOM_AUDIT.md` — a
**top-level** file alongside `README.md`, *not* tucked away in `docs/`.

The audit is a primary-visibility document: anyone landing on the repo should see it
immediately. It is the canonical source of truth on (i) which axioms are still
active, (ii) their vetting verdict, (iii) the plan to discharge each, and (iv)
downstream consumers. **Refresh in the same commit that adds, removes, or rewrites
an axiom** — never let the audit drift behind the code.

> The audit holds **verdicts, plans, and consumers**. The *evidence* behind each
> verdict (model + version + verbatim prompt + reply) lives in the per-axiom
> records under `docs/vetting/` — see [`VETTING.md`](VETTING.md). The audit's
> `Vetting` column should link to the record.
>
> The axiom **list and counts are kernel facts** — do not hand-author them; derive
> from the generated `#print axioms` report (see [`FORMALIZATION_YAML.md`](FORMALIZATION_YAML.md)).

## Source codes (the `Vetting` column)

| Code | Meaning |
|------|---------|
| `DT` | Gemini deep-think (slow, high-reasoning pass — `mcp__gemini__deep_think_gemini`) |
| `GR` | Gemini chat review (`mcp__gemini__chat_gemini`) |
| `CX` | Codex (independent re-derivation / cross-check) |
| `LP` | Literature proof with explicit page / theorem reference |
| `SA` | Self-audit (author cross-checked against a textbook by hand) |
| `PR` | Peer review (external mathematician; rare) |

Multiple codes per axiom are common (`DT, LP, SA`). Always record the **model +
version + date** alongside (different model versions occasionally disagree) — in the
linked `docs/vetting/` record, and briefly in the audit's notes.

## Rating scale (the `Rating` column)

| Rating | When to use |
|---|---|
| **Standard** | Well-established textbook fact, multiple references, ≥1 vetting source confirming. |
| **Likely correct** | Checked, consistent with textbooks, but only one vetting source or only `SA`. |
| **Needs review** | Plausible but not yet vetted. Always pair with a planned vet. |
| **Placeholder** | Known to need replacement (type stub / infrastructure). Never downstream-load-bearing. |
| **Flagged** | A concrete concern raised. **Do not consume downstream until resolved**; link the concern. |

## What an axiom *is* here

In these projects an **axiom** is a *vetted, provable theorem with a vetted
discharge plan* — a **staging point**, not a fundamental assumption. Each entry is
(1) a standard textbook fact with citation, (2) reviewed for type correctness,
hypothesis sufficiency, non-vacuity, and **satisfiability**, and (3) accompanied by
a concrete plan to discharge it into a Lean theorem. The goal is for every entry to
become a proved theorem.

## Doc structure

```
# Axiom audit — <project>
*Last updated YYYY-MM-DD.*

## Purpose            — the staging-point framing (above); pointer to this format doc.
**Active axioms: N.** — one-line summary (count derived from the generated report).
## Conventions        — 1-line pointer here, or a short source-codes/ratings recap.
## Summary            — clusters, totals, anything notable.
## Active axioms       — per-axiom entries (headed or table form, see below).
## Recently discharged — table of eliminated axioms + where the proof now lives.
## Inactive / future   — declared-but-not-in-active-build axioms (listed, not counted).
## Verification        — the shell command(s) to re-derive the count/list.
```

### Per-axiom entry (headed form)

```
### <Axiom name>
**File**: `<path>:<line>`
**Statement** (informal): one paragraph, plain language.
**Vetting**: **<Rating>** (`<codes>`) — reference + brief justification → [vetting](docs/vetting/<Axiom>.md)
**Strategy / Plan**: how to discharge + estimate; if >3 lines, link a discharge-plan doc.
**Consumers**: downstream theorems that load-bear on this axiom.
```

### Per-axiom row (table form, for larger projects)

```
| Axiom | File:Line | Reference | Rating | Vetting | Strategy / Plan | Consumers |
```

Use anchored markdown links for `File:Line`. If a plan exceeds ~3 lines, break out a
`docs/<axiom-name>-discharge-plan.md` (goal + effort, API to reuse, step-by-step
outline with lemma signatures, acceptance criteria) and link it.

## Updating discipline

- **Same-commit rule**: add/remove/modify an axiom → update the audit in the *same*
  commit. (Counts themselves are CI-generated, not hand-edited — see the rule above.)
- **Per-discharge**: when an axiom becomes a theorem, move its row to *Recently
  discharged* with the strategy + proof file:line. Keep its `docs/vetting/` record
  (`discharged: true`) for provenance.
- **Per-vet**: when a vet completes, add the source code + link the `docs/vetting/`
  record; re-vet (new record, `superseded_by` the old) when a statement is
  **strengthened**.
- **On rename/move**: chase every reference; the audit names the axioms.
