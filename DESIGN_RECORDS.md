# Design records

A design record documents a settled design choice in a formalization project: what we
decided, what the alternatives were, and why we chose as we did. It is the place to write
down the reasoning behind an encoding or structuring decision so that it is not rediscovered,
re-argued, or silently reversed later.

This fills a gap between the other documents. `OPEN_QUESTIONS.md` records tensions that are
*unresolved*. The object-contract cards (`OBJECT_CONTRACTS.md`) record the *resulting* spec of
each definition. A design record records the *resolved choice and its rationale*: the step in
between. It is also the natural thing to point a reviewer at when they ask "why is the object
encoded this way?".

## When to write one

Write a design record when a choice is non-obvious, affects the public API, or has a credible
alternative someone will later wonder about. Typical triggers:

- The encoding of a central object (for example, a strict function vs an a.e.-equivalence
  class; an unbounded operator as `LinearPMap` vs a bespoke domain + map; total-with-guard vs
  a subtype-indexed parameter).
- The generality level chosen (and what was deliberately *not* generalized, with why).
- A bundling decision (one bundled structure vs separate mixins).
- A comparison with another formalization of the same mathematics (as in the worked example
  below), recording where the two differ and how they would bridge.

If a choice is trivial or has no real alternative, do not write one. Records are for decisions
that carry weight.

## Where they live

- A single project: a top-level `DESIGN.md` with one section per decision is enough.
- A project with several substantial decisions: a `docs/decisions/NNNN-slug.md` file per
  record (numbered in order), with a one-line index in `DESIGN.md`.

Either way, keep each record short and link it from the relevant object-contract card (see the
optional `encoding_rationale` pointer in `OBJECT_CONTRACTS.md`).

## Format

Use the template in [`templates/design-record.md`](templates/design-record.md). A record has:

1. **Title and status** — a short title, a date, and a status (`accepted`, `superseded by …`,
   or `proposed` if still under discussion, in which case it likely belongs in
   `OPEN_QUESTIONS.md` instead).
2. **Context** — the decision that had to be made and the constraints around it.
3. **Decision** — what was chosen, stated plainly.
4. **Alternatives considered** — the real options, each with its trade-off. A table comparing
   the options on the axes that matter is often the clearest form.
5. **Consequences** — what the choice buys and what it costs, including any debt it creates.
6. **Bridging / migration** (optional) — if the alternative is also in use somewhere, how to
   translate between them, and in which direction the translation is lossy.

Keep it factual and specific. A design record describes a decision, not a proof; the proof
obligations it creates belong in the project's audit documents.

## Relationship to the other documents

- `OPEN_QUESTIONS.md` — *unresolved* tensions. A `proposed` design record that gets rejected,
  or a decision later reopened, moves here.
- `OBJECT_CONTRACTS.md` — the *spec* that results from the decision. Link the card to the
  record.
- `VERIFICATION_VALIDATION.md` / `FAITHFULNESS.md` — whether the chosen encoding is *faithful*
  to the intended mathematics. A design record explains the choice; faithfulness checks it.

## Worked example

`random-fields/graphons` records the single most basic encoding choice for graph-limit theory,
strict function vs L⁰-class kernel, by comparison with an independent formalization
(`cameronfreer/graphon`): the two encode the same mathematics but make opposite carrier
choices, and the record captures the axes (symmetry everywhere vs a.e., measurability explicit
vs intrinsic, module structure vs none), the generality-vs-ergonomics trade-off, and the
asymmetric bridge between the two. That note is the model this convention generalizes.
