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
