# Axiom audit — <project>

*Last updated YYYY-MM-DD.*

## Purpose

In this project an **axiom** is a *vetted, provable theorem with a vetted discharge
plan* — a staging point, not a fundamental assumption. Format + conventions:
[`math-commons/formalization-assurance/AXIOM_AUDIT_FORMAT.md`](https://github.com/math-commons/formalization-assurance/blob/main/AXIOM_AUDIT_FORMAT.md).
Per-axiom *evidence* (model + prompt + reply) is under `docs/vetting/`.

---

**Active axioms in build: N.** <one-line summary>
*(Count is derived from the generated `docs/axiom-report.txt` — do not hand-edit.)*

---

## Active axioms

### <Axiom name>

**File**: `<path>:<line>`
**Statement** (informal): <one paragraph>.
**Vetting**: **<Rating>** (`<codes>`) — <reference + brief justification> →
[vetting](docs/vetting/<AxiomName>.md)
**Strategy / Plan**: <how to discharge + estimate; link a plan doc if long>.
**Consumers**: <downstream theorems that load-bear on this axiom>.

<!-- repeat per axiom; or use the table form from AXIOM_AUDIT_FORMAT.md -->

---

## Recently discharged

| Axiom | Discharged via | Where the proof lives |
|---|---|---|
| `foo` | <strategy> | `path/to/file.lean` |

---

## Verification

```bash
find . -name '*.lean' -not -path './future/*' | xargs grep -lE '^axiom '
```
