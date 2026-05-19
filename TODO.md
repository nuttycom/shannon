# TODO

Tracking list for Shannon work. Entries are brief; details for any item live in `docs/<topic>.md` when one exists.

## Tests

<!-- See also: `docs/testing.md` is the canonical source for the test design and the per-script test-case tables. The TODO items below should not enumerate test counts or per-case details (those belong in `docs/testing.md`); edits to the test design happen there, not here. If a TODO item adds or removes a *script* to be tested, both files need updating. -->

- [ ] Write the bats suite for `check-memory-synthesis.sh`. See `docs/testing.md` for the per-case table (currently eight cases covering memory-body, `user_*.md`, `MEMORY.md`, Edit-shaped input, malformed JSON, missing `file_path`, project-memory path, and non-memory path — `docs/testing.md` is authoritative if this list drifts).
- [ ] Add `tests/fixtures/` (stdin payloads, fixture memory corpora at green / yellow / red sizes, sample transcript).
- [ ] `.github/workflows/test.yml` running `bats tests/` via `bats-core/bats-action`.
- [ ] `session-start.sh` and `save-session.sh` test cases once those scripts are in `hooks/`.

## Hook scripts

- [x] Move `check-memory-synthesis.sh`, `session-start.sh`, and `save-session.sh` into `hooks/`. The three scripts now live in `shannon/hooks/` and the maintainer's `~/.claude/<name>.sh` paths are symlinks pointing at them. Content is verbatim from the maintainer's setup (no separate sanitization pass was needed — the scripts were already generic).
- [x] Decide the `~/.claude/jsonl-to-md.py` dependency for `save-session.sh`. Decision: ship the helper in Shannon — it now lives at `hooks/jsonl-to-md.py`, and the maintainer's `~/.claude/jsonl-to-md.py` is a symlink pointing at it. `save-session.sh`'s reference path is unchanged; on the end-user install, the installer will place the helper at `~/.claude/jsonl-to-md.py` alongside the scripts.
- [x] Refine `check-memory-synthesis.sh` to handle the two path classes that the default reminder does not fit. **`MEMORY.md`** → no output (the index has nothing to synthesize or sanitize). **`user_*.md`** → path-aware reminder: the synthesis half still applies ("is this addition at home in this user profile, or does it belong in a feedback memory?") but the sanitization half is dropped, since named attribution is the point of user profile memories. **Both exclusions live in the script** rather than in the hook entry's `if` field: the original design specified `MEMORY.md` exclusion via `if`, but Claude Code's permission-rule syntax supports prefix matches and not negation, so expressing "any memory path EXCEPT `MEMORY.md`" as a positive prefix requires enumerating the file-name prefixes that DO match, which is brittle. The script's case statement handles all three path classes directly. Pipe-tested for `MEMORY.md`, `user_*.md`, regular memory bodies, and non-memory paths.

## Installer

- [x] Fill in `install.sh`. Implements `--copy` (default) and `--link` modes plus `--dry-run`. Non-destructive: existing files are skipped, never overwritten. Settings.json: writes directly if absent, prints snippet for manual merge if present. `--force` is not yet implemented; defer to a future enhancement (the manual-mv-aside workflow covers the current need). The maintainer's setup of pre-symlinked files is correctly recognised as already-installed (every `install_file` reports "skip (exists)").
- [x] Write `hooks/settings.json.snippet`. The fragment mirrors the maintainer's current `~/.claude/settings.json` hooks block: `PreCompact` → `save-session.sh`, `SessionStart` → `session-start.sh`, `PreToolUse` on `Write|Edit` → `check-memory-synthesis.sh`. The installer's job is to deep-merge this into the user's existing `settings.json`.
- [x] Post-install message mentions the no-hot-reload fact: the user should restart Claude Code or open `/hooks` to activate the new hooks.
- [x] **Symlink install mode (`--link`).** Implemented in `install.sh`. Copy-install is the default for end users; `--link` is for developers and contributors who want their edits to flow back into Shannon's source. Per-file symlinks point `~/.claude/memory/<seed>.md` at `<shannon-checkout>/memory-seed/<seed>.md` (and the same for `~/.claude/<name>.sh` → `<shannon-checkout>/hooks/<name>.sh`, and `~/.claude/CLAUDE.md` → `<shannon-checkout>/claude-md/CLAUDE.example.md`). Editing a memory or hook then directly modifies the Shannon source. The installer rejects `--link` on filesystems without symlink support (probes with a test symlink to `/dev/null`); developers on Windows should use WSL. Existing files at the destination are SKIPPED rather than overwritten — manual move-aside is the current workflow for resolving conflicts.
- [x] One-time memory-seed reconcile for the maintainer's setup. Each of the four seed memories that overlapped with `~/.claude/memory/` — `feedback_memory_size_budget.md`, `feedback_rich_memory_summaries.md`, `feedback_external_reports.md`, and `feedback_memory_vs_skill.md` — now exists as a symlink in `~/.claude/memory/` pointing at the Shannon source. No drift possible for these files.
- [x] One-time hook-script reconcile for the maintainer's setup. All three scripts (`check-memory-synthesis.sh`, `session-start.sh`, `save-session.sh`) now live in `shannon/hooks/`; the maintainer's `~/.claude/<name>.sh` paths are symlinks pointing at them. No drift possible.
- [ ] **Idea (option B): content-hash manifest install** (dpkg-conffile pattern), as an alternative for the *end-user* install mode. The installer copies seeds into `~/.claude/memory/` *and* records source hashes in a sidecar manifest (`~/.claude/memory/.shannon-manifest.json`). On `shannon update`: if the user's local file is unchanged from the recorded hash, auto-update; if diverged, prompt or offer a 3-way merge. Strictly better than "skip if exists" because it surfaces drift to the user instead of letting it accumulate silently. Considered complicated for v1 — kept as a future enhancement.
- [ ] **Filed-elsewhere (option C): Anthropic feature request for native multi-directory memory loading.** Drafted but not filed yet. If accepted, this obsoletes both A and B by letting Claude Code load from a list of memory directories natively — no copy or symlink step.

## CLAUDE.md template

- [x] Write `claude-md/CLAUDE.example.md`. Covers the four candidate imperatives (synthesis-check before memory writes, no-push-without-explicit-request, attribution for assisted artifacts, scrub project-identifying details in global memories and external reports) plus a closing "adding rules to this file" guard that keeps the template minimal over time.

## Seed memories — remaining

- [ ] `feedback_no_push_without_request.md` (port from `~/.claude/memory/`, sanitize).
- [ ] `feedback_commit_coauthor.md` (port + sanitize; the per-org short-form rule must be replaced with generic guidance, since it is specific to the originating user's repos).
- [ ] `feedback_factor_hook_scripts.md` (port + sanitize).
- [ ] `feedback_silent_progress_polling.md` — port, or move to the opt-in tier if narration thresholds are too user-specific.

Already written: `feedback_memory_size_budget`, `feedback_rich_memory_summaries`, `feedback_external_reports`, `feedback_memory_vs_skill`, `feedback_shell_quoting_review`.

## Opt-in memories tier

- [ ] Decide the mechanism: subdirectory under `memory-seed/`? Separate top-level dir? How does the installer let users opt in?
- [ ] Fill in the README "Opt-in memories" TBD section once the mechanism is settled.
- [ ] Likely initial candidates: git-cluster memories, narration discipline, commit-message conventions, prose-style preferences.

## README placeholders

- [x] Resolve `<owner>` in the quick-start `git clone` URL once canonical home is decided.
- [ ] Fill in the "How current LLMs do and don't remember things" TBD section — background on context vs memory-files, context limits, what compaction is, why compaction can be lossy, the role of the harness.
- [ ] Fill in the "Opt-in memories" TBD (depends on the opt-in mechanism above).
- [ ] Decide whether to split a `docs/design-principles.md` out of the README when length crosses a threshold.

## Other docs

- [ ] Write `docs/extending.md` — referenced from the README "Extending" section, does not exist yet.
- [ ] Consider `docs/installer-caveats.md` for the no-hot-reload fact, the settings-watcher subtlety, and other harness-loading semantics relevant to the installer.

## Commits

- [x] Commit the initial accumulated state — done in `fe8b46f`. Scope: contributor `CLAUDE.md` extensions (sanitization rules, tests-expected, task-body-pointer); `Memories vs skills` section in the README + `feedback_memory_vs_skill.md` seed; `feedback_external_reports.md` seed; `docs/testing.md`; the friction-reduction principle in the README; `TODO.md` (this file).
- [ ] Commit the further state accumulated since `fe8b46f`. Includes: installer-section design captures in `TODO.md` (option A current direction, option B as future enhancement, option C draft pointer); `docs/testing.md` additions for the `MEMORY.md` and `user_*.md` exclusion cases; contributor `CLAUDE.md` extension recording the *match-in-script vs match-in-`if`-field* hook-design rule. (The seed-memory symlink reconcile lives outside the Shannon repo — no commit needed there.)
