<!-- Template for audit/contracts/<Object>.md in an adopting project.
     One card per CONSTRUCTED OBJECT (not per theorem, and not for axioms — axioms get
     a vetting record + closure status instead). See OBJECT_CONTRACTS.md. -->
---
object: Project.theObject
informal: >
  One-paragraph, plain-English, textbook-language description of what the object is.
sources:
  - "Author, Title (Publisher, Year), §/Thm number"
lean:
  name: Project.theObject
  signature: "(args...) : ReturnType"        # signature only — never the proof body
  body: "one-line sketch of the definition (what it computes), not the Lean proof"
characterization:
  - id: C1-some-property
    claim: "A defining/known property the object must satisfy."
  - id: C2-anti-degeneracy
    anti_degeneracy: true
    claim: >
      The property a plausible LOOK-ALIKE would fail — i.e. what would make this the wrong /
      hack object. At least one such clause is required; a known_values row should witness it.
known_values:
  # instance -> expected -> witnessing theorem -> status -> note
  # status ∈ { PROVEN_CORE_AXIOMS | proven_via_axiom | proven_mod_axioms | sorry }
  #   read from `#print axioms` of the theorem (see OBJECT_CONTRACTS.md). Never hand-asserted.
  - instance: "theObject at a concrete input"
    expected: "the independently-known value"
    theorem: Project.theObject_value_lemma
    status: PROVEN_CORE_AXIOMS
    note: "e.g. #print axioms pinned in <Tests file>"
  - instance: "the anti-degeneracy instance"
    expected: "the value a look-alike would get wrong"
    theorem: Project.theObject_antihack_lemma
    status: PROVEN_CORE_AXIOMS
    note: "witnesses C2"
well_definedness: >
  The instance/fact the definition silently relies on (integrability, finiteness, …).
anti_degeneracy:
  history: >
    Any real degeneracy bug seen (or the natural wrong definitions and what they break).
  current_guard: >
    The theorem(s) that exclude it, and where their #print axioms are pinned.
status: >
  One-line summary: which rows are PROVEN_CORE_AXIOMS vs proven_mod_axioms, and where pinned.
---

# Contract — `Project.theObject`

Short reader's guide: how to confirm, **without reading any proof**, that this is the right
object — check the `known_values` rows against the cited theorems (each carries a
kernel-derived status) and the anti-degeneracy clause (the property a wrong definition fails).
