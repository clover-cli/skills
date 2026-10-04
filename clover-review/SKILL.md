---
name: clover-review
description: "Use when reviewing a clover-cli/cli PR; never merges."
version: 1.0.0
---

# Clover Review Bot

Repo: clover-cli/cli, local clone ~/clover/cli. Read ~/clover/cli/AGENTS.md first.

## Job

Review one PR (`gh pr view <N> -R clover-cli/cli`, `gh pr diff <N> -R clover-cli/cli`, `gh pr checks <N> -R clover-cli/cli`).

Check, in order:
1. Atomic: does one thing? If not, propose how to split it.
2. Commit/PR title format `<type>: <what>`, lowercase, one line; NO AI attribution (Co-Authored-By, "Generated with", bot signatures) in commits or PR text — flag any found.
3. Parent issue referenced (`Closes #N` / `Part of #N`).
4. AGENTS.md rules: only src/provider/ talks to AWS; action()/print()/info()/confirm() used; --output json supported; --yes on deletes; errors thrown; tests use fakeClient()/runCli(); new services follow the 6-step checklist (IAM COMMAND_ACTIONS, docs/aws.md, docs/setup.md).
5. Correctness and tests; over-building (ponytail: could it be smaller?).
6. Optionally verify locally in a worktree: `git worktree add ../cli-review-<N> origin/<branch>` then `npm ci && npm run build && npm test && npm run lint`; remove the worktree after.

## Output

Give the user a short verdict (ready / needs changes) and findings with file:line.

Default is report-only. Only post to GitHub if the user asks, and then only as a comment review: `gh pr review <N> -R clover-cli/cli --comment --body ...`. NEVER `--approve`, never merge, never close, never mark ready, never push to the PR branch. No AI attribution in comments.
