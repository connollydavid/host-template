# UPGRADING.md — the version-to-version upgrade ledger

Adopting the methodology records a template revision in `.host`. The
template then moves on, and an adopted repo must **upgrade** across the revision
span — re-applying spine doc changes *and* the **structural migrations** the span
introduced (re-embedding the software, bumping a tool). A `git diff` of the
template shows the prose; it does not say "convert the submodule" or "untrack a
symlink". This ledger does.

Apply every entry below whose revision is **newer** than your stamp.
`host-lifecycle upgrade <dir>` prints exactly those — decided by git ancestry
against this template, so same-day revisions order correctly. Fetch the template
to the target revision first; then apply the list and re-stamp. Each entry is
keyed by the template revision at which its action became required.

[upgrade "8c28e33"]
    title    = Software is a bare store with worktrees (call/0004)
    action   = Convert the embedded gitlink submodule to a bare store + worktrees — see the `host` repo (converting an existing submodule): preserve the pin, de-register the gitlink, write .host-software, gitignore the trees, then `host-lifecycle software --materialize`.
    requires = host-lifecycle v0.3.0

[upgrade "325f2cf"]
    title    = Worktree-absence coherence (call/0005)
    action   = Untrack artifacts that depend on the worktree (skill symlinks into the software path); gitignore them and recreate after materialize. Keep an un-materialized CI job green. `host-lifecycle software --check` must report no HAZARD.
    requires = host-lifecycle v0.3.1

[upgrade "71d12a8"]
    title    = Coherence generalized to tool submodules (call/0005)
    action   = Re-run `host-lifecycle software --check` (now flags any tracked symlink whose target is not tracked here, not just software-worktree paths). Untrack any flagged tool-skill symlinks and generate them instead (the link-skills.sh pattern); keep them gitignored.
    requires = host-lifecycle v0.4.1

[upgrade "bbbfdc3"]
    title    = Reserve agentic-host for the meta repo; stamp is `.host`, scaffold is `host-template`
    action   = Rename your `.agentic-host` stamp to `.host`. Re-point your `template-agentic-host` submodule and its URL to `host-template`. A repo that adopts the methodology is "an agentic project" (e.g. `agentic-acme`); `agentic-host` now names only the meta repo.
    requires = host-lifecycle v0.5.0

[upgrade "7ae93cd"]
    title    = Publish via `host-lifecycle book` (the canonical doc-site publisher)
    action   = Replace any hand-rolled mdBook generator/SUMMARY with `host-lifecycle book .`; it writes book.toml (src = "docs", never ".") + docs/ in lifecycle order, renders specs, and emits a Where stub from .host-software. Run `host-lifecycle book . && host-lifecycle book --check .` before `mdbook build` (see .github/workflows/site.yml); gitignore the generated book.toml/docs/.
    requires = host-lifecycle v0.6.1

[upgrade "6db01f3"]
    title    = Anti-ouroboros: call/ is for the software, methodology lives in the spine
    action   = The methodology is owned by the spine (copy-at-version), not your call/. In a single dedicated commit, set `Status: superseded by the spine` on every accepted call/ decision that merely restates a methodology rule now settled upstream (leave the file in place — MADR records are immutable). Add a `Scope:` header to each remaining accepted decision. Do not treat any host/management repo's top-level instance rooms as normative. `host-lifecycle validate <call-dir>` now fails an accepted decision missing `Scope:` or declaring `Scope: methodology`.
    requires = host-lifecycle v0.7.0

[upgrade "07025a7"]
    title    = book publisher v0.7.1: nested specs + retired-decision Archive
    action   = Bump your pinned `host-lifecycle book` revision to v0.7.1. Two rendering changes follow on the next build: specs nested in `spec/<topic>/` now render (previously dropped), and decisions whose MADR `Status:` is superseded/deprecated/rejected move into a trailing "Archive / Record" section (banner + nav-label suffix) instead of shipping as current chapters. The record signal is `Status:` only — `.host-lintignore` does not affect the book.
    requires = host-lifecycle v0.7.1

[upgrade "e3b174d"]
    title    = Reproducible builds are the production anchor
    action   = Record build provenance in `.host-software` per component: `build` + `toolchain` (the deterministic recipe), `deploy` (which line ships), and `artifact = <worktree-path> <sha256>`. Add a CI job that runs `host-lifecycle software --verify-build` (rebuild from the pin; fail unless the artifact reproduces) — see the reference `.github/workflows/reproducible-build.yml`. Software initiated under the methodology must reproduce. Pre-existing software not yet reproducible may carry `repro-exempt = call/NNNN` citing a software-scoped case decision; the exemption is retired as it converges and is never available to greenfield software.
    requires = host-lifecycle v0.8.0

[upgrade "c137567"]
    title    = Per-platform builds for multi-platform software
    action   = For a component whose one source pin ships on several platforms, replace the flat single-build fields with one `[build "<name>" "<platform>"]` subsection per platform under its `[software "<name>"]` stanza — each with its own `build`/`toolchain`/`artifact`/`deploy` (and optional `repro-exempt`) plus an `attest-host` naming the OS (`linux`/`windows`/`macos`) that reproduces it. `host-lifecycle software --check`/`--verify-build` then attest each build only on its `attest-host`, skipping foreign-host builds rather than failing. Single-platform components need no change: the flat form stays valid.
    requires = host-lifecycle v0.9.0
    independent = true

[upgrade "d3dc5ed"]
    title    = --check artifact mismatch is a note, not a failure
    action   = Bump your pinned `host-lifecycle` revision to v0.9.1. `software --check` no longer hard-fails when a present artifact's hash differs from the recorded canonical hash (a local-toolchain build legitimately differs) — it prints a `note` instead, the way `--install-hooks` already does. A match still reports `verified`; `--verify-build` (the container/CI lane) remains the reproducibility proof. No recipe change is required; this only removes a dev-box false failure.
    requires = host-lifecycle v0.9.1
    independent = true

[upgrade "b6232a5"]
    title    = Specs live with the software, not the host plan tree
    action   = Move any behaviour (.allium) or timing (.tla) specs out of the host's `plan/<milestone>/spec/` and into the software repo, beside the code they constrain. Add the verification lane to the *software's* CI, not the host's: allium gated with both `allium check` and `allium analyse` (install `allium-cli@3.4.2`; both exit non-zero on any error or warning), and specula/TLC (`tla2tools v1.8.0` on a Temurin `21` JDK). Remove the host's spec copies and its specula workflow; leave a forward note in the affected milestone. The host `plan/<milestone>/` now references a spec by path and software pin rather than containing it.
    requires = host-lifecycle v0.9.1

[upgrade "c771d60"]
    title    = Verification lanes are mandatory when a spec of their kind exists
    action   = Wire the lane tools you use as submodules and generate their skills (gitignored) with `link-skills.sh`: `tools/allium` (+ its elicit/distill/tend/weed/propagate skills) for any `.allium`, `tools/specula` for any `.tla`. Author and maintain `.allium` specs THROUGH the skills, not by hand. In the *software* repo's CI, gate each `.allium` with `allium check` + `allium analyse` + `allium plan` (install `allium-cli@3.4.2`; both check and analyse exit non-zero on any error or warning) and discharge the `plan` test obligations with the suite; TLC-check each `.tla` (`tla2tools v1.8.0` on Temurin `21`). The lane is conditional — TLA+ stays optional until a `.tla` exists, allium until a `.allium` exists — but a spec present without its full lane is a defect, not a choice. Do not treat the lanes as reference decoration.
    requires = host-lifecycle v0.9.1
    depends  = b6232a5

[upgrade "b8c54fc"]
    title    = The spec-lane MUST is enforced by software --check
    action   = Bump your pinned host-lifecycle to v0.10.0. `host-lifecycle software --check` now raises a HAZARD when a materialized component carries a `.allium` spec with no CI workflow running `allium check` + `allium analyse`, or a `.tla` with no TLC lane (`tlc2.TLC`/`tla2tools`). Wire the missing lane (per the previous entry) so --check is clean; an un-materialized worktree is skipped.
    requires = host-lifecycle v0.10.0
    depends  = c771d60

[upgrade "821a216"]
    title    = Allium plan obligations must be dispositioned per component
    action   = For each `.allium`, add a sibling `<spec>.obligations` manifest that dispositions every obligation `allium plan` derives — `test:<name>` (a named test discharges it), `structural` (the check/analyse lane covers it), or `waived: <reason>`. Run `host-lifecycle obligations <spec> --tests <dir>` in the software's CI (it fails on any undispositioned/stale obligation or absent test ref). Bump host-lifecycle to v0.11.1, where `software --check` also HAZARDs a `.allium` with no `.obligations` manifest.
    requires = host-lifecycle v0.11.1
    depends  = b6232a5 c771d60

[upgrade "f62d766"]
    title    = Lifecycle phases are an unconditional MUST, driven by host-lifecycle skills
    action   = Bump tools/host-lifecycle and run link-skills.sh so the phase skills (classify, adopt, embed, remap, verify, publish, upgrade) appear under .claude/skills/ alongside the allium/specula skills. Operate the methodology through these phases and their commands — scaffolding, embedding, migration, verification, publishing and upgrading — not ad-hoc. Unlike a verification lane, this is unconditional: there is no opt-out, and hand-operating any phase is a defect.
    requires = host-lifecycle v0.11.1
    independent = true

[upgrade "0e83e3f"]
    title    = Tag every release (a version bump MUST carry a matching vX.Y.Z tag)
    action   = Audit your tools and software for version bumps that were never tagged (`git tag` vs the manifest `version`); back-fill an annotated `vX.Y.Z` tag at each bump commit and push it. From now on, every version bump is committed with its matching tag, pushed alongside — the tag is the release a `v*` CI workflow builds. Do not re-pin `.host-software` or a tool pointer to a version-bumped commit that has no matching tag.
    requires = host-lifecycle v0.11.1
    independent = true

[upgrade "ae1e688"]
    title    = Never adopt a software repository in place
    action   = Bump your pinned host-lifecycle to v0.12.0 and re-run the classify phase as the first step of any adoption. `host-lifecycle classify <dir>` now refuses (exits non-zero, prints the embedding steps) when the target is itself a software repository — a root build manifest (`Cargo.toml`, `package.json`, `go.mod`, `pyproject.toml`, …) with no `.host` stamp and no `.host-software` recipe — instead of printing a case letter. A host is a separate meta-repo; the software is embedded as its Where room. If you previously adopted a methodology host directly on top of a software repo, split them: move the code into a bare store with worktrees recorded in the host's `.host-software`, leaving the host root carrying only the rooms and config.
    requires = host-lifecycle v0.12.0
    independent = true

[upgrade "7de7cb1"]
    title    = Materialized worktrees must live under the host root
    action   = Bump your pinned host-lifecycle to v0.13.0. Audit `.host-software` for any worktree whose path escapes the host root (an absolute or `..`-climbing `worktrees`/`worktree` entry, or a Where-room worktree materialized at a disjoint external path with no in-structure handle). Bring each under the root: for a store that must live off-tree (another filesystem or platform), record it on the parallel line as `worktree = <dir> <branch> <pin> store=<path> [host=<os>]` and re-run `software --materialize`, which realises the store at `<path>` and the in-tree `<dir>` as a symlink/junction to it. `software --check` now HAZARDs an escaping path and a `store=` line whose in-tree handle is missing or does not resolve to the store; `host=` gates a line to one OS so a foreign-OS CI skips it rather than failing.
    requires = host-lifecycle v0.13.0
    independent = true

[upgrade "3f7c065"]
    title    = Honest partial upgrades — the applied-set stamp model
    action   = Bump your pinned host-lifecycle to v0.14.0. The `.host` stamp now records what is applied as a `baseline` ledger entry (every entry at or before its position in this file counts applied) plus an optional `applied` set of out-of-order entries — keyed by ledger position, never git ancestry. Your first `host-lifecycle upgrade` migrates a legacy single-`revision` stamp once (derives the baseline; no manual edit). Thereafter: `upgrade` lists pending by position, `upgrade --next` prints the single next safe action, and you record an applied entry with `host-lifecycle upgrade --record <id>` (id, unambiguous prefix, or ledger ordinal) — which validates the id, refuses if a `depends` is unapplied, runs the entry's `verify` post-condition or requires `--unverified call/NNNN`, and appends an append-only claim. Never hand-edit the stamp. A late `independent` entry may be cherry-applied without an earlier unrelated one (deferred entries stay pending and re-list — a forgotten or premature record can never hide owed work); `upgrade --advance` compacts a contiguous applied run into the baseline; `software --check` re-checks every recorded claim.
    requires = host-lifecycle v0.14.0
    independent = true

[upgrade "4a98d92"]
    title    = Deeper verification rungs — Apalache (symbolic), TLAPS (proof), Kani (code-conformance)
    action   = Bump your pinned host-lifecycle to v0.15.0. The verification ladder gains opt-in deeper rungs above the bounded lanes, driven by a referenced `tools/host-prove` submodule (wire it and run `link-skills.sh` only if you use a rung; its `apalache-symbolic`, `tlaps-proof` and `kani-conformance` skills turn each verifier into a one-line verdict). An obligation may now be dispositioned `kani:<harness>`, `apalache:<inv>` or `tlaps:<theorem>` instead of `test:`/`structural`/`waived:` — a proof discharges it for all inputs/parameters; `host-lifecycle obligations <spec> --prove <dir>` validates that the named proof exists. Declaring a rung obliges its CI lane: `software --check` now HAZARDs an obligation that declares `kani:` with no `cargo kani` lane, `apalache:` with no `apalache-mc` lane, or `tlaps:` with no `tlapm` lane. Nothing is required until you declare a rung — bare `.tla`/crate presence never activates one. Route a parametric or unbounded `.tla` invariant to Apalache/TLAPS, and a code-conformance claim to your language's verifier (Rust → Kani); prefer byte/char-level Kani targets (`str::split`/`Vec` blow CBMC up). Verifiers install from official prebuilt binaries pinned by version + SHA256; no Docker.
    requires = host-lifecycle v0.15.0
    depends  = b6232a5 c771d60
