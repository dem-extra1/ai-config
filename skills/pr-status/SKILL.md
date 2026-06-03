---
name: pr-status
description: Report a PR's true review status by reading the LATEST review comment, not a cached or earlier verdict, and parse it for any remaining findings before declaring "clean" / "ready to merge". Use when asked "what's the status of PR #N", "is this PR ready to merge", or before you report any PR as mergeable. Handles the @claude bot login variants so you never false-pass on a stale or null read.
user-invocable: true
allowed-tools:
  - Bash
---

# pr-status

Report a PR's review status honestly: based on the **most recent** review, with
its findings actually parsed — never on an earlier cached "verdict." A newer
review may have landed since (from the `@claude` bot, a human, or a
re-trigger), and it may carry findings the old one missed.

## When this fires

- "what's the status of PR #N", "is #N ready to merge", "is the review clean".
- Before you state, anywhere, that a PR is mergeable / clean / ready.

## CI green ≠ review clean

`gh pr checks <N>` going green is about **CI state**, not the review verdict. A
PR can have all checks passing and still have unaddressed review findings.
Always parse the latest **review body** for findings — don't infer "clean"
from green checks.

## Read the LATEST review

```bash
gh pr view <N> --json comments \
  --jq '[.comments[] | select(.author.login | startswith("claude"))] | last | .body'
```

The reviewer's bot login **varies by API and setup**:

- `gh pr view` reports it as `claude`
- the REST API (`gh api .../comments`) reports it as `claude[bot]`
- some setups post reviews as `github-actions[bot]`

`startswith("claude")` matches the @claude bot across both `gh pr view` and
`gh api`. If your reviewer posts under a different login (e.g.
`github-actions[bot]`), **broaden the filter** — otherwise the `--jq` returns
`null` and you silently false-pass a PR with open findings. (Structured MCP
GitHub tools like `mcp__github_ci__get_ci_status` are an alternative where the
`gh` JSON parsing gets fragile.)

## Parse for findings before declaring clean

Read the full latest review body and scan for any "Findings", "Issues",
"Remaining", "Non-blocking", "Minor", "Could improve", "Consider", etc.
section. The bar for reporting **clean**: "Looks good" / "no findings" /
"approved" with **zero** follow-on bullets under any heading.

Do **not** report "ready to merge with one minor nit noted" / "harmless
as-is" / "can address if you want" — that hedging just pushes triage back to
the user. If there are open items, report them as open (and offer to run
`iterate` to clear them).

## Output

State, plainly: the latest review's verdict, who/what posted it, and the list
of any open findings (or "none"). If you read `null`, say the filter didn't
match a reviewer login — don't report it as clean.
