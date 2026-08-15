---
name: implement-backlog
description: Fan out across a project's whole open issues/ backlog (or a named feature/issue subset) instead of implementing one issue at a time. Computes a dependency + file-conflict plan (waves of issues that can run in parallel, sequenced where they'd collide), shows it to the user for a go-ahead, then launches capped batches of subagents — one per issue, isolated in a git worktree when the project is a git repo — each following implement-issue's implementation rules and TDD-with-the-project's-own-test-framework convention, but never touching shared files (issues/INDEX.md, issues/FEATURES.md, SPEC.md) or running per-issue code-review/security-review themselves. The orchestrator merges each wave's branches locally, updates bookkeeping centrally, and once the whole run finishes, runs one consolidated adversarial code-review pass (via the code-review skill, focused using a generic baseline plus bug patterns mined from this project's own CLAUDE.md/KNOWN-ISSUES.md history) across the cumulative diff, plus security-review if any security:true issue was touched — reopening whatever it implicates. Never commits automatically. Use when the user wants to work through many backlog issues at once instead of one-by-one, or invokes /implement-backlog. Requires an issues/ folder (from doc-to-issues) — if none, use doc-to-issues first.
---

# implement-backlog

Implement many issues from `issues/` in one run instead of one at a time, using parallel subagents
where it's safe and sequencing where it isn't — then review the whole batch together, once, instead
of per issue. This is `implement-issue`'s conventions applied at backlog scale, not a different way
of building things.

## Checking for stuck work first

Same check as `implement-issue`, run once before planning anything new: scan `issues/INDEX.md` for
`status: in-progress` issues (an interrupted previous session — check for matching uncommitted work,
ask the user whether to resume or reset rather than guessing) and `done ⚠️ needs-review` issues
(surface them, ask whether to address one now). Don't start planning a new batch while either sits
unresolved.

## Selecting the scope

Default to every eligible issue across the whole backlog — same eligibility rule as
`implement-issue`: `status: open`, `owner` is `agent` or `placeholder`, every ID in `depends_on` is
`status: done`. `owner: user` issues are never picked up here either; they surface in the
waiting-on-you report at the end, same as `implement-issue`.

If the user names a feature or a specific list of issue IDs when invoking, scope the run to just
those (still applying the same eligibility rule within that subset) rather than the whole backlog.

## Building the execution plan

Read every issue in scope — file, description, acceptance criteria, `depends_on` — and ground
yourself in the actual codebase enough to judge which files each issue is likely to touch (read the
relevant modules, don't guess from the issue title alone).

Compute:

1. **Dependency waves.** An issue can't be scheduled until every issue in its `depends_on` has
   actually landed (see "Marking a wave done" below for what "landed" means here) — this can span
   multiple waves for a chain of dependencies.
2. **File-conflict split within a wave.** Two issues that would touch overlapping files can't run in
   the same parallel batch even without a formal `depends_on` between them — push one into a later
   sub-wave instead. When in doubt about overlap, sequence rather than risk a collision.
3. **Concurrency cap.** Default to at most ~4-6 subagents running at once. If a wave has more
   eligible, non-conflicting issues than the cap, split it into capped sub-batches rather than
   launching all of them together. Let the user override the cap when they invoke this skill.

Present the resulting plan before touching anything — which issues run in which wave, which run in
parallel vs. get sequenced and why (dependency vs. file conflict) — and get a go-ahead from the user
before launching any subagent. This is a multi-agent, potentially costly operation; a silent launch
isn't appropriate here even though single-issue `implement-issue` runs don't need this pause.

## Isolation

Check whether the project is a git repository.

- **Git repo:** launch each subagent in its own isolated worktree (the `Agent` tool's
  `isolation: "worktree"` option) — this holds even if the file-conflict analysis above missed
  something, since each subagent's changes land on their own branch rather than the shared working
  tree.
- **No git repo:** fall back to shared-working-directory parallel execution for that wave, relying
  on the file-conflict split to keep parallel subagents on disjoint files.

## Per-issue subagent contract

Launch one subagent per issue in the current parallel batch (a single message with multiple `Agent`
tool calls when running more than one in parallel — sequential calls only for issues deliberately
sequenced). Each subagent's prompt is self-contained (it has no memory of this conversation) and
includes:

- The full issue file content: description, acceptance criteria, notes, `depends_on`, `security`
  flag.
- A pointer to `CLAUDE.md`/`SPEC.md` for grounding in project conventions.
- `implement-issue`'s implementation rules verbatim: existing conventions win over generic best
  practice, no premature abstraction or scope creep beyond this one issue's acceptance criteria,
  keep changes scoped to what the issue actually requires, comments only where the *why* is
  non-obvious, TDD (failing test first) when the project already has a test suite/framework in
  place — using whatever that framework actually is, never bolting a new one on for a single issue
  (flag that gap in the issue's Notes instead).
- `implement-issue`'s placeholder handling, if the issue is `owner: placeholder`: build it for real
  with a clearly-fake default, note what's placeholder.
- Explicit exclusions: **do not** edit `issues/INDEX.md`, `issues/FEATURES.md`, or `SPEC.md` —
  instead, report back what changed and what `SPEC.md` should say, in prose, for the orchestrator to
  apply centrally. **Do not** run `/code-review`, `/security-review`, `/documentation`, or
  `doc-to-issues` — those happen once, centrally, after the whole run (see below).
- The rest of `implement-issue`'s definition-of-done still applies: every acceptance criterion
  actually met, existing tests/build/lint passing (or no *new* failures beyond a pre-existing
  baseline), and if something turns out blocked or wrong in a way `depends_on` didn't capture, don't
  force it — report what's blocking it instead of guessing to a false done.
- Ask for a structured report back: what was implemented, files touched, test results, anything
  uncertain or blocking, and the suggested `SPEC.md` delta.

## After each wave

1. Merge that wave's worktree branches into the working branch locally (not pushed) — later waves
   need this wave's code actually present to build on, even for git projects.
2. Apply bookkeeping centrally, once, for the whole wave: mark issues that came back clean
   `status: done` in `issues/INDEX.md` *optimistically* (the consolidated review at the end of the
   whole run can still reopen one), leave failed/blocked ones `in-progress` with the subagent's
   notes appended. Fold each subagent's suggested `SPEC.md` delta into one coherent edit rather than
   applying them as separate, potentially colliding writes.
3. Recompute which issues are now unblocked (their `depends_on` satisfied by this wave) and continue
   to the next wave per the plan, re-checking the file-conflict split against anything newly
   eligible.

## After the whole run finishes

Once every wave in the plan has completed, run the consolidated review across the run's *cumulative*
diff — not per wave, not per issue:

1. Spawn a subagent that invokes the existing `code-review` skill at a high/max effort level against
   the merged diff. Give it a focus list: a small generic baseline (null/exception handling, input
   validation, resource-scope assumptions that don't hold across contexts, off-by-one, concurrency
   hazards from the parallel work itself) plus anything mined from this project's own
   `CLAUDE.md`/`KNOWN-ISSUES.md`/closed-issue notes describing bug patterns that have actually hit
   this project before. Don't invent project-specific patterns that aren't actually documented
   anywhere — the mined half of the list should come from real history, not guesses.
2. If any issue in the run was `security: true`, also run `security-review` once against the same
   merged diff, in addition to code-review — same as `implement-issue` does per issue, just
   consolidated across the batch instead of repeated per issue.
3. Handle findings the same way `implement-issue` does: fix clear, small findings directly yourself
   and note it in the final summary. For anything significant, ambiguous, or that would change an
   issue's approach — and for anything security-review flags, unconditionally — don't silently fix
   it. Reopen the issue(s) it implicates (`status: in-progress`, feedback appended, even if that
   issue was already marked `done` in step 2 above) and surface it to the user instead of deciding
   unilaterally.
4. Run `documentation` once to record what changed across the whole batch, then re-run
   `doc-to-issues` once to resync the backlog.

Never commit automatically at any point in this flow — matching the existing rule that nothing gets
committed without the user explicitly asking. Everything from every wave ends up merged into the
working branch but uncommitted, for the user to review and commit on their own terms.

## Final report

Report, once, for the whole run:

- The plan that was executed — which issues ran in which wave, what ran in parallel vs. was
  sequenced and why (dependency vs. file conflict).
- Which issues shipped clean.
- Which were reopened by the consolidated review, and why.
- Any that stayed blocked mid-implementation, with what's blocking them.
- The same waiting-on-you report `implement-issue` ends with: open `owner: user` issues and what
  each is waiting on, plus `done ⚠️ placeholder` issues and what they're standing in for. Skip either
  list, or the whole report, if there's nothing to include.
