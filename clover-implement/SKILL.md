---
name: clover-implement
description: "Use when implementing a clover-cli task as one PR."
version: 1.0.0
---

# Clover Implement Bot

Repo: clover-cli/cli, local clone ~/clover/cli, default branch main.
Read ~/clover/cli/AGENTS.md first and follow it exactly.

## Hard rules (from the user + AGENTS.md)

- ONE task = ONE branch = ONE PR. If the work is bigger, stop, implement only the first atomic piece, and list the rest.
- Commits and PRs come from the user's own identity only (git config / gh as andresdanielmtz). Never change git user.name/email.
- NO AI attribution anywhere: no `Co-Authored-By` trailers, no "Generated with", no bot signatures, in commits, PR titles, PR bodies or comments.
- Commit message: `<type>: <what>`, lowercase, one line, no body. Types per AGENTS.md (feat, add, fix, fix-docs, fix-lint, docs, update, remove, move, ver).
- NEVER merge, approve or close. Never push to main. Never force-push a branch the user has touched. The user reviews and merges.
- Open PRs ready for review (no `--draft`): the user reviews them as soon as they're up.

## Steps

1. `cd ~/clover/cli && git status` — if the tree is dirty, stop and ask; don't stash the user's work.
2. `git fetch origin && git switch -c <type>/<short-slug> origin/main`.
3. Read the issue: `gh issue view <N> -R clover-cli/cli --comments`. If it's large, apply the clover-triage split and pick the requested (or first) task.
4. Make the smallest change that solves the task (ponytail mindset: reuse helpers in shared.ts, no new deps unless the task is `add:` a dep). Add/adjust tests per AGENTS.md (fakeClient(), runCli(); never call AWS).
5. Run `npm run build && npm test && npm run lint`. All must pass. Fix or stop and report.
6. `git diff --stat` — sanity-check it's atomic. Commit with plain `git commit -m "<type>: <what>"` (no trailers).
7. `git push -u origin HEAD`.
8. Open the PR, ready for review:
   ```
   gh pr create -R clover-cli/cli --base main --title "<type>: <what> (#N, k)" --body "<body>"
   ```
   Title ends with `(#N, k)`: the parent issue and the task's position in the triage plan (1 for a
   single-task issue), e.g. `fix: region fallback (#20, 1)`. The commit message has no suffix.
   No `Series:` line in the body.
   Body (plain, short):
   - one or two lines on what changed and why;
   - `Closes #N` if this PR fully resolves the issue;
   - `Part of #N` if it's one piece of a larger issue (always reference the parent issue), plus `Depends on #<pr>` if stacked;
   - how it was tested (build/test/lint passed).
9. Report the PR URL and remaining tasks (if any). Then `git switch` back to the branch the user was on.

## Pitfalls

- Local npm strips `libc` fields and syncs the lockfile version on `npm install`. When adding a dep,
  keep package-lock.json additive only (restore those fields from origin/main) and check `npm ci` passes.
- Mocking a provider module globally in test/setup.ts also mocks it in that provider's own unit test.
  Add `vi.unmock('<path>')` there.

## Stacked tasks

If task B needs unmerged task A, branch B from A's branch, set `--base <A-branch>`, and write `Depends on #<A>` in the body. Prefer independent PRs off main when possible.
