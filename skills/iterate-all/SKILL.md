---
name: iterate-all
description: Apply the `iterate` skill to every open PR in the repo — drive each one to a clean review verdict in turn. Use when asked to "iterate all PRs", "carry every open PR to clean", "review-loop all my PRs", or to run the review-until-clean loop across the whole open-PR queue rather than a single PR.
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
---

# iterate-all

Run the [`iterate`](../iterate/SKILL.md) loop over **all** open PRs in the
repository, one PR at a time, then report a per-PR status table. This is a thin
orchestrator: every per-PR rule lives in `iterate` (claim → sync main →
request review → address **every** flagged item → re-request → repeat until
zero findings). Do not reimplement that loop here — follow it for each PR.

## When this fires

- "iterate all PRs", "iterate every open PR", "carry all my PRs to clean".
- "run the review loop across all open PRs", "review-loop the whole queue".
- Any request to apply the review-until-clean process to the open-PR queue
  rather than a single named PR.

## Step 1 — enumerate the open PRs

List the open PRs and decide which are in scope:

```bash
gh pr list --state open --limit 100 \
  --json number,title,headRefName,isDraft,author,reviewDecision
```

Scope rules (state them when you report, so the user can correct):

- **Skip drafts** by default (`isDraft: true`) — they aren't ready for the
  clean-verdict bar. Include a draft only if the user explicitly asks.
- **By default, only iterate PRs the user owns / is responsible for.** In a
  shared repo, don't start review loops (which push commits) on other people's
  PRs unless the user says to. If unsure who owns what, ask before touching
  PRs authored by someone else.
- If the list is empty, say so and stop — nothing to do.

Report the in-scope list (with bare PR URLs) before you start, so the user can
veto any before the loop pushes commits.

## Step 2 — iterate each PR in series

Process PRs **one at a time**, not concurrently. Each PR's `iterate` run
pushes commits, triggers `@claude` review workflows, and polls for the result;
running them in parallel would interleave pushes, collide on shared review
runners, and make the per-PR status illegible. Series keeps each loop's claim,
sync, and latest-review-only reads correct.

For each PR, run the full `iterate` loop to its terminal state:

- **Clean** — reviewer returns zero flagged items under any heading. Post the
  unclaim comment, record the round count.
- **Asymptotic noise** — per `iterate`'s guard, if after 3–4 rounds the
  reviewer keeps emitting *new* nits, stop that PR, record it as "stalled
  (noise)", and move on. Don't let one PR block the rest of the queue.
- **Blocked** — needs a human decision, has unresolvable conflicts, or fails
  preflight in a way your change didn't cause. Record what's blocking and move
  on; don't silently skip it.

Keep going to the next PR after each terminal state — one PR stalling or
blocking must not abort the batch.

## Step 3 — report

End with a per-PR status table. Link each PR number to its URL (bare URL form):

| PR | Title | Rounds | Final state |
|----|-------|--------|-------------|
| https://github.com/<owner>/<repo>/pull/<N> | … | 2 | clean |
| https://github.com/<owner>/<repo>/pull/<M> | … | 4 | stalled (noise) — open items: … |
| https://github.com/<owner>/<repo>/pull/<K> | … | 1 | blocked — needs human decision on … |

For any PR not driven to clean, list its remaining open items so triage is one
glance, not a re-investigation. Don't merge anything — opening merges is the
user's call.

## Recurring / unattended runs

If asked to keep the queue clean on an interval, drive this skill from a
recurring runner (e.g. the `loop` skill) rather than busy-waiting inside one
invocation. Each tick re-enumerates open PRs (new ones appear, merged ones drop
off) and runs the series loop over the current set.
