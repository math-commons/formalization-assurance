# Open questions and limitations

These conventions are a work in progress. This file records the design tensions we
know about, who raised them, and where we currently stand. We would rather state
these in the open than imply the framework settles questions it does not.

For actionable, trackable items we use the GitHub issue tracker. This file is for
the durable questions that a reader should see up front. Several of these came out
of discussion on the Lean Zulip; we link out there for the live conversation.

## 1. Do axioms undercut the whole point?

**The concern.** A development that assumes axioms is conditional, not absolute. If
the headline theorem rests on an unproven `axiom`, calling the project "assured" can
oversell it. (Raised by Kim Morrison and Johan Commelin.)

**Where we stand.** We agree, and we try not to hide it. An axiom is a debt, not a
free lemma. That is exactly why "assumption review" is its own layer
(`VETTING.md`, `AXIOM_AUDIT_FORMAT.md`): every axiom is reviewed for soundness, rated,
and recorded, and the project's axiom count is generated from `#print axioms` rather
than asserted. A result proved modulo a clearly labeled, well-reviewed axiom is still
useful, but it is a weaker claim than an axiom-free one, and our artifacts are meant to
make the difference visible rather than blur it. Axiom-free remains the goal; the
acceptance ladder treats it as the top rung.

## 2. Conventions written by an LLM

**The concern.** Rules for what counts as a trustworthy formalization should be laid
down by humans, not by a language model, and the prose here has used new jargon that
signals confidence without adding clarity. If most of the writing is machine-generated,
who is the intended reader? (Raised by Johan Commelin and Filippo Nuccio.)

**Where we stand.** Fair, and we are acting on it. The conventions are owned and edited
by people; an LLM drafting text does not change who is responsible for the rules. We are
doing a pass to remove coined terms and confidence-signalling language in favor of plain
statements, and to keep each document short enough that a mathematician will actually
read it. If a term is not standard, we define it in plain words on first use or drop it.

## 3. Machine-checkable facts vs human judgment

**The concern.** The things a machine can check (does it build, what axioms appear) and
the things only a person can judge (is this the right definition, is this axiom true)
should be kept clearly separate, not blended into one "score." (This is the spirit of
Rémy Degenne's work on auto-generated exposition.)

**Where we stand.** We try to keep the line sharp. Kernel facts (build status, axiom
lists, sorry counts) are generated and enforced in CI; nobody is asked to assert them by
hand. Human judgments (faithfulness of a definition, soundness of an axiom) are recorded
as reviewed claims with their evidence, and labeled as judgments. We see auto-generated
exposition as complementary: it presents what the formalization says; our artifacts add
the human-reviewed claim that what it says is the intended mathematics.

## 4. Visual trust markers

**The question.** Tools like PhysLean's `@[sorryful]` attribute surface trust state
directly in the source and in generated docs, so a reader sees at a glance what rests on
a `sorry`. Should these conventions adopt or recommend something similar?

**Where we stand.** We think this is a good direction and are considering adopting and
crediting it, so that trust state is visible in the code and not only in the audit
documents.

## 5. How far can "the right object" ever be certified?

**The question.** Validation tops out at a categorical certificate: a proof that the
specification has exactly one model up to isomorphism. Most working APIs are not
categorical, so for them the strongest available evidence is an acceptance ladder
(named theorems, known-value checks) rather than a uniqueness proof.

**Where we stand.** We are honest about this ceiling. Where a categorical certificate is
available we aim for it; where it is not, the acceptance ladder is evidence, not proof,
that the definition is the intended one, and we say so.

---

If you have a critique that is not captured here, please open an issue or raise it on
Zulip. Recording the disagreement is part of the point.
