# Work for upstream

Rules for a change that we offer to `AprilNEA/OpenLogi`. Sources: `AGENTS.md`, the section "GitHub workflow" of `docs/DEVELOPMENT.md`, `.agents/rules/ci.md`, and the merged PRs #2382, #2404 and #2405. Upstream has no `CONTRIBUTING.md`.

## Order

1. Search upstream for an open issue or PR on the same subject: `gh search prs --repo AprilNEA/OpenLogi "<words>"`, then the same with `gh search issues`. When a PR exists, test it on our hardware and report the result in it. Open no second PR for the same change. Why: the maintainer closes a duplicate. Done when you know each open PR and issue on the subject.
2. Run `git fetch upstream master`. Make the PR branch `type/kebab-description` from `upstream/master`, in a worktree. Done when `git log upstream/master..HEAD` is empty on the new branch.
3. Set the upstream of the branch before the first `ship`: `git config branch.<branch>.remote origin` and `git config branch.<branch>.merge refs/heads/<branch>`. Why: with no upstream, `ship` pushes a worktree branch to the default branch of the fork.
4. Make the change. Read the subsystem rule that the table at the end of `AGENTS.md` names for each changed path.
5. Run the local gate of `.agents/rules/ci.md` on the final tree, and `typos --config .config/typos.toml` on the changed paths. Done when each command passes. A skipped job is not a pass: name it in the PR.
6. Write the PR body. The owner approves each text before it goes to GitHub: a PR body, an issue, a comment.
7. Push with `ship`. Open the PR with `gh pr create --repo AprilNEA/OpenLogi --base master --head praisethebit:<branch> --title "<title>" --body-file <file>`. Done when the PR is open.
8. Read the Greptile review comment on the PR. Fix each finding, reply to it and resolve it. Why: the maintainer waits for the Greptile check and for CI before a merge.
9. When a maintainer pushes commits to the PR branch, pull them, test again and report the result.

## Rules for the code

- One subject in one PR. Keep the PR small. Why: the small PRs with hardware proof merged in less than one day, and the large G-series PRs got no review for months.
- Give hardware proof. State the device, the firmware, the connection and the exact command with its result.
- Follow an existing pattern, and name it in the PR body. Example: a new `0x8110` button layout follows the G903 layout of #2186.
- Deferred work becomes a linked upstream issue, not a TODO comment.
- A comment in a file for upstream follows the upstream style. Use no `why:` prefix there.
- Rust version: CI uses the current stable Rust. When the local stable is older, Clippy can fail on code that we did not change. Install the CI version with `rustup toolchain install <version>`, and run the gate with `RUSTUP_TOOLCHAIN=<version>`.

## Rules for the PR text

- Title: `type(scope): imperative lowercase description`. The types and scopes are in the section "Git & GitHub" of `AGENTS.md`.
- Body: `## Summary`, `## Changes` (one bullet for each crate), `## Testing` (the exact commands and the hardware state), then `Fixes #N` or `Part of #N`.
- In `## Testing`, write "not runtime-tested on hardware" when that is true, and say how to test the change.
- Add screenshots for a UI change.
- Write each text for GitHub in English.
- Add no AI attribution to a commit, a PR or an issue: no "Generated with" line and no AI co-author trailer. Why: `AGENTS.md` of upstream forbids it.

## Device data

Use the skill `.agents/skills/contributing-device-fixtures/SKILL.md`. The merged pattern is #2404. These facts are not in the skill:

- Phase 1 needs the OpenLogi Agent, and phase 2 needs the Agent stopped. Close the agent and the tray of G HUB or Options+ before both phases.
- The wizard records eight cases, and the corpus holds seven. After the wizard, delete `cases/firmware-entities.json`. In `manifest.json`, remove its entry in `cases` and its 2 occurrences in `identity_ledger` (receiver serial and unit ID). Then run `openlogi fixture verify <dir>`. Why: the corpus test stops on a firmware case.
- Add the reviewed expectations in the same PR: `crates/openlogi-cli/src/cmd/fixture/record_case/corpus_diagnostics.json`, the tab list in `crates/openlogi-desktop/src/app/tests/corpus.rs`, one section in `fixtures/README.md`, and the specimen and read counts in `docs/MOCK_DEVICE_TESTING.md`.
- Decode the expectations from the recorded response payloads, and compare them with live `openlogi diag features`, `openlogi diag controls` and `openlogi diag battery` reads. Do not copy them from the replay output.
- Tests: `cargo test -p openlogi-cli -p openlogi-agent-core -p openlogi-agent -p openlogi-desktop contributed_corpus` and `cargo test -p openlogi-cli fixture::verify`.
