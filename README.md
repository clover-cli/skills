# Clover skills

Agent skills for working on [clover-cli/cli](https://github.com/clover-cli/cli).

| Skill | What it does |
| --- | --- |
| `clover-triage` | Splits an issue into atomic PR tasks |
| `clover-implement` | Implements one task as one PR |
| `clover-ship` | Ships a whole issue as a stack of PRs |
| `clover-review` | Reviews a PR (never merges) |
| `clover-verify` | Checks every PR made for an issue, alone and as a series |
| `clover-reformat` | Rewrites a PR's code to be maintainable, without changing behavior |

## Install

```sh
git clone https://github.com/clover-cli/skills ~/.hermes/skills/clover
```

Each skill is a `SKILL.md` and works with any agent that reads that format.
