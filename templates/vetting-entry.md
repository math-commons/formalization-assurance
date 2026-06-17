<!-- Template for audit/vetting/<AxiomName>.md in an adopting project.
     One ## entry per vetting event, newest first. Keep prior entries
     (re-vettings append; mark the superseded one via `superseded_by`).
     See VETTING.md for the convention. -->

# Vetting — `AX_Foo`

Captured soundness-review records for `AX_Foo` (`<path>/Foo.lean:NN`).
Linked from `AXIOM_AUDIT.md`.

---

```yaml
---
axiom: AX_Foo
file: <path>/Foo.lean:NN
statement_hash: sha256:<hash of the normalized axiom statement>
model: gemini-3.1-pro-preview        # or: codex/GPT-5.4 ; or n/a for LP/SA
tool: mcp__gemini__deep_think_gemini
source_code: DT                      # DT | CX | GR | LP | SA | PR
date: YYYY-MM-DD
questions: [typing, strength, non-vacuity, satisfiability]
verdict: SATISFIABLE_FAITHFUL        # | FLAGGED | NEEDS_REVISION
rating: Likely correct               # Standard | Likely correct | Needs review | Flagged | Placeholder
discharged: false                    # true once proved; keep this record for provenance
superseded_by: null                  # date of a later entry if the statement changed
---
```

**Axiom statement vetted (verbatim):**
```lean
axiom AX_Foo {X : Type*} [...] : ...
```

**Prompt (verbatim):**
> Paste the exact query sent to the model, in full (reproducibility over brevity).

**Reply (verbatim; excerpt only boilerplate, never paraphrase the reasoning):**
> Paste the model's full answer.

**Reasoning digest:** `<verdict>` — one or two lines of the load-bearing argument
(this is what gets mirrored into `AXIOM_AUDIT.md`).

**Conditions / follow-ups:** flags, conditional hypotheses, "re-vet if strengthened";
or `none`.
