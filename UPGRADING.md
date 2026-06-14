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
    action   = Convert the embedded gitlink submodule to a bare store + worktrees — MIGRATION.md "Converting an existing submodule": preserve the pin, de-register the gitlink, write .host-software, gitignore the trees, then `host-lifecycle software --materialize`.
    requires = host-lifecycle v0.3.0

[upgrade "325f2cf"]
    title    = Worktree-absence coherence (call/0005)
    action   = Untrack artifacts that depend on the worktree (skill symlinks into the software path); gitignore them and recreate after materialize. Keep an un-materialized CI job green. `host-lifecycle software --check` must report no HAZARD.
    requires = host-lifecycle v0.3.1
