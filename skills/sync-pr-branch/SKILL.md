---
name: sync-pr-branch
description: Bring a PR branch up to date with main by merging origin/main into it, resolving conflicts, running the repo's pre-commit checks, and pushing. Use before triggering a review or pushing fixes, when "update the branch", "merge main in", "the branch is behind main", or whenever main has moved ahead of a PR branch you're working on.
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
---

# sync-pr-branch

Keep a PR branch current with `main` so reviewers (human and the `@claude`
bot) evaluate it against today's `main`, not a stale snapshot. The standing
rule: whenever `main` has moved ahead, **merge it in** — don't wait for a
conflict to surface or for the user to ask.

## When this fires

- Before pushing fixes to a PR branch, or before triggering a fresh review.
- "update the branch", "merge main in", "sync with main", "the branch is
  behind main", "resolve the conflicts with main".
- Any time you notice `main` is ahead of the branch you're working on.

## The procedure

1. **Check whether main is ahead.**
   ```bash
   git fetch origin main
   git log --oneline ..origin/main | head   # any commits listed → main is ahead
   ```
   If nothing is listed, the branch is current — say so and stop.

2. **Merge, don't rebase.** A merge commit matches GitHub's "Update branch"
   button and preserves PR history. **Never** rebase or squash-rewrite a
   *published* PR branch unless the user explicitly asks.
   ```bash
   git merge origin/main
   ```

3. **Resolve any conflicts** in the working tree. Resolve them fully — don't
   push a half-resolved merge.

4. **Run the repo's pre-commit checks before committing the merge.** For this
   repo (rme) that's the mandatory checklist — render each affected parent
   chapter, lint changed R/`.qmd`, spellcheck:
   ```bash
   quarto render <chapter.qmd> --to html      # each parent chapter touched by the merge
   Rscript -e 'lintr::lint("path/to/file")'   # each changed .R / .qmd
   Rscript -e 'spelling::spell_check_package()'
   ```
   (If the repo ships `quarto-preflight` / `render` / `lint` / `spell` skills,
   use them.) Only proceed once they pass. In a different repo, run that
   repo's equivalent checks.

5. **Commit the merge resolution** (if there were conflicts; a clean merge
   auto-commits) and **push**:
   ```bash
   git push
   ```

## Notes

- If the merge is clean (no conflicts, no content change to validate), you can
  push straight away — but still confirm the render is unaffected if subfiles
  changed underneath you.
- This skill is the front half of an `iterate` round (step 2, "sync with
  main"). When iterating, run it before each review trigger.
- Don't merge **other open PRs'** branches into the one in progress — only
  `origin/main`. Cross-PR changes belong in their own branch.
