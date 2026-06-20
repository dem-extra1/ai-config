---
name: use-preferred-style
description: "Write or revise user-facing prose in the user's preferred style — limit dependent (subordinate) clauses, cut low-content filler and jargon, prefer short declarative sentences, and join ideas with coordinating conjunctions (and/but/so/or) over subordinate constructions. Apply when drafting or rewriting any prose: PR/issue/commit text, docs, READMEs, comments, release notes, emails, or chat replies. Use when asked to 'use my style', 'apply my preferred style', 'rewrite in my voice', 'tighten this', 'plain-language this', or '/style'."
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
---

# use-preferred-style — write the way the user prefers

Apply the user's prose style to anything user-facing. The style is plain and
direct. Say the point in a short sentence. Stack fewer clauses. Drop filler.

## When this fires

- The user asks to "use my style", "apply my preferred style", "rewrite in my
  voice", "tighten this", "plain-language this", or invokes `/style`.
- You are drafting or revising any prose the user will read: PR/issue/commit
  bodies, docs, READMEs, code comments, release notes, emails, chat replies.
- Apply it by default to your own drafts, even when not asked. This is a
  standing preference, not just an on-demand command.

## The four rules

1. **Limit dependent clauses.** A dependent (subordinate) clause cannot stand
   alone. It starts with a word like *because*, *although*, *which*, *while*,
   *since*, *when*, or *if*. One per sentence is plenty. Two or more stacked
   together is a rewrite signal. Break the sentence apart instead.

2. **Cut low-content filler and jargon.** Delete words that add no information.
   Replace jargon with the plain word. See the swap table below.

3. **Prefer simple declarative sentences.** State the fact. Put the subject
   first, then the verb, then the rest. Short beats clever.

4. **Join independent clauses with coordinating conjunctions.** Two complete
   thoughts read better side by side than nested. Use *and*, *but*, *so*, *or*,
   *yet*, *nor*, *for*. Prefer "X is fast, but Y is correct" over "While X is
   fast, Y is correct."

## Filler and jargon to cut or swap

| Cut / avoid | Use instead |
|-------------|-------------|
| in order to | to |
| due to the fact that | because |
| at this point in time | now |
| in the event that | if |
| has the ability to | can |
| utilize, leverage | use |
| facilitate | help, let |
| it is worth noting that | (delete) |
| it should be noted that | (delete) |
| needless to say | (delete) |
| as a matter of fact | (delete) |
| basically, essentially, actually | (delete) |
| very, really, quite, simply | (delete) |
| a number of | some, several, N |
| in terms of | (rephrase or delete) |
| with regard to | about |

Treat the table as a starting set, not a closed list. Any word that carries no
information is filler. Cut it.

## Procedure

1. **Read the target prose.** A file, a diff, a draft, or text the user pasted.
2. **Find the long sentences first.** A sentence over ~25 words usually hides a
   dependent clause you can split out. Split it.
3. **Strip filler.** Run the swap table over the text. Delete dead words.
4. **Flatten subordination.** Turn "Although A, B" into "A. But B." or
   "A, but B." Turn "which" relative clauses into a second sentence.
5. **Keep the meaning exact.** Style edits must not change facts, scope, hedges
   the user meant, or technical precision. When a hedge is load-bearing, keep
   it. Plainness is the goal, not false confidence.
6. **Read it back.** Each sentence should make one point. The data should flow
   top to bottom.

## Before / after

> **Before:** It is worth noting that, due to the fact that the cache was not
> being invalidated when the user updated their profile, which caused stale data
> to be served, we decided to utilize a new eviction strategy in order to
> resolve the issue.

> **After:** The cache was not invalidated on profile update. So it served stale
> data. We switched to a new eviction strategy to fix it.

The "after" is three sentences. Each makes one point. No filler, no `utilize`,
no stacked `which`/`due to the fact that` clauses.

## When to keep the nesting

This is a default, not an absolute rule. Keep a dependent clause when splitting
it would read worse or lose a real logical link. Keep a technical term when it
is the precise word and the plain swap would be wrong. Readability wins, not
minimalism for its own sake. (Mirrors the "avoid nesting, but not blindly"
stance in the coding-style rule.)

## Relationship to other skills

- **`find-ai-tells`** (issue #49, in progress) — the detector counterpart. It
  *scans* finished text for AI-authorship tells. This skill *prescribes* how to
  write up front. Run `find-ai-tells` after; run `use-preferred-style` during.
- **`simplify` / `tidy`** — the same "cut what adds no value" instinct, applied
  to code instead of prose.
- **`memorize` / `remember`** — for a one-off wording preference, write a memory
  instead of editing this skill.

## Anti-patterns

- ❌ Changing the meaning, scope, or technical precision while "tightening."
- ❌ Dropping a hedge the user meant to keep (turning honest uncertainty into
  false confidence).
- ❌ Chopping every sentence to staccato fragments — vary length; aim for clear,
  not robotic.
- ❌ Applying it to code identifiers or quoted material that must stay verbatim.
