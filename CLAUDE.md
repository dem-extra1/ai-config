# Claude Instructions for ai-config

This repository stores Claude Code configuration — skills, commands, and workflow settings.
On a developer's local machine, `bootstrap.sh` symlinks `skills/` → `~/.claude/skills/` and
`commands/` → `~/.claude/commands/`. In GitHub Actions, the skills live at `skills/` in the
project root.

## Skills

The `skills/` directory contains user-invocable skills. Each skill is defined by a
`skills/<name>/SKILL.md` file with a YAML frontmatter header and Markdown instructions.

### Invoking a skill

When you receive a prompt of the form `@claude <skill-name>` or just `<skill-name>` on its own,
check whether `skills/<skill-name>/SKILL.md` exists. If it does, read that file and follow its
instructions exactly. If it doesn't exist, tell the user the skill wasn't found and list the
available skills below.

### Available skills

| Skill | Description |
|-------|-------------|
| `claim-pr` | Post a "paws off" claim comment before working a PR/issue, unclaim when done |
| `claude-agent-workflow` | Add or modify the `@claude` agent GitHub Actions workflow |
| `claude-review-workflow` | Add or modify the Claude PR review workflow |
| `defer-issue` | File a follow-up GitHub issue when deferring work out of scope |
| `grade-work` | Grade student submissions against a solution; produce ranked common-errors catalog |
| `iterate` | Drive a PR to a clean review verdict (loop: review → fix all findings → re-review) |
| `iterate-all` | Apply `iterate` to every open PR in the repo |
| `merge-main` | Alias for `sync-pr-branch` — merge origin/main into the current PR branch |
| `plan-review-session` | Turn a common-errors catalog into a review-session chapter and open a PR |
| `pr-status` | Report a PR's true review status (reads the latest review, not a cached verdict) |
| `pr-status-all` | Print a status table for every open PR |
| `r-pkg-spellcheck` | Run R-package spellcheck before pushing changes that touch user-facing text |
| `reprexes` | Isolate a bug into a minimal reproducible example and iterate fixes on it |
| `request-pr-review` | Request d-morrison as reviewer after creating a GitHub PR |
| `sync-pr-branch` | Bring a PR branch up to date with main (merge origin/main, resolve conflicts, push) |
| `workaround-watcher` | Scaffold a scheduled workflow that auto-opens a revert PR when an upstream bug is fixed |
