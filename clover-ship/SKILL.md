---
name: clover-ship
description: "Use when shipping a clover-cli issue as many PRs."
version: 1.0.0
---

# Clover Ship Bot

Split an issue into atomic tasks, then implement each one as its own PR, in order.
This chains clover-triage and clover-implement: load both with skill_view and follow them.
Every clover-implement hard rule applies to every PR: open ready for review (not draft), the
user's identity, no AI attribution, never merge/approve/close, never push to main, never force-push a branch the
user touched.

Repo: clover-cli/cli, local clone ~/clover/cli. Read ~/clover/cli/AGENTS.md first.

## Steps

1. `cd ~/clover/cli && git status`. If the tree is dirty, stop and ask. Note the current branch so you
   can switch back to it at the end.
2. Triage issue N per clover-triage: read the issue and comments, check existing PRs and branches, grep
   the code. If the issue is small it's one task, and this is just clover-implement.
3. Show the plan as numbered tasks (commit title, files, depends on, `Part of #N` / `Closes #N`; only
   the last PR closes the issue), then proceed WITHOUT waiting for approval. Ask the user only when
   truly blocked: a decision that can't be defaulted, failing checks out of scope, or a dirty tree.
   Pick sensible defaults (fewest deps, reuse existing helpers) and state them in the report.
   Always ship EVERY task in the plan in one run; never stop after a first wave or wait for merges.
   When tasks depend on unmerged work, stack them (a linear chain is fine). Keep every PR as small
   as possible for review.
4. For each task, in order:
   - Base: `origin/main` if it's independent; otherwise the branch of the task it depends on (stacked).
   - `git switch -c <type>/<slug> <base>`. Make the smallest change, plus tests. Run
     `npm run build && npm test && npm run lint`.
   - `git diff --stat` to confirm it's atomic. `git commit -m "<type>: <what>"`. `git push -u origin HEAD`.
   - `gh pr create -R clover-cli/cli --base <main or parent branch> --title "<type>: <what> (#N, k)" --body ...`.
     Title ends with `(#N, k)`: the parent issue and the PR's position in the stack (1-based), e.g.
     `add: gcp provider core (#14, 1)`. The commit message stays `<type>: <what>` without it.
     Body: what and why in 1-2 lines; `Part of #N` or `Closes #N`; `Depends on #<pr>` if stacked;
     the checks that passed. No `Series:` line.
   - If build/test/lint fails and can't be fixed within the task's scope: stop. Leave the opened PRs
     as they are, don't push the failing branch, and report what's done and what's left.
5. Report the review order (the `(#N, k)` title suffix gives it): k, PR URL, title, base, +/- lines.
   Then `git switch` back to the user's original branch. Don't edit the PRs after opening them
   unless the user asks.

## Stacking

- Prefer independent PRs off main. Stack only when a task needs unmerged code from an earlier one.
  Sibling PRs that all touch the same registration file/docs table will conflict with each other;
  chain those linearly instead.
- Stacked PRs target their parent branch. When the parent merges and its branch is deleted, GitHub
  retargets the child to main. After a squash-merge the child may still need a rebase.
- Restack only when the user asks, and only branches the user hasn't touched: the remote tip must equal
  the commit this bot pushed (`git rev-parse origin/<branch>` matches the local branch). Otherwise ask
  first. Restack with `git rebase --onto origin/main <old-parent> <branch>`, then
  `git push --force-with-lease`.

## Pitfalls

- Big issues (many services): commit the shared helpers to a scratch base branch first, then have
  subagents build one service each in `git worktree`s off it (node_modules symlinked; add
  `node_modules` to .git/info/exclude because `node_modules/` in .gitignore doesn't match a symlink).
  Assemble the chain yourself, editing the shared registration file and docs per PR, and run the
  checks on every commit. Delete the worktrees and scratch branches afterwards.
- Each PR, merged in order, must keep main green on its own: no dead imports, no tests that only pass
  once the next PR lands.
- Don't pad the series. Work the issue didn't ask for goes in the final report as a suggestion, not a PR.
