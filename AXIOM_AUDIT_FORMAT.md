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
> records under `audit/vetting/` — see [`VETTING.md`](VETTING.md). The audit's
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
linked `audit/vetting/` record, and briefly in the audit's notes.

## Rating scale (the `Rating` column)

| Rating | When to use |
|---|---|
| **Standard** | Well-established textbook fact, multiple references, ≥1 vetting source confirming. |
| **Likely correct** | Checked, consistent with textbooks, but only one vetting source or only `SA`. |
| **Needs review** | Plausible but not yet vetted. Always pair with a planned vet. |
| **Placeholder** | Known to need replacement (type stub / infrastructure). Never downstream-load-bearing. |
| **Flagged** | A concrete concern raised. **Do not consume downstream until resolved**; link the concern. |

## Closure status (orthogonal to the rating)

Rating measures *trust*; **closure status** measures *where the axiom sits relative to
the headline theorems*. Conflating the two is the single biggest source of stale,
slightly-false status prose ("axiom-free", "the only remaining axiom") — track it as a
separate, kernel-derived field:

| Status | Meaning | Kernel test |
|---|---|---|
| **discharged** | now a theorem; no longer declared | name absent from the axiom set |
| **declared-on-closure** | declared **and** appears in a headline's `#print axioms` | in the headline closure |
| **declared-off-closure** | declared but in **no** headline closure — scaffolding / off-path witness kept for a non-headline story | declared, but 0 headline mentions |

`declared-off-closure` is **rare but real** (e.g. a polarization form or a cycle-basis
scaffold kept after the headlines were rerouted off it). It is exactly what lets you
honestly say *"every headline is `#print axioms` = standard-3"* while the kernel axiom
**count is still > 0** — so state both numbers and never collapse them. All three states
are kernel facts: derive them from the generated `#print axioms` report + the headline
closure list, don't assert them in prose.

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
**Vetting**: **<Rating>** (`<codes>`) — reference + brief justification → [vetting](audit/vetting/<Axiom>.md)
**Strategy / Plan**: how to discharge + estimate; if >3 lines, link a discharge-plan doc.
**Consumers**: downstream theorems that load-bear on this axiom.
```

### Per-axiom row (table form, for larger projects)

```
| Axiom | File:Line | Reference | Rating | Vetting | Strategy / Plan | Consumers |
```

Use anchored markdown links for `File:Line`. If a plan exceeds ~3 lines, break out a
`audit/<axiom-name>-discharge-plan.md` (goal + effort, API to reuse, step-by-step
outline with lemma signatures, acceptance criteria) and link it.

## Updating discipline

- **Same-commit rule**: add/remove/modify an axiom → update the audit in the *same*
  commit. (Counts themselves are CI-generated, not hand-edited — see the rule above.)
- **Per-discharge**: when an axiom becomes a theorem, move its row to *Recently
  discharged* with the strategy + proof file:line. Keep its `audit/vetting/` record
  (`discharged: true`) for provenance.
- **Per-vet**: when a vet completes, add the source code + link the `audit/vetting/`
  record; re-vet (new record, `superseded_by` the old) when a statement is
  **strengthened**.
- **On rename/move**: chase every reference; the audit names the axioms.

## Auditing presence & guarding drift

Two failure modes seen repeatedly in agent-driven projects; both are cheap to prevent.

**Audit presence by extracting the axiom *set*, not per-name pattern-matching.** When
checking "is axiom X still live?", do **not** grep for each name individually — it is a
footgun:
- `git grep -E` silently ignores `\b` word boundaries (the pattern matches nothing → a
  live axiom reads as discharged);
- a trailing-space pattern (`axiom X `) misses declarations whose name ends the line
  (multi-line signatures) → false "discharged".

Instead, **extract the actual set once** and set-match against it:

```bash
# the live axiom set (ground truth)
git grep -hE "^(noncomputable )?axiom " -- '<lib>' \
  | sed -E 's/^(noncomputable )?axiom +([A-Za-z0-9_]+).*/\2/' | sort -u
# better, when a build exists: enumerate from `#print axioms` of the headline set
```

Then `comm`/`grep -xF` each tracked name against that set. (Authoritative form: the
generated `#print axioms` report — see [`FORMALIZATION_YAML.md`](FORMALIZATION_YAML.md).)

**Add a drift guard.** Status that is *hand-written in prose* drifts behind the kernel —
the same discharge fact ends up wrong in the README, the faithfulness digest, planning
docs, **and the issue tracker**, all lagging one merged PR. The kernel ledger
(generated `audit/axiom-report.txt`, CI-diffed) is the single source of truth; guard
everything else against it:
- **CI check**: grep the prose docs (and open **issue titles**) for axiom names that are
  **absent** from the live axiom set; fail/flag any hit (a doc or open issue naming a
  discharged axiom as if active).
- **Link axiom ⟷ tracker issue**; discharge → close is then mechanical (a still-open
  issue whose axiom is gone is a drift-guard hit).
- Phrase headline claims against **closure status**, not raw counts: "every headline is
  `#print axioms` = standard-3" (closure) is true even when "N declared axioms" (count)
  is > 0 — see *Closure status* above.
