---
name: cancel-superseded
description: "Cancel superseded (older) pipelines on the same branch, freeing runners for the latest push. Use when asked to 'cancel old pipelines', 'cancel superseded', 'free up runners', or when you notice stale pipelines blocking newer ones."
user-invocable: true
allowed-tools:
  - Bash
---

# Cancel Superseded Pipelines

Cancel older pipelines on the same branch that have been superseded by a newer push,
freeing CI runners for the latest pipeline.

> **GitLab-only.** This skill drives `glab` against GitLab CI pipelines. There
> is no GitHub equivalent here — on a GitHub repo, cancel superseded runs with
> `gh run cancel <run-id>` instead (or rely on per-PR `concurrency` in the
> workflow, which auto-cancels superseded runs).

## When to use

- After pushing a fix commit when an older pipeline on the same branch is still running
- When runners are busy and newer pipelines are stuck in `pending`
- When asked to "cancel old pipelines", "free up runners", or "cancel superseded"
- Proactively during ARDI loops when multiple pushes happen in quick succession

## Procedure

0. **Resolve the project ID once** (so the rest of the steps reference
   `$PROJECT_ID` instead of a manual placeholder):

```bash
PROJECT_ID="$(glab api "projects?search=$(basename "$(git rev-parse --show-toplevel)")" 2>/dev/null | \
  python3 -c "import json,sys; print(json.load(sys.stdin)[0]['id'])")"
echo "PROJECT_ID=$PROJECT_ID"
```

If the search returns more than one project (same repo name in different
groups), set `PROJECT_ID` by hand from `glab api "projects?search=<name>"`.

1. **Identify the current branch and its latest pipeline:**

```bash
BRANCH=$(git branch --show-current)
# Get pipelines for this branch, newest first
glab api "projects/$PROJECT_ID/pipelines?ref=$BRANCH&sort=desc&per_page=10" 2>&1 | \
  python3 -c "
import json, sys
pipelines = json.load(sys.stdin)
for p in pipelines:
    print(f'{p[\"id\"]:>6}  {p[\"status\"]:12s}  {p[\"ref\"]}')
" | cat
```

2. **Preview which pipelines would be canceled.** Keep the **first** (newest)
   pipeline; everything older that's `running`/`pending`/`created` is a
   cancel candidate. This step only *prints* — nothing is canceled yet:

```bash
glab api "projects/$PROJECT_ID/pipelines?ref=$BRANCH&sort=desc&per_page=10" 2>&1 | \
  python3 -c "
import json, sys
active = [p for p in json.load(sys.stdin) if p['status'] in ('running', 'pending', 'created')]
if len(active) <= 1:
    print('Nothing to cancel — at most one active pipeline.'); sys.exit(0)
print(f'Keeping newest: #{active[0][\"id\"]} ({active[0][\"status\"]})')
for p in active[1:]:
    print(f'Would cancel:   #{p[\"id\"]} ({p[\"status\"]})')
" | cat
```

3. **Execute the cancels.** Once the preview looks right, re-run the same query
   and pipe the emitted `glab api` commands straight into a shell — the IDs are
   filled in automatically, so there's nothing to copy-paste or substitute by
   hand. `sh -x` echoes each command as it runs:

```bash
glab api "projects/$PROJECT_ID/pipelines?ref=$BRANCH&sort=desc&per_page=10" 2>&1 | \
  python3 -c "
import json, sys
active = [p for p in json.load(sys.stdin) if p['status'] in ('running', 'pending', 'created')]
for p in active[1:]:
    print(f'glab api -X POST projects/$PROJECT_ID/pipelines/{p[\"id\"]}/cancel')
" | sh -x
```

> The preview (step 2) **is** the confirmation gate — eyeball it before running
> step 3. To re-add an explicit prompt, drop `| sh -x` and pipe through
> `while read c; do read -p \"run: $c ? \" a </dev/tty && [ \"\$a\" = y ] && eval \"\$c\"; done` instead.

4. **For multi-branch cleanup** (e.g., after pushing fixes to multiple MR
   branches), cancel superseded pipelines on each ref in one pass. Emit the
   `glab api` commands and pipe to `sh -x` (same eval-able pattern as step 3):

```bash
for BRANCH in branch1 branch2; do
  echo "=== $BRANCH ==="
  glab api "projects/$PROJECT_ID/pipelines?ref=$BRANCH&sort=desc&per_page=5" 2>&1 | \
    python3 -c "
import json, sys
active = [p for p in json.load(sys.stdin) if p['status'] in ('running', 'pending', 'created')]
for p in active[1:]:
    print(f'glab api -X POST projects/$PROJECT_ID/pipelines/{p[\"id\"]}/cancel')
" | sh -x
done
```

(Drop `| sh -x` → `| cat` to preview without canceling.)

## Notes

- **Project ID**: step 0 resolves it automatically from the repo name. If that
  fails (ambiguous name, no API access), look it up manually with
  `glab api "projects?search=<repo-name>" | python3 -c "..."`.
- **Don't cancel across branches** — only cancel older pipelines on the *same* ref.
- **Don't cancel `success` or `failed`** pipelines — they're already done.
- The `claude-review` job runs early and fast; the `check-package` job is usually
  what's hogging the runner. Canceling a superseded pipeline frees the runner for
  the newer one.
- Always pipe `glab api` through `| cat` to avoid pager issues.
