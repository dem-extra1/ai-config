---
name: sparta-gotchas
description: "Operational gotchas and reviewer conventions for Lacaedemon/sparta (Godot tactical battle game)"
metadata:
  type: feedback
---

# Sparta — working notes

## Website docs scope in stacked PRs

Sparta requires user-facing PRs to update the `website/` docs (the website-update
policy in the repo's `CLAUDE.md`). That requirement makes it easy to over-document:
on a stacked PR, write docs only for features whose code is on the *current branch's*
ancestry, not for a sibling branch's feature.

This is the sparta instance of the general rule in `preferences.md` ("only document
features present on the current branch's ancestry — grep first").

**Concrete case:** in the terrain-speed PR (#185), website docs were written for the
order-response delay feature (from `feat/order-response-delay`, a separate branch also
targeting `main`). That code was never in `feat/terrain-speed`'s ancestry, so the
reviewer correctly flagged it as a "hallucinated feature." Before documenting a feature,
`grep` for its symbol/constant (e.g. `order_response_delay`) on the current branch; if
it's absent, move the docs to the branch where the code lives.

## Demo scenario design — battle mechanics cheat sheet

When writing a `demos/scenarios/*.json` replay, these mechanics determine timing:

**Auto-advance**: Only team 1 (enemy AI, `_run_enemy_ai()`) advances automatically.
Team 0 (player units) is **stationary** until given explicit orders. Any scenario
that needs team 0 units engaged must include a move order at tick 0 (or early).

**Unit positions** (default 5v5, seed 12345, `FIELD = Rect2(0,0,1600,1000)`):
- Team 0 spawns at y=300, team 1 at y=700; spacing=150px, start_x=500
- UIDs (team 0): 0=Spearmen, 1=Infantry, 2=Archers, 3=Cavalry, 4=Cavalry
- UIDs (team 1): 5=Spearmen, 6=Infantry, 7=Archers, 8=Cavalry, 9=Cavalry

**Effective speeds** (base_spd × SPEED_SCALE 0.6):
- Spearmen: 48 px/s | Infantry: 54 px/s | Archers: 57 px/s | Cavalry: 96 px/s

**Engagement timing** (both teams advancing toward each other, 400px gap):
- Spearmen vs Spearmen: ~4.2 s (96 px/s combined) — but only if team 0 has a move order
- Cavalry vs Cavalry: ~2.1 s (192 px/s combined) — cavalry auto-engages fast

**Order `target` field semantics** (from `Replay.gd`):
- `-1` → plain move to (x, y)
- `-3` → formation-only change (no movement)
- friendly uid (same team, not in `units[]`) → line relief (`begin_relief`)
- enemy uid (different team) → attack order

**`max_frames`**: video frames at 30 fps (not physics ticks). 480 = 16 s, 600 = 20 s.

**Scenario validation process**: after writing a scenario, calculate engagement
timing on paper before relying on the CI clip to confirm — a wrong scenario wastes
a CI run and may silently record an unrelated clip.
