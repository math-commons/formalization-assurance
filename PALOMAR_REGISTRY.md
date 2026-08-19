# `PALOMAR_REGISTRY.md` — submitting to a public registry of formalized results

What the [Palomar Registry](https://palomar-registry.org) requires of a submission, and what a
passing entry does and does not establish. Companion to
[`COMPARATOR.md`](COMPARATOR.md) (the kernel-replay protocol Palomar runs) and
[`FORMALIZATION_YAML.md`](FORMALIZATION_YAML.md) (the project card it consumes).

**Sources.** [`PalomarRegistry/PalomarPolicy`](https://github.com/PalomarRegistry/PalomarPolicy)
at commit `24494b2` — `CONTRIBUTING.md`, `rubric.json`, `prompts/*.md`,
`taxonomies/classification-guide.md`, `docs/specification.md` — plus
[`PalomarRegistry/PalomarTemplate`](https://github.com/PalomarRegistry/PalomarTemplate).
Entry data is served from `data.palomar-registry.org` (`recent.json`,
`entries/<id>-v<n>.json`); the website is a JS app and returns nothing to a fetcher.

> Palomar is pre-launch and its policy is changing. Re-check the pinned commit before relying
> on any specific number or enum below.

---

## Why this sits in an assurance repo

Palomar's own §5 states the limit of its mechanical gate in almost the terms this repository's
README uses about kernels. A passing verification establishes that the recorded Solution
satisfies the recorded Challenge — and explicitly **not**

> that the Challenge says what the metadata claims, that a definition has its ordinary
> mathematical meaning, that the metadata is accurate, that the result is novel, or that the
> result is interesting.

That is the verification/validation split of
[`VERIFICATION_VALIDATION.md`](VERIFICATION_VALIDATION.md), drawn by a third party. Palomar
closes the gap the same way we do — with an artifact a reader can audit *without reading the
proofs* — and then adds an editorial layer on top.

---

## 1. The submission shape

Two Lean modules, plus a Comparator configuration and a project card.

- **`Challenge.lean`** — the statement surface a mathematician audits, carrying one `sorry`
  per advertised declaration.
- **`Solution.lean`** — the same declarations, proved.

**The binding constraint is the Challenge's transitive import closure.** Every file reachable
from it must be Lean core, Mathlib at a verified revision, or Tau Ceti. **No project-specific
source may appear anywhere in that closure** — so the Challenge cannot import the development
it advertises, and everything it needs must be restated against Mathlib. Recursive imports
count exactly like direct ones.

This is the design's whole point: the audit surface is written in a vocabulary the reader
already trusts. It is also the main engineering cost, because the two modules are elaborated
in *different environments*, and every constant they share must match **structurally**. See
[`COMPARATOR.md`](COMPARATOR.md) for what that failure mode looks like in practice.

Solution-only dependencies may come from any public GitHub repository and are explicitly
outside the definition-fidelity review.

**Sizes.** Hard limits 100 KiB / 1,000 lines for the Challenge; a mechanical warning above
32 KiB **or** 300 lines. The warning feeds an `auditability` score, not a blocking one.

**`comparator.json`** takes exactly four required keys — `challenge_module`,
`solution_module`, `theorem_names`, `permitted_axioms` — plus optional `definition_names` and
`enable_nanoda`. No other keys. `permitted_axioms` may contain only `propext`, `Quot.sound`,
`Classical.choice`. `enable_nanoda` is accepted but **non-authoritative**: Palomar ignores the
submitted value and writes its own protected configuration, so the field may be omitted. Older
local Comparator binaries reject the omission — patch a local copy, not the committed file.

**Dependencies.** Every Git package in `lake-manifest.json` on a credential-free
`https://github.com/owner/repository` URL pinned to a full 40-character lowercase SHA. No
submodules, no LFS pointers, no compiled artifacts committed outside `.lake`.

---

## 2. `formalization.yaml` under Palomar

Palomar consumes the Mathlib-Initiative card (see
[`FORMALIZATION_YAML.md`](FORMALIZATION_YAML.md)) at **v0.4**, makes the
subject-classification, responsible-maintainer and provenance fields mandatory, and accepts
unknown fields.

Mechanically required: `project.name`; `project.authors`; `project.license` matching the SPDX
identifier detected from the licence file *exactly*; `project.responsible_maintainers`;
`classification.arxiv` with **one or two** codes; `classification.msc2020` with one to eight;
`automation.methods` nonempty, each with a `method`; `review.status` nonempty; and every
`sources` entry with a `title` and a `relationship` from `formalizes | adapts |
independently-proves | background | other`.

**Two traps worth knowing before you hit them.**

1. A source `type`, when present, must be `paper`, `book`, `web discussion`, `folklore`,
   `original-proof`, or `other`. The *upstream* v0.4 schema documents the field as "article,
   book, web post, …" — so `article` looks correct and Palomar rejects it.
2. `classification.arxiv` caps at **two** codes. A third fails. The policy also says to
   classify the mathematics, not the use of Lean or AI, so a `cs.LO` added for the
   formalization itself is doubly wrong.

**`project.description`** is not in the upstream schema and is not mechanically required, but
Palomar's template carries it and the statement-alignment reviewer reads it as *the* concise
public abstract. Write it.

**Thin wrappers.** A repository existing only to expose another development's declarations
supplies

```yaml
repository:
  substantive_formalization:
    id: owner/repository
    revision: <full 40-char lowercase SHA>
```

Palomar then records that underlying repository as the substantive formalization, and
submitter authorization must concern *it*, not the wrapper.

**Authorization** is a separate question from write access. You must be a responsible author
or maintainer of the substantive formalization, or have approval from one. Write access, a
shared owner, organisation membership, a fork and a transferred repository are explicitly none
of them: "they say what you can do, not what the work is or whose it is."

---

## 3. The mechanical gate

The verifier checks out the commit, **discards submitted Lake build state**, materialises the
pinned dependencies, compiles the Challenge separately against Lean core plus the verified
Mathlib/Tau Ceti, records every source file that compilation touched and rejects anything
outside the permitted set, protects the compiled Challenge from replacement by project build
output, writes its own Comparator configuration with NanoDa enabled outside any writable
directory, runs Comparator with no network or credentials, and requires every exported proof
to pass Lean's kernel **and** replay through the pinned NanoDa kernel.

Note the shape: the Challenge is compiled in isolation *before* the project is allowed near
it, and the configuration that governs the comparison is written by the verifier, not the
submitter. Both are defences against a submission that arranges for the audit surface to be
something other than what it appears.

---

## 4. The editorial gate

A language model working through a fixed prompt sequence. **No person reads a submission
before a decision, and no person signs one.** It is not peer review.

Six passes, each returning a verdict (`neutral` / `warning` / `failure`), findings tied to
evidence, and only the scores it owns. Every substantive pass must return a coverage manifest
listing every Comparator theorem then every definition in configuration order; an incomplete
or reordered manifest is rejected.

| pass | examines | scores |
|---|---|---|
| classification | plausibility of every arXiv/MSC code | `classification` |
| metadata | clarity, accuracy, completeness of metadata, provenance, narrative | `clarity`, `provenance` |
| statement alignment | does each compared theorem express the claim the prose advertises — definitions, quantifiers, hypotheses, coercions, degenerate cases, scope | `statement_alignment` |
| definition fidelity | do the definitions the statements rest on mean what they claim; auditability of the Challenge closure | `definition_fidelity`, `auditability` |
| literature & notability | literature account and research interest | `notability`, `literature` |
| proof account | *(only when an informal proof account exists)* compares it against the actual Solution proof | — |

The definition-fidelity pass is the one an assurance-minded project should read closely: it
hunts for definitions that "manufacture the conclusion, omit a necessary well-formedness
condition, collapse a reachable case, or otherwise make the claim vacuous or materially
different" — the same failure mode [`OBJECT_CONTRACTS.md`](OBJECT_CONTRACTS.md) guards against
internally. Mathlib-only Challenge imports earn `high` trust; Tau Ceti earns `qualified`.

**Scoring.** `rubric.json` sets `minimum_score: 4`. Registry scores are `statement_alignment`,
`definition_fidelity`, `notability`, `literature`, `clarity`. A 4 or 5 "requires concrete
positive evidence, not merely successful compilation, populated fields, familiar terminology,
or the absence of an obvious contradiction." Non-registry dimensions — `auditability`,
`provenance` — may sit at 3 with a warning without blocking.

**`notability` is the only dimension whose shortfall mandates rejection**, and its anchors put
4 at "plausibly paper-worthy, with a specifically identified credible research audience" and 3
at exactly the case where that "has not been affirmatively established." The burden is on the
submission. Two things follow: **novelty is not required** — the prompt says so, and forbids
inferring novelty from a missing citation — and *"formal verification, effort, length, and
polish do not by themselves establish research interest,"* so the scale of a development is
not an argument for it.

**Findings are public; scores are not.** No reader-visible text may "state, bound, or imply a
score" — even "every score meets the minimum" is forbidden, since it bounds all of them.

---

## 5. Lifecycle, and what an attempt costs

```
submission + push access proved
  -> verifying -> verification-failed
              -> awaiting-review -> reviewing -> review-failed
                                             -> review-ready -> registered | withdrawn
```

From verification onward the **repository, commit, submission identifier, verification run and
mechanical logs are public**. The **editorial review and its outcome stay private** unless the
submitter registers a review that identified no blocking problem, which requires explicit
consent to the review digest. The submitter may withdraw from any non-terminal state, leaving
no public editorial outcome; a `revision_required` or `rejected` result is not registered, and
a corrected commit enters as a **new submission**, not a new version.

So an attempt is cheap and registration is the commitment. Since 2026-08-10 the database is
append-only — "a registration made now is permanent publication history" — and a record leaves
public view only by moderator-authorised suppression, which leaves a date-only tombstone.

**Proving push access, and the agent route.** A person signs in with GitHub OAuth and the
server reads `permissions.push` before discarding the token. An agent with no browser instead
creates **a tag at the submitted commit** and **a secret gist carrying the same challenge**;
the ref proves write access and the gist supplies an identity, "because a ref records no
author and a third party cannot ask GitHub who has push." The agent route is deliberately
weaker — it establishes that someone who can push submitted and that an account named itself,
"which are not provably the same account" — and the private record carries the distinction.
The published record names neither route nor submitter.

**Versions are a one-way door on repository identity.** A correction or dependency update
cites the existing identifier and becomes version 2, 3, …, but "automated updates must retain
the stable source repository, selected project path, and Comparator configuration path;
repository transfers require operator review." **Choose between an all-in-one repository and a
thin wrapper before the first submission**, because moving afterwards is not a cheap change of
mind. A source commit already in an identifier's history cannot be registered again, and every
version independently proves write access and re-declares authorization.

---

## 6. Start from the template

[`PalomarRegistry/PalomarTemplate`](https://github.com/PalomarRegistry/PalomarTemplate) is a
public starter: `Challenge.lean`, `Solution.lean`, `comparator.json`, a heavily commented
`formalization.yaml` whose placeholder values are *deliberately invalid* until chosen, a
nested doc-gen4 project, and CI that builds with `lean-action` and independently checks the
statement with Comparator.

Most useful for anyone already following [`COMPARATOR.md`](COMPARATOR.md):
`scripts/verify-comparator.sh` runs **pinned Comparator, lean4export, NanoDa and Landrun**
revisions against the checked-in configuration, and `scripts/landrun-wrapper.sh` "refuses any
Comparator request to switch off part of the sandbox." That is a real-sandbox run, already
scripted — strictly stronger than the macOS `fake-landrun` shim, which preserves the
kernel-acceptance and axiom-whitelist guarantees but not the sandbox.

Note the template keeps its proof development *inside* the submission repository, so it models
the all-in-one layout rather than the thin wrapper.
