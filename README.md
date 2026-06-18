# claude-config

Portable Claude Code config — synced across machines via git.

Each top-level subdir mirrors a directory under `~/.claude/`. `bootstrap.sh`
symlinks each one into place.

## Setup on a new machine

```sh
git clone <this-repo-url> ~/Documents/GitHub/claude-config
bash ~/Documents/GitHub/claude-config/bootstrap.sh
```

Rerun `bootstrap.sh` any time a new top-level dir is added to the repo
(e.g., `agents/`, `commands/`).

## Use as a Claude Code plugin (web / remote sessions)

`bootstrap.sh` only covers local machines. Claude Code on the web starts each
cloud session in a fresh container where `~/.claude` is empty and
`bootstrap.sh` never runs — and skills uploaded to claude.ai/customize do **not**
cross over into Claude Code. So this repo also publishes itself as a **plugin
marketplace**, the supported way to load these skills in cloud sessions.

The repo is simultaneously:

- the marketplace — `.claude-plugin/marketplace.json`
- a single plugin — `.claude-plugin/plugin.json` with `source: "./"`, which
  bundles the existing top-level `skills/` and `commands/` (no duplication;
  `skills/` and `commands/` are auto-discovered at the plugin root).

To load these skills in another repo's cloud sessions, commit this to **that
repo's** `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "d-morrison": {
      "source": { "source": "github", "repo": "d-morrison/ai-config" }
    }
  },
  "enabledPlugins": {
    "ai-config@d-morrison": true
  }
}
```

Claude Code installs the plugin at session start (needs network access to reach
GitHub). Plugin skills are namespaced, e.g. `/ai-config:reprexes`,
`/ai-config:grade-work`.

Locally (or to try it), add and install interactively:

```sh
/plugin marketplace add d-morrison/ai-config
/plugin install ai-config@d-morrison
```

No `version` is pinned, so every commit to this repo counts as a new version —
sessions with marketplace auto-update pick up the latest automatically.

## What's tracked

- `skills/` — user-level skills (`~/.claude/skills/`)
- `commands/` — user-level slash commands (`~/.claude/commands/`)

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
