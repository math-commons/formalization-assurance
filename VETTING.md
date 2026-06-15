# Axiom soundness vetting — capturing the evidence

The Lean kernel verifies *proofs*; it cannot verify that an `axiom` is **true**. A
false axiom type-checks and passes CI while making the kernel inconsistent. So the
truth / satisfiability of every project axiom is established separately by
**soundness review** — the *assumption-review* layer (see
`VERIFICATION_VALIDATION.md`), distinct from verification (which axioms are
assumed) and validation (is the object right).

A verdict alone is not enough. `AXIOM_AUDIT.md` records the *rating + source code +
a one-line reasoning digest*; it does **not** preserve the *evidence* (which model
and version, the exact prompt, the full reply). That evidence otherwise lives only
in ephemeral, often gitignored, MCP logs. **This convention captures it.**

## What to capture

One **vetting record per axiom**, in `docs/vetting/<AxiomName>.md`, with one entry
per *vetting event* (re-vettings append; newest first). Each entry — see
[`templates/vetting-entry.md`](templates/vetting-entry.md) — carries a
machine-readable YAML front-matter header plus a verbatim body:

```yaml
---
axiom: AX_Foo
file: Jacobians/Axioms/Foo.lean:88
statement_hash: sha256:…        # of the normalized axiom statement (for staleness)
model: gemini-3.1-pro-preview
tool: mcp__gemini__deep_think_gemini
source_code: DT                 # DT/CX/GR/LP/SA/PR (see AXIOM_AUDIT_FORMAT.md)
date: 2026-06-09
questions: [typing, strength, non-vacuity, satisfiability]
verdict: SATISFIABLE_FAITHFUL   # | FLAGGED | …
rating: Likely correct          # Standard | Likely correct | Needs review | Flagged | Placeholder
discharged: false               # true if since proved; keep the record for provenance
superseded_by: null             # date of a later entry if the statement changed
---
```

Body (verbatim, never paraphrased): the **axiom statement** vetted, the **prompt**
sent, the **reply** received (excerpt only boilerplate), a one- or two-line
**reasoning digest** (mirrored into `AXIOM_AUDIT.md`), and any **conditions /
follow-ups**.

Rules:
- **Verbatim, not paraphrased.** Models are nondeterministic; the saved reply is
  the evidence. Do not rely on gitignored session logs — copy the exchange in.
- **One focused query per axiom** (not a combined batch), so each record is
  self-contained.
- **Re-vet on strengthening.** A *stronger statement at the same axiom count* is
  the dangerous case: re-vet and mark the prior entry `superseded_by`.
- **Keep records after discharge** (`discharged: true`) for provenance.
- **Link from `AXIOM_AUDIT.md`**: each row points to its vetting file.

## Strictness ladder — a per-project knob

Enforcement is calibrated to a project's contributor sophistication and stakes, not
mandated. A project declares one level in [`templates/policy.yml`](templates/policy.yml)
(`vetting_strictness: L2`), which CI reads.

| Level | CI behavior | Fits |
|---|---|---|
| **L0 — none** | convention + `AXIOM_AUDIT` link only; no CI check | solo expert; early-stage |
| **L1 — warn** | CI lists active axioms (from the kernel axiom set) with no vetting record; non-blocking | small trusted team; bootstrapping / pre-backfill |
| **L2 — gate (coverage)** | CI **fails** if any active project axiom has no vetting record | multi-contributor / agent-driven; soundness matters |
| **L3 — gate (coverage + freshness)** | L2 **plus** the record's `statement_hash` must match the current in-source axiom statement (catches "strengthened-but-not-re-vetted") | high-assurance; axioms edited in place |

Mechanics that make L2/L3 cheap: the **kernel-authoritative axiom set** comes from
the `#print axioms` report (same generated source as the axiom counts — see
`FORMALIZATION_YAML.md` "generate, don't hand-author"); coverage is a set-difference
against the `docs/vetting/*.md` front-matter, and freshness is a `statement_hash`
compare. **No contributor anticipates anything.**

Recommended rollout for a new adopter: **L1 now** (so CI doesn't red before records
exist) → **L2** once the backfill lands → **L3** once `statement_hash`es are
populated.

## Relationship to `AXIOM_AUDIT.md` and `formalization.yaml`

- `AXIOM_AUDIT.md` (per `AXIOM_AUDIT_FORMAT.md`) holds the *verdict + plan + consumers*;
  the `docs/vetting/` records hold the *evidence* it summarizes.
- `formalization.yaml › review` can aggregate the vetting front-matter (model,
  date, verdict per axiom) — generated, not hand-written.
