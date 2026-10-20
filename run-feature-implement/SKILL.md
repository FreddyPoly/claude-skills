---
name: run-feature-implement
description: INTERNAL worker for the run-feature orchestrator — do not invoke directly or auto-trigger. Implements ONE named issue by following implement-issue's rules unchanged, with loop-specific overrides - already running as a subagent, works on the issue it is given instead of selecting one, resumes reopened issues (rework true) without asking, ends with a machine-readable RESULT line. Use /implement-issue for manual, one-off work.
---

# run-feature-implement

Worker brief for one implementation step of `/run-feature`. You were launched as a subagent by the
orchestrator with an issue ID. This skill does **not** restate how to implement — that is
`implement-issue`'s job, and it stays the single source of truth. Read it and apply it with the
overrides below.

## Do this

1. Read `~/.claude/skills/implement-issue/SKILL.md` in full.
2. Follow it exactly as written, **except** for the overrides in the next section.
3. End with the result block described at the bottom.

## Overrides to implement-issue

- **You are already the subagent.** Skip the section *Running this as a subagent* — don't launch
  another `Agent` for the whole job. Follow everything from *Everything from here on* onward.
- **The issue is given, not selected.** Work on the issue ID the orchestrator named. Skip *Checking
  for stuck work first* and *Selecting an issue* (the orchestrator already did both). Keep the
  `depends_on` check from the user-named-issue paragraph: if a dependency isn't `done`, stop with
  `RESULT: blocked`.
- **Rework mode.** If the issue's frontmatter has `rework: true`, it was reopened by an independent
  review: its status is already `in-progress` and a `## Review feedback` section was appended to the
  issue file. That is *not* stuck work — do not ask the user how to proceed. Read the feedback,
  address every point, re-run the acceptance criteria, tests and code-review (plus security-review
  if `security: true`) as usual, then mark the issue `done ⚠️ pending-review` again. Finally set
  `rework: false` in the frontmatter. The issue's earlier work is already in the working tree —
  fix it, don't redo it from scratch.
- **Orchestrator-owned frontmatter.** `snapshot_before`, `snapshot_after`, `review_attempts` are
  written by the orchestrator. Never edit them. `rework` is the one field you may change (to
  `false`, at the end of a rework). When you re-run `doc-to-issues` at the end, these four fields are
  runtime state — make sure they survive untouched on every issue file.
- **No waiting-on-you report.** Skip the final *Waiting-on-you report*; the orchestrator produces
  one for the whole run.
- **If you cannot spawn subagents** (the `Explore` agent, or agents that `code-review` /
  `security-review` would launch), do the equivalent work yourself, inline, with ranged reads and
  greps. Never skip a review gate because a subagent was unavailable. Mention it in your summary.
- **Nothing is committed or pushed**, ever. The working tree accumulates every issue's changes;
  that is expected.
- **A finding you can't resolve with confidence stops you.** Per implement-issue: significant,
  ambiguous or security-related findings are surfaced, not patched on a guess. In loop mode that
  means: leave the issue `in-progress` with the finding written in its Notes, and report it.

## Result block

Your final message must end with exactly this block (nothing after it):

```
ISSUE: <issue id>
RESULT: <done | blocked | security-finding | needs-user>
SUMMARY: <≤8 lines: what changed, files touched, review findings fixed, pre-existing test failures if any, what doc/backlog sync surfaced>
```

- `done` — issue marked `done ⚠️ pending-review` (or `done ⚠️ placeholder ⚠️ pending-review`) in the
  issue file and in `issues/INDEX.md`, all gates passed.
- `blocked` — left `in-progress` with a Notes entry explaining the blocker (false assumption,
  missing prerequisite, failing tests that you could not fix, unmet dependency).
- `security-finding` — a security-review (or code-review) finding is unresolved and needs the
  user's decision; issue left `in-progress`, finding described in Notes and SUMMARY.
- `needs-user` — any other ambiguity only the user can decide.
