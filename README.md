# ai-config

Portable AI agent config — skills, memories, and commands synced across
machines via git. Works with Claude Code, VS Code Copilot, and any agent
that reads markdown instruction files.

Each top-level subdir is symlinked into the appropriate consumer directory
by `bootstrap.sh`.

## Setup on a new machine

```sh
git clone https://github.com/d-morrison/ai-config.git ~/Documents/GitHub/ai-config
bash ~/Documents/GitHub/ai-config/bootstrap.sh
```

Rerun `bootstrap.sh` any time a new top-level dir is added to the repo.

## What's tracked

- `skills/` — reusable workflow skills (`~/.claude/skills/`)
- `commands/` — slash commands (`~/.claude/commands/`)
- `memories/` — persistent notes & preferences (symlinked into VS Code Copilot memory dir)

Add more by creating a top-level dir here (e.g., `agents/`,
`output-styles/`) and rerunning `bootstrap.sh`.

## What's deliberately NOT tracked

These are either machine-specific, sensitive, or pure session state:

- `settings.json` / `settings.local.json` — permission allowlists and
  `additionalDirectories` bake in absolute paths and per-machine choices.
- `sessions/`, `history.jsonl`, `tasks/`, `plans/`, `projects/` — session
  and per-CWD memory state, keyed by absolute home path.
- `cache/`, `shell-snapshots/`, `file-history/`, `ide/`, `telemetry/`,
  `backups/`, `downloads/`, `session-env/` — ephemera.
- `plugins/` — managed by Claude Code itself from marketplaces.

If a per-machine variation appears that's worth syncing (e.g., a global
`CLAUDE.md`), add it as a top-level entry here and update `bootstrap.sh`
only if it needs special handling beyond a directory symlink.
