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

## Recommended: a committed one-command `verify.sh`

The sibling-dir + env-var invocation above is the *mechanics*. For a released project,
**commit the workspace and a self-bootstrapping `scripts/verify.sh` into the repo** so any
reviewer reproduces the external check with one command on a fresh clone — no manual setup,
no remembering env vars. This is the single highest-value reviewer affordance (prior art:
[`rkirov/jacobian-claude`](https://github.com/rkirov/jacobian-claude)'s `verify.sh`).

Layout (in-repo, vs the sibling `<project>-comparator-run/`):
- `scripts/comparator/` — `Challenge.lean`, `Solution.lean`, `config.json` (+ extra configs),
  `lean-toolchain`, and a `lakefile.toml` whose `[[require]]` points at the parent repo with
  `path = "../.."` (the workspace lives one dir below `scripts/`). `.gitignore` its `.lake/` +
  `lake-manifest.json`. The stray `.lean` files are **not** swept into the main build as long
  as the project's `lean_lib` roots at its own module dir (the usual case).
- `scripts/verify.sh` — clones `comparator` + `lean4export` **pinned to the workspace's
  `lean-toolchain`** (the `.olean` export format is toolchain-locked), builds them, builds the
  workspace, and runs the comparator.

Reference script (adjust the libs/configs to the project):

```bash
#!/usr/bin/env bash
# External re-verification: real leanprover/comparator (statement match + permitted-axiom
# check + independent kernel replay via lean4export). The library .oleans are NOT trusted.
set -euo pipefail
HERE="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"; WS="$HERE/comparator"
CFG="${1:-config.json}"; [ -f "$WS/$CFG" ] || { echo "no config: $WS/$CFG" >&2; exit 2; }
TAG="$(tr -d '[:space:]' < "$WS/lean-toolchain" | sed -e 's#^leanprover/lean4:##')"
WORK="${COMPARATOR_WORK:-$HOME/.cache/<project>-comparator}"; mkdir -p "$WORK"
for r in comparator lean4export; do
  [ -d "$WORK/$r" ] || git clone --branch "$TAG" --depth 1 "https://github.com/leanprover/$r" "$WORK/$r"
  ( cd "$WORK/$r" && lake build )
done
if [ -z "${COMPARATOR_LANDRUN:-}" ] && command -v landrun >/dev/null 2>&1; then
  COMPARATOR_LANDRUN="$(command -v landrun)"; fi   # Linux sandbox; macOS runs unsandboxed
[ -n "${COMPARATOR_LANDRUN:-}" ] && export COMPARATOR_LANDRUN
export PATH="$WORK/lean4export/.lake/build/bin:$PATH"
cd "$WS"; lake build Challenge Solution
exec lake env "$WORK/comparator/.lake/build/bin/comparator" "$CFG"
```

**Internal vs external — keep both, label them.** A project's "how to re-check" should split:
*internal* checks (`#print axioms` / axiom-report diff / count consistency) **trust the
project's own `.oleans` and report generator**; `verify.sh` is the *external* check that trusts
only the kernel + Mathlib + comparator + the verbatim spec, re-deriving every proof. Cite the
external one as authoritative.

**Trust boundary to state in the script header:** only the Lean kernel, Mathlib, comparator,
and `Challenge.lean` (the statement) are trusted; the project library + any vendored port are
re-exported and re-checked, so a corrupted `.olean`, poisoned build cache, or rogue axiom is
caught even when the project's own `#print axioms` (on its build) would not surface it.

> If `verify.sh` is the trust root, add it (and any comparator CI job) to `CODEOWNERS` so it
> can't be silently weakened — same rationale as protecting the axiom-report generator.

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
