---
name: run-feature
description: Orchestrate a feature's whole backlog in one command instead of alternating implement-issue and review-issue by hand with a clear before each. Reads issues/INDEX.md each turn and runs the right step in a fresh subagent - open issue → implementation (run-feature-implement, which follows implement-issue), done ⚠️ pending-review → independent Opus review (run-feature-review, which follows review-issue) - strictly impl N → review N → impl N+1. A review fail triggers an automatic rework (max 2 per issue). Stops for the user on security findings, blocked or stuck issues, needs-review flags, a per-issue failure limit, an issue cap, or the end of the agent backlog, then sends a notification and a summary report. Never commits or pushes. Optional pause=M adds an M-minute delay between steps to spread quota use for a non-priority feature. Invoke explicitly with /run-feature [max=N] [pause=M]; not for one-off single-issue work (use /implement-issue or /review-issue). Requires issues/INDEX.md (from doc-to-issues).
---

# run-feature

Replaces the manual loop "clear → read `issues/INDEX.md` → launch `implement-issue` or
`review-issue`" with one command, run in the main session of the VS Code window of one feature
(one worktree / branch per feature; `INDEX.md` there only lists that feature's issues). Design
decisions and rejected alternatives: `SPEC.md` next to this file.

You are the **orchestrator**. You stay light: you read the index, launch fresh subagents, check what
they did, and print one-line progress. You never implement or review anything yourself, and you
never read source code of the project. Each step's context lives and dies inside its subagent —
that is what replaces the manual `clear`.

`Status` cells in `issues/INDEX.md` start with an emoji (`✅ done ⚠️ pending-review`, …). Match on
the **keyword**, ignore leading emojis (see "Status emojis" in `doc-to-issues`). `⚠️` markers
stack: `done ⚠️ placeholder ⚠️ pending-review` contains `pending-review`.

## Arguments

`/run-feature [max=N] [pause=M]` (any order).

- `max` caps how many **distinct issues get their first implementation** in this run (reworks
  don't count). Default **10**. Reaching the cap is a normal stop, not a failure.
- `pause` is a delay in **minutes** inserted **between two steps** (after a subagent returns and
  its result is verified, before the next subagent is launched), to spread quota use over time for
  a non-priority feature. Default: absent = **no delay** (priority feature, steps chain at once).
  Integer ≥ 1; reject anything else and stop before starting. No pause before the first step, and
  none after a STOP (go straight to "At the end").

## Never

- Never `git commit`, `git push`, switch branches, stash or reset. The working tree accumulates every
  issue's changes by design.
- Never fix or review code yourself, and never edit an issue's content (only the frontmatter
  fields listed under "Issue state" below).
- Never ask the user mid-loop unless a stop condition fires — a stop *is* the question.

## Issue state (frontmatter fields you own)

Issue files (`issues/<feature-slug>/<id>*.md`) have YAML frontmatter. You add and maintain four
fields in it (a missing field means its default). Other skills never write the first three;
`rework` is also cleared by the implementation worker at the end of a rework.

| Field | Default | Meaning |
|---|---|---|
| `snapshot_before` | absent | git tree SHA of the working tree just before the issue's **first** implementation. Never overwritten on rework |
| `snapshot_after` | absent | git tree SHA just after the **latest** implementation |
| `review_attempts` | 0 | number of `fail` verdicts received from review |
| `rework` | false | `true` from a review fail until the rework ends. Marks "reopened with feedback", as opposed to "interrupted" |

Find an issue's file with `ls issues/*/<id>*.md` (IDs look like `auth-003`). Edit frontmatter with
the Edit tool (keep every other field and the body untouched).

## Snapshots

A snapshot is a git **tree object** of the whole working tree (tracked and untracked files,
`.gitignore` respected), made through a temporary index so the real index, branch and history never
move. Define and use this exact helper in Bash (works in a repo with no commit yet):

```bash
snap() { tmp=$(mktemp -u); GIT_INDEX_FILE=$tmp git read-tree HEAD 2>/dev/null || true; GIT_INDEX_FILE=$tmp git add -A; GIT_INDEX_FILE=$tmp git write-tree; rm -f "$tmp"; }
snap   # prints a 40-char tree SHA
```

If it fails or prints nothing, stop (see "Stop conditions") — the review step depends on it. The
review diff is `git diff <before> <after> -- . ':(exclude)issues'`; `issues/` bookkeeping is
excluded on purpose.

## Setup

1. Check `issues/INDEX.md` exists and has rows; if not, tell the user to run `doc-to-issues`
   first and stop.
2. Confirm you're in a git repository (`git rev-parse --is-inside-work-tree`); if not, stop and say
   snapshots need git.
3. Parse `max` and `pause`, set counters: `implemented = 0`, plus a log of every step for the final
   report. If `pause` is set, say so in one line at the start (e.g. `⏳ mode temporisé : 20 min
   entre chaque étape`).
4. Load the notification tool once, now, so it is ready at the end:
   `ToolSearch` with query `select:PushNotification` (if unavailable, skip notifications silently).

## The loop

Each turn: re-read `issues/INDEX.md` (it is the source of truth, not your memory), then apply the
**first** matching rule:

1. Any row `done` with `needs-review` → **STOP** `needs-review`.
2. Any row `in-progress` whose issue file has `rework: true` → **IMPLEMENT (rework)** that issue.
3. Any other row `in-progress` → **STOP** `stuck` (interrupted work: the user decides to resume it
   or reset it to `open`).
4. First row with `pending-review` in its status → **REVIEW** that issue.
5. First row `open`, `owner` = `agent` or `placeholder`, every `depends_on` ID `done` →
   if `implemented >= max` → **STOP** `cap`; else **IMPLEMENT** that issue (and `implemented += 1`).
6. Otherwise → **STOP** `backlog-end` (what remains is `owner: user`, blocked, or `stale`).

Rule order enforces the strict alternation: a just-implemented issue is `pending-review`, so rule 4
fires before rule 5 can start the next one.

### PAUSE (only when `pause` is set)

Between two steps — i.e. when a step ended with "next turn" — wait before the next turn:

1. Print one line: `⏸ pause <M> min (reprise vers <HH:MM>)`.
2. Start the wait with Bash, **in the background** (foreground `sleep` is blocked):
   `Bash({ command: "sleep <M*60>", run_in_background: true, description: "Pause entre deux étapes" })`.
3. End your turn right there, with no other text. The harness wakes you when the command exits;
   then re-read `INDEX.md` and go on with the loop as usual (the files are the source of truth).

If the user writes during a pause, answer them; if they say to stop, run "At the end" with code
`user-stop`; if they say to skip the wait, stop the background command (`TaskStop`) and continue.
The subagents themselves are never slowed down: the delay only sits between them.

### IMPLEMENT

1. Fresh issue (not rework): run `snap`, write the SHA to the issue's `snapshot_before` — **only if
   the field is absent**. Rework: leave `snapshot_before` as is.
2. Launch **one** subagent, foreground, no `isolation`, no `model` (inherits the session's):

   ```
   Agent({
     description: "Implement <id>",
     run_in_background: false,
     prompt: "You are the implementation worker of a run-feature loop. Read and follow
       ~/.claude/skills/dev/skills/run-feature-implement/SKILL.md (it points you to implement-issue).
       Issue to implement: <id><, in REWORK mode — rework: true, review feedback is in the issue file>.
       Working directory: <absolute project path>."
   })
   ```
3. After it returns: run `snap`, write the SHA to `snapshot_after`.
4. Parse the `RESULT:` line of its final block, then **verify against the files**: re-read
   `INDEX.md` and the issue's frontmatter.
   - `RESULT: done` and the row now says `done` with `pending-review` and (if it was a rework)
     `rework` is `false` → print the progress line, next turn.
   - Row/frontmatter disagree with the result (e.g. says done but row isn't), or no parsable
     `RESULT` → **STOP** `inconsistent`.
   - `blocked` → **STOP** `blocked`. `security-finding` → **STOP** `security`. `needs-user` →
     **STOP** `needs-user`.

### REVIEW

1. Read `snapshot_before` and `snapshot_after` from the issue's frontmatter. If either is missing
   (issue implemented before this run, outside the loop) → **STOP** `no-snapshot`, telling the user
   to run `/review-issue <id>` manually for that one or reset it.
2. Launch **one** subagent, foreground, **`model: "opus"`**, no `isolation`:

   ```
   Agent({
     description: "Review <id>",
     model: "opus",
     run_in_background: false,
     prompt: "You are the independent reviewer of a run-feature loop. Read and follow
       ~/.claude/skills/dev/skills/run-feature-review/SKILL.md (it points you to review-issue).
       Issue to review: <id>. snapshot_before=<sha> snapshot_after=<sha>.
       Working directory: <absolute project path>."
   })
   ```
3. Parse `VERDICT:` and `SECURITY:`, then verify against `INDEX.md`:
   - **pass**: the row must now be `done` without `pending-review` → progress line, next turn.
     (If the row still says `pending-review` → **STOP** `inconsistent`.)
   - **fail**: the row must now be `in-progress`, with a new `## Review feedback (review N)` section
     in the issue file (else **STOP** `inconsistent`). Then:
     - increment `review_attempts` in the frontmatter;
     - `SECURITY: yes` → **STOP** `security` (leave `rework` unset: the user decides what to do);
     - `review_attempts >= 3` (this was the 3rd fail, i.e. 2 reworks already used) → **STOP**
       `retries-exhausted`;
     - otherwise set `rework: true` and continue (next turn hits rule 2).
   - **error** or unparsable → **STOP** `inconsistent`.

## Progress output

One line per step, in the user's language (French by default for this user), e.g.:

```
✅ impl auth-003 → done ⚠️ pending-review (4 fichiers)
🔎 review auth-003 → pass
🔎 review auth-004 → fail (rework 1/2) — critère 3 non couvert
🔨 impl auth-004 (rework 1/2) → done ⚠️ pending-review
```

Add a short reason after a fail. Don't paste agent summaries in full while looping; keep them for
the final report.

## Stop conditions

On any STOP: print a one-line reason, then run "At the end". Reasons and what the user must do:

| Code | Meaning | Ask the user to |
|---|---|---|
| `security` | an unresolved security finding (implementation or review) | read the finding in the issue Notes / feedback and decide |
| `blocked` | worker left the issue `in-progress` with a blocker note (false assumption, missing prerequisite, tests/build failing) | read the Notes and unblock or reshape the issue |
| `stuck` | `in-progress` issue with no `rework` marker (interrupted session) | resume it or reset it to `open` |
| `needs-review` | `done ⚠️ needs-review` (security spec change) | have it addressed with `/implement-issue` |
| `retries-exhausted` | 3rd review fail on one issue | read the review feedback sections and decide |
| `needs-user` / `inconsistent` / `no-snapshot` | worker needs a decision, or index/frontmatter disagree with a worker's report, or snapshots missing | inspect the issue; say precisely what disagreed |
| `cap` | `max` distinct issues implemented | relaunch `/run-feature` to continue |
| `backlog-end` | normal end of the agent backlog | review the waiting-on-you list, run `/qc` on the feature when ready |
| `user-stop` | the user asked to stop during a pause | relaunch `/run-feature` (with or without `pause`) to continue |

Failing tests or build after an implementation are not a separate stop: the implementation worker
leaves the issue `in-progress` with a blocker note, which surfaces as `blocked`. Pre-existing
failures are only reported.

## At the end

1. **Final report in chat**, in the user's language, short:
   - why the loop stopped (code + one sentence) and, if it needs the user, exactly what to do next;
   - issues implemented / reviewed (pass), reworks with their cause, count of issues still `open`;
   - from `INDEX.md`: open `owner: user` issues and what each waits on, `done ⚠️ placeholder` issues,
     remaining `pending-review` issues;
   - pre-existing failures or inline-fallbacks that workers mentioned in their summaries.
2. **Notification** (if `PushNotification` loaded): one phrase — feature/branch name, stop code, and
   whether the user's action is needed. It's a heads-up for someone working in another window.
3. Do not chain `/report`, `/qc` or `documentation` — `qc` is the user's manual test; `documentation`
   was already run by each implementation.
