---
name: clover-verify
description: "Use when verifying all PRs made for a clover-cli issue."
version: 1.0.0
---

# Clover Verify Bot

Given an issue N, find every PR made for it and check that each one is valid, alone and as a
series. Read-only: never edit, push, comment, approve, merge or close unless the user asks.

Repo: clover-cli/cli, local clone ~/clover/cli. Read ~/clover/cli/AGENTS.md first. Load
clover-review (per-PR checklist) and clover-implement (PR conventions) with skill_view.

## 1. Collect

- Issue: `gh issue view N -R clover-cli/cli --json title,body,state,comments` (plain `gh issue view`
  sometimes prints nothing; use `--json`).
- PRs: `gh pr list -R clover-cli/cli --state all --limit 100 --search "N" --json number,title,body,state,isDraft,baseRefName,headRefName,mergedAt`,
  keep those whose title ends with `(#N, k)` or whose body says `Part of #N` / `Closes #N`. Also
  check `closingIssuesReferences`. Order by k. It stays empty for PRs whose base isn't the default
  branch (stacked PRs); that's expected, not a finding. GitHub links it once the PR retargets to main.

## 2. Check the series

- Positions are 1..n with no gaps or duplicates; every PR in the series has the `(#N, k)` suffix.
- Exactly one PR closes the issue, and it's the last one (k = n). The others say `Part of #N`.
- Stacked PRs say `Depends on #<pr of k-1>` and their base is that PR's branch (or `main` once it merged).
- Open PRs are not drafts. Merged PRs are actually on main (`git merge-base --is-ancestor` or the
  change is present after a squash). If every PR merged, the issue is closed.
- Coverage: the union of the diffs does everything the issue asks. List gaps and extra work the
  issue didn't ask for.

## 3. Check each PR (in k order)

- clover-review checklist: atomic, title/commit format `<type>: <what>`, AGENTS.md rules (provider-only
  network calls, action/print/info/confirm, --output json, --yes, throw on error, comment rule, tests
  use fakeClient/fakeGcpClient + runCli), tests cover the change, nothing over-built.
- No AI attribution in title, body or commits (`gh pr view <n> --json commits`). Author is the user.
- Builds on its own: `git worktree add ~/clover/verify-<n> <head sha>`, symlink or `npm ci`, then
  `npm run build && npm test && npm run lint`. It must pass at its own position in the stack, not
  only at the tip. Remove the worktree after.
- Diff size: compare against the merge base (`git diff origin/<base>...origin/<head>`, three dots).
  A two-dot diff against a main that moved on shows unrelated changes.
- CI: `gh pr checks <n>` (merged PRs: the merge commit's checks).

## 4. Double-check every finding

Before reporting a problem, prove it: re-read the exact lines, reproduce the failure (run the test,
the command, or `node dist/index.js ...`), or show the rule it breaks. Drop anything you can't
confirm, and say which suspicions were dismissed and why. Don't report style preferences as errors.

## Output

1. Verdict for the issue: valid / needs changes.
2. Table: k, PR, state, base, checks (build/test/lint/CI), verdict.
3. Confirmed findings only, each with PR, file:line, what's wrong, and evidence (command + output).
4. Coverage gaps, then dismissed suspicions in one line each.

Offer fixes as a follow-up (clover-implement for new PRs, clover-reformat for cleanups); don't make them
unasked.
