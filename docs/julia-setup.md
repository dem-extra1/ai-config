# Installing Julia in Claude Code on the web

Cloud (web) sessions start from an ephemeral container that has **no Julia**
by default, and outbound network access is gated by the environment's network
policy. Installing Julia without first allowlisting the download hosts fails
with `HTTP 403 host_not_allowed`.

## How it's wired up here

This repo's `SessionStart` hook (`.claude/hooks/session-start.sh`) installs
Julia automatically. After it symlinks the config into `~/.claude/`, it runs
`juliaup` (the official Julia installer). The install is:

- **guarded / idempotent** — a no-op once `juliaup` or `julia` is already on
  PATH, so it only does real work on a fresh container's first startup;
- **non-fatal** — if the network allowlist is missing, the `403` is swallowed
  (with a warning to stderr) so config symlinking still succeeds; you just
  don't get Julia.

So the **only** thing you must configure at the environment level is the
**network allowlist** below — the hook handles the install itself.

## Network allowlist (required)

The install only succeeds if the environment's network policy permits these
hosts:

| Host | Why |
|---|---|
| `install.julialang.org` | the install one-liner |
| `julialang-s3.julialang.org` | Julia binary tarballs |
| `github.com`, `objects.githubusercontent.com` | the `juliaup` binary (GitHub Releases) |
| `pkg.julialang.org` | only if you later use `Pkg` to install Julia packages |

If the policy supports domain globs, `*.julialang.org` + `*.githubusercontent.com`
covers most of it. Set this where the environment is defined — see the
[Claude Code on the web docs](https://code.claude.com/docs/en/claude-code-on-the-web).

## Alternative: install at build time

The `SessionStart` hook installs Julia on the first session start, which adds a
one-time delay to that startup. To pay the cost at **build time** instead (so
sessions start with Julia already present), paste the same logic into the
environment's **Setup script** field. The hook's guard makes the two
approaches compose safely — whichever runs first wins, the other is a no-op:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Install Julia via juliaup (official installer). --yes = non-interactive.
if ! command -v juliaup >/dev/null 2>&1; then
  curl -fsSL https://install.julialang.org | sh -s -- --yes
fi

# juliaup installs its shims into ~/.juliaup/bin; persist on PATH for shells.
if ! grep -qs '.juliaup/bin' "$HOME/.bashrc" 2>/dev/null; then
  echo "export PATH=\"\$HOME/.juliaup/bin:\$PATH\"" >> "$HOME/.bashrc"
fi

# Optional: pin a version instead of the rolling 'release' channel
# juliaup add 1.11 && juliaup default 1.11

julia --version
```

## Notes

- The network allowlist is **environment** config, not repo config — it lives
  where the environment is defined, not in `ai-config`.
- To pin a Julia version rather than track the rolling `release` channel, run
  `juliaup add <version> && juliaup default <version>` (see the commented line
  above).
