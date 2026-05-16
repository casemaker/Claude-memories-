```
$ cat feedback_reviewer_comments_to_pr.md
---
name: Post reviewer summaries to the PR
description: When running the two-reviewer (or three-reviewer) protocol, post the consolidated findings as a comment on the PR itself — don't keep them in conversation only
type: feedback
---
When running the two-reviewer (or three-reviewer) protocol on a PR, post the consolidated reviewer findings + which NITs/BLOCKERs were addressed as a `gh pr comment` on the PR itself. Do this as part of the standard protocol flow, before requesting merge.

**Why:** the user noticed I was running multi-reviewer rounds and applying fixes but the PRs themselves had no record of what was reviewed or addressed — just the original `gh pr create` description. The reviewer findings live in conversation, which the user can't easily reference after the conversation ends. The PR comment is the durable record for future-self / external readers.

**How to apply:**
- After all reviewer rounds complete and review-fix commits are pushed, post a single consolidated comment summarizing each reviewer's BLOCKERs/NITs and whether each was addressed/deferred.
- Use markdown headings per reviewer (Reviewer A — correctness, Reviewer B — design, etc.).
- Backend and frontend PRs each get their own summary; the frontend can reference the backend PR by number for shared findings.
- Do this BEFORE asking the user to merge, so the PR is fully self-documenting.

If the conversation has already moved past the protocol without comments being posted, offer to backfill on request — the comment is still useful even after merge.
```

```
$ cat feedback_stacked_pr_base_branch.md
---
name: feedback-stacked-pr-base-branch
description: "Stacked PRs MUST target main (not the parent's branch) — squash-merging the parent with --delete-branch auto-closes the child permanently"
metadata:
  node_type: memory
  type: feedback
---

When opening a child PR that builds on an unmerged parent PR, set the
PR's base to `main` — NEVER to the parent's branch.

**Why:** Hit this on PR173 → PR174. PR174 was cut with
`gh pr create --base fix/broker-state-spx-price ...`. After
squash-merging PR173 with `--delete-branch` (the standard pattern per
[[feedback-pr-workflow]]), PR174 auto-closed because GitHub closes any
PR whose base branch was deleted. Worse: once closed with a missing
base, GitHub **refuses to reopen** (`gh pr reopen` returns "Could not
open the pull request"). The only fix is to recreate as a new PR,
losing the original number + any review threads + comments. Recovered
as PR175 in that case, but the original review history on #174 was
unrecoverable.

**How to apply:**

1. When cutting a stacked PR, always use `gh pr create --base main ...`
   even if the local branch is built on top of a parent branch.
2. Document the stacking dependency in the PR description body
   ("This PR depends on #173 — merge that first, then I'll rebase").
3. After the parent merges, rebase the child to drop the now-redundant
   commits cleanly: `git rebase --onto main <parent-tip-sha> <branch>`,
   then `git push --force-with-lease`. GitHub recomputes the diff
   automatically.
4. Don't bother with `gh pr edit --base main` to retarget AFTER the
   parent's branch is deleted — GitHub rejects base changes on a
   closed PR with HTTP 422.

**Alternative escape hatch** (if you forget rule 1): retarget the
child's base to `main` *before* the parent is merged, while both
branches still exist. `gh pr edit <child> --base main` works as
long as the current base branch hasn't been deleted yet.
```

```
$ cat feedback_pr_workflow.md
---
name: Always use PR workflow
description: Never push directly to main — always create a branch, PR, and run QA review before merging
type: feedback
---

Never push commits directly to main. Always:
1. Create a feature/fix branch
2. Push to the branch
3. Create a PR with description
4. Run QA review agent
5. Fix any blocking issues
6. Post QA review comment to the PR
7. Merge via `gh pr merge`

**Why:** User explicitly asked for proper audit trail. When a direct-to-main push happened, user requested it be reverted and re-pushed through a PR.

**How to apply:** Every code change, no matter how small, goes through this workflow.
```

```
$ cat feedback_invasive_pr_review_rounds.md
---
name: Invasive-PR review rounds
description: For highly invasive PRs (schema change + cross-repo + many touched files), run the two-reviewer protocol with a minimum of 3 rounds per PR.
type: feedback
---
When a PR is "highly invasive" (schema/type changes that cascade across many files, cross-repo changes that must land together, or large-scope refactors), apply the two-reviewer protocol with **at least 3 rounds per PR per reviewer**:

- R1 (correctness): ≥3 rounds across the changes — value pins, schema consistency, no dead references surviving.
- R2 (UX/ops): ≥3 rounds across the changes — copy clarity, ordering, deploy safety, localStorage / persisted-state migration.

Pace iteratively: address each round's NITs before the next round begins.

**Why:** Confirmed after the 2026-05-10 Capital-tab refactor (Task #265). That PR replaced 4 Copeland-era allocation policies + dropped 2 schema fields + regenerated a parity fixture across two repos. The scope was deep enough that a single pass per reviewer would miss things — multiple rounds are needed to flush out drift between backend/frontend/tests.

**How to apply:** When the PR description or commit shows (a) cross-repo coupling, (b) schema/Literal/type changes, (c) 8+ touched files, OR the user explicitly flags the change as invasive — default to ≥3 rounds per reviewer. For routine bugfix or single-file PRs the standard one-round-per-reviewer cadence is still fine.
```
