---
name: run-feature-review
description: INTERNAL worker for the run-feature orchestrator — do not invoke directly or auto-trigger. Performs the independent review of ONE named issue by following review-issue's rules unchanged, with loop-specific overrides - the agent running it IS the Opus reviewer (no nested subagent), the diff is bounded by git tree snapshots instead of the working tree, and it ends with a machine-readable VERDICT line. Use /review-issue for manual, one-off reviews.
---

# run-feature-review

Worker brief for one review step of `/run-feature`. You were launched by the orchestrator as an
**Opus subagent** with an issue ID and two git tree SHAs. This skill does **not** restate how to
review — `review-issue` stays the single source of truth. Read it and apply it with the overrides
below.

## Do this

1. Read `../review-issue/SKILL.md` (relative to this file's folder: the `review-issue` skill next to this one) in full.
2. Follow it as written, **except** for the overrides in the next section.
3. End with the result block described at the bottom.

## Overrides to review-issue

- **You are the reviewer.** `review-issue` tells its caller to spawn an Opus subagent in *Running
  the review*. Don't — you already are that subagent, launched fresh and independent of whoever
  implemented the code. Do the review yourself, using the briefing and scope from that section
  (conformance primary, fresh bug pass secondary, style and security out of scope).
- **The issue is given, not selected.** Review the issue ID the orchestrator named. Skip *Selecting
  an issue* but still confirm its `Status` in `issues/INDEX.md` contains `pending-review`; if not,
  stop with `VERDICT: error` and say why.
- **The diff is bounded by snapshots.** The orchestrator passes `snapshot_before` and
  `snapshot_after` (also in the issue's frontmatter) — git tree SHAs captured around the
  implementation. Use exactly:

  ```bash
  git diff <snapshot_before> <snapshot_after> -- . ':(exclude)issues'
  ```

  Do **not** use the plain working-tree `git diff`: nothing is committed in this pipeline, so it
  would include earlier issues' changes. `issues/` bookkeeping is excluded on purpose. If a
  snapshot is missing or `git diff` fails, stop with `VERDICT: error`. Files whose current content
  you need for context may be read from the working tree, but your findings concern the diff.
- **On a fail**, follow *On the verdict → Fail* as written (set `in-progress`, append concrete
  feedback to the issue file, update `INDEX.md` and drop the `⚠️ pending-review` marker), with two
  precisions:
  - Put the feedback under a heading `## Review feedback (review N)` where N = the issue's
    `review_attempts` frontmatter value + 1 (treat a missing field as 0). Earlier feedback sections
    stay.
  - Don't touch `rework`, `review_attempts` or the snapshot fields — the orchestrator owns them.
- **On a pass**, follow *On the verdict → Pass* as written, plus one precision: `implement-issue`
  writes `done ⚠️ pending-review` in **both** `INDEX.md` and the issue file's `status:` field, but
  `review-issue` only mentions clearing the index row. Also set the issue file's frontmatter to
  plain `status: done` (marker removed, no emoji) so the two stay consistent.
- **Never fix anything** and never re-invoke implementation. Nothing is committed.

## Result block

Your final message must end with exactly this block (nothing after it):

```
ISSUE: <issue id>
VERDICT: <pass | fail | error>
SECURITY: <yes | no>
SUMMARY: <≤8 lines: acceptance criteria verdicts in brief, genuine findings with file:line, nitpicks mentioned separately>
```

`SECURITY: yes` if any finding (blocking or not) concerns injection, authz, secret handling,
untrusted input or another security matter — even though security review isn't your focus, don't
withhold it if you noticed it. `error` means the review could not be carried out (no state
change was made except what you describe in SUMMARY).
