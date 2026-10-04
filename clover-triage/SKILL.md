---
name: clover-triage
description: "Use when triaging clover-cli issues into atomic PR tasks."
version: 1.0.0
---

# Clover Triage Bot

Repo: clover-cli/cli, local clone ~/clover/cli, default branch main.
Read ~/clover/cli/AGENTS.md first; it is the source of truth for layout and rules.

## Job

Given an issue number (or "all open issues"), produce a plan of atomic tasks. Read-only: never edit files, push, comment, or create issues unless the user explicitly asks.

1. `gh issue view <N> -R clover-cli/cli --comments` (or `gh issue list -R clover-cli/cli --state open`).
2. Check what already exists: `gh pr list -R clover-cli/cli --search "<N>" --state all` and grep the code.
3. Split into tasks where each task is ONE PR that:
   - does one thing, fits one commit message `<type>: <what>` from AGENTS.md;
   - is reviewable in a few minutes (aim < ~150 changed lines, tests included);
   - leaves main building, testing and linting green on its own.
   Follow the AGENTS.md "Adding a service" order when relevant (provider -> commands -> register -> IAM -> tests -> docs), merging steps only when splitting would break the build.
4. Output a numbered list: title (as commit message), files touched, dependencies on earlier tasks, and the parent issue it references.

If an issue is already small, say so: one task. Don't invent work the issue didn't ask for.

Hand off: the user runs /clover-implement with an issue + task number.
