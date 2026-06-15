# Comparator verification — protocol

External kernel-replay + axiom-whitelist check of a project headline theorem, via
the [Lean FRO comparator](https://github.com/leanprover/comparator). This is the
strongest external **verification** signal: a third party re-exports the term and
replays it through the Lean kernel, independent of the project's own CI, and
confirms it uses only whitelisted axioms.

## When to run

A project is a candidate when, **on the verified commit**: the headline theorem is
`sorry`-free, project axioms are zero (or all on the standard whitelist
`{propext, Quot.sound, Classical.choice}`), the repo is public/released, and
external trust matters (Lean FRO reservoir, public release, paper companion). Skip
for in-flux work.

## One-time local prerequisites

Build once (paths below are an example local layout; adjust to your machine):

- `comparator` binary — built with its own toolchain.
- `lean4export` binary — built at the **tag** matching the *project's* Lean version
  (e.g. `git clone --branch v4.30.0 https://github.com/leanprover/lean4export`).
  Released versions use the `vX.Y.Z` **tag**; `-rcN` use the `bump_to_…` **branch**.
- The official insecure macOS `fake-landrun.sh` shim (no Linux sandbox on darwin).

## Per-project test layout

A sibling `<project>-comparator-run/`:

- `lakefile.toml` — `[[require]] name="<Project>" path="../<project>"`, plus
  `[[lean_lib]] name="Solution"` and `[[lean_lib]] name="Challenge"`.
- `lean-toolchain` — match the **project's** (not the comparator's).
- `Challenge.lean` — `import <Project>.…`; state the headline theorem `:= sorry`.
- `Solution.lean` — same signature; body delegates to the real
  `_root_.<qualified_name>`.
- `config.json` — `challenge_module`/`solution_module` = `"Challenge"`/`"Solution"`,
  `theorem_names: ["…"]`, `permitted_axioms: ["propext","Quot.sound","Classical.choice"]`,
  `enable_nanoda: false`.

## macOS APFS-clone trick (mandatory for speed)

A fresh path-required project triggers lake to re-clone all transitive deps (Mathlib
alone is multi-GB). Bypass with an APFS clonefile:

```bash
rm -rf <project>-comparator-run/.lake
cp -c -R <project>/.lake <project>-comparator-run/.lake
cp <project>/lake-manifest.json <project>-comparator-run/lake-manifest.json
( cd <project>-comparator-run && lake update <Project> )   # registers the path-dep
( cd <project>-comparator-run && lake build )
```

## Invocation

```bash
cd <project>-comparator-run && \
  COMPARATOR_LANDRUN=…/comparator/scripts/fake-landrun.sh \
  COMPARATOR_LEAN4EXPORT=…/lean4export/.lake/build/bin/lean4export \
  lake env …/comparator/.lake/build/bin/comparator config.json
```

Pass = `Lean default kernel accepts the solution / Your solution is okay!`

## Gotchas

- `comparator/runtests.lean` overwrites the test project's `lean-toolchain` —
  **invoke the binary directly** for projects on a different Lean.
- `fake-landrun.sh` keeps the **kernel-acceptance + axiom-whitelist** guarantees
  (those run in `lean4export` + kernel replay); the **sandbox** guarantee does not.
  For adversarial judging use a Linux box with real `landrun`.
- `[[lean_lib]]` names must match the module filenames literally (`Challenge`,
  `Solution`).
- **In-tree path deps** (e.g. a vendored port): the copied `lake-manifest.json`
  records them relative to the workspace root, so `lake update <Project>` fails with
  "package directory not found". Rewrite that manifest entry to
  `../<project>/vendor/<port>` before `lake update`.

## After a pass

Add one line to the project's `README.md` naming the verified theorem(s) + the
commit hash actually checked, linking the comparator. Append a row to the registry
below (update in place on re-check).

## Verified-runs registry

Each adopting project appends its comparator-verified results (update a row in place
on re-check).

| Project | Commit | Theorem(s) |
|---|---|---|
| [`mrdouglasny/jacobian-challenge`](https://github.com/mrdouglasny/jacobian-challenge) | `67af290` | `riemannRochL3` |
| `mrdouglasny/jacobian-challenge` | `2bd75f8` | all 11 Buzzard property theorems axiom-free (whitelist-only, `config-buzzard.json`, 2026-06-14) |
