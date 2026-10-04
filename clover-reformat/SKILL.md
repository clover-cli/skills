---
name: clover-reformat
description: "Use when reformatting a clover-cli PR for maintainability."
version: 1.0.0
---

# Clover Reformat Bot

Given a PR, rewrite its code so it is highly maintainable and easy for an agent to read and change.
This is mandatory, not a suggestion: always produce the reformat commit unless the code already meets
every rule below (then say so, rule by rule). Behavior must not change.

Repo: clover-cli/cli, local clone ~/clover/cli. Read ~/clover/cli/AGENTS.md first. All
clover-implement hard rules apply (user identity, no AI attribution, never merge/approve/close,
never push to main, never force-push).

## Scope

Only the files and lines the PR adds or changes (`gh pr diff <n> --name-only`). Don't touch unrelated
code, even if it's messy; list it at the end as a suggestion.

## Rules

- Reuse what exists: helpers in `src/commands/aws/shared.ts`, `src/commands/<provider>/shared.ts`,
  `src/provider/<provider>.ts`, `test/helpers.ts`. Delete local copies of them.
- Match the sibling files: same structure, naming, option names and argument order (client first in
  provider functions) as the closest existing service.
- One concept per file; file and export names say what they are. Descriptive names, no abbreviations.
- Small functions with one job; early returns over nesting; no dead code, unused exports or params.
- No magic values repeated: one named constant. No one-use abstractions, interfaces or factories.
- Types explicit at module boundaries (exported functions); no `any`; no unneeded casts.
- Comments: follow AGENTS.md. Delete comments that restate the code and file-header comments; keep
  only non-obvious "why".
- Tests: named by behavior, one behavior each, shared setup through test/helpers.ts.
- No new dependencies. No changes to CLI flags, output, exit codes or API requests.

## Steps

1. `cd ~/clover/cli && git status`. If dirty, stop and ask. Note the current branch.
2. `gh pr view <n> --json state,headRefName,baseRefName`. Work in a worktree:
   `git worktree add ~/clover/reformat-<n> origin/<head>` (merged/closed PR: branch
   `update/reformat-<slug>` off origin/main instead).
3. Before editing, record a baseline: `npm run build && npm test` (save the test count) and the
   `--help` output of every command the PR touches (`node dist/index.js <provider> <service> <action> --help`).
4. Reformat per the rules.
5. Verify: `npm run build && npm test && npm run lint` pass with zero warnings; test count is not
   lower; every saved `--help` output is identical; re-read the diff and confirm each change keeps
   behavior (same requests, same printed output).
6. Commit `update: reformat <what>` (one line, no trailers). Push as a new commit on the PR branch
   (never force-push; never rewrite the user's commits). For a merged/closed PR, push the new branch
   and open a PR ready for review titled `update: reformat <what> (#N, k)` reusing the original
   suffix, body: what was cleaned up, `Follows #<pr>`, checks that passed.
7. If the PR has open stacked children, merge its branch into each child (`git merge`, no force),
   run the checks there and push, so the stack stays consistent.
8. Report: commit/PR URL, what changed by rule, the checks, and out-of-scope suggestions. Remove the
   worktree and `git switch` back to the user's branch.
