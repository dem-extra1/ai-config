---
name: claim-pr
description: Post a "paws off" claim comment on a GitHub PR or issue before starting a work session on it, and an unclaim comment when done, so other humans and the @claude CI bot don't start a colliding parallel session. Use before fetching a branch, editing, or running review cycles on a PR/issue — and after the work is paused, merged, or closed.
user-invocable: true
allowed-tools:
  - Bash
---

# claim-pr

Before working a GitHub PR or issue — fetching its branch, editing, or running
`@claude` review cycles — post a brief comment so other people and the
`@claude` CI bot know not to start a conflicting parallel session. Post a
closing comment when the session ends so it's free for the next person.

## When this fires

- Before any **write** session on a PR/issue: fix, implement, debug, refactor,
  review-and-edit, or an iterative `@claude review` loop that pushes commits.
- Triggered by a prompt referencing a PR/issue by `#N` or URL that asks you to
  *change* something.

It does **NOT** fire for read-only inspection — "show me PR #X", "what's the
status of #Y", "explain the diff on #Z". Those don't risk a parallel session.

## Claim (start of session)

First check whether you've already claimed it — if your (Claude's) most recent
comment on the thread already says you're working on it, **skip** re-posting.

```bash
# PR:
gh pr comment <N> --body "Claude Code CLI (local session) is working on this — paws off until I'm done."
# Issue:
gh issue comment <N> --body "Claude Code CLI (local session) is working on this — paws off until I'm done."
```

Then proceed with the work.

## Unclaim (end of session)

After the work is done (PR merged, issue closed) or paused, post a short
closing comment so the thread is unclaimed:

```bash
gh pr comment <N> --body "Done with my local session — unclaiming."
gh issue comment <N> --body "Done with my local session — unclaiming."
```

## Notes

- If `@claude` agent runs are in flight on the branch, wait for them before
  pushing or polling — don't edit while the bot is mid-session.
- This is the claim ritual referenced by `iterate` (step 1) and `iterate-all`;
  when those run, they cover the claim for you.
