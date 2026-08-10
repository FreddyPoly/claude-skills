---
name: qc
description: Guide the user through manually testing a complete feature (issues under issues/<feature-slug>/, all owner:agent/placeholder issues status:done) against SPEC.md, so the user — not the agent — confirms it works. Derives a test script from the spec and the feature's issues (adding security scenarios when flagged security: true, excluding placeholder details like colors/copy from pass/fail judgment, noting gaps from any open owner:user blocker), drives the app to each scenario (via the run skill), and records verdicts on issues/FEATURES.md. Recommends a full security-review pass once every feature has passed, plus a summary of remaining open issues/placeholders. On failure, investigates which issue(s) are responsible, writes feedback into them, and reopens them for implement-issue. Use to test/verify a finished feature, or invoke /qc. Requires SPEC.md and a done feature under issues/.
---

# qc

Confirm that a complete feature works, with the user actually trying it — not the agent
self-certifying. The agent's job is to figure out what to test and set it up; judging whether it
works is the user's call.

## Selecting a feature

Read `issues/FEATURES.md`. Recompute each feature's readiness live before doing anything else —
don't trust a stale `Status` column: a feature is `ready-for-qc` once every `owner: agent` and
`owner: placeholder` issue under `issues/<feature-slug>/` has `status: done`. An open `owner: user`
issue doesn't block readiness — note it as a caveat instead (see below) rather than waiting on it
indefinitely; a pending business decision shouldn't stall testing everything else in the feature.

- If the user names a feature, test that one — but still tell them plainly if it's not actually
  ready (some issues not done yet) instead of testing it partially.
- If they don't name one, pick the first feature that's ready-for-qc and hasn't been tested since
  its issues last changed (i.e. not already `qc-passed` with nothing reopened since). If none are
  ready, say what's still outstanding instead of testing something incomplete.

## Building the test script

Don't test issue-by-issue. Read:

- The feature's section in `SPEC.md` — what the feature is supposed to do as a whole, including
  any use cases or edge cases captured there.
- Every issue under `issues/<feature-slug>/` — their acceptance criteria and notes, combined.

From these, write a short scenario list covering the feature end-to-end, including how its pieces
interact (this is the part per-issue testing structurally can't catch — e.g. does the login flow
issue actually compose correctly with the session-handling issue). Order scenarios the way a user
would naturally encounter them, not the order the issues happen to be numbered.

If any issue under the feature is `security: true`, add adversarial scenarios drawn from SPEC.md's
Security section on top of the normal functional ones — e.g. try the flow unauthenticated, try
accessing another user's data, submit malformed/oversized/unexpected input, try an action the spec
says should be forbidden for this role. These aren't edge cases of "does it work," they're "does it
resist misuse" — call them out to the user as security scenarios specifically, so a "looks fine" on
the happy path doesn't get mistaken for a security pass.

Check the feature's issues for two more things before finalizing the script:

- **`owner: placeholder` issues** — note which scenarios touch a flagged placeholder (a color,
  copy, an asset) and exclude those specifics from pass/fail judgment. Don't ask the user to verify
  a placeholder color is "correct" — it was never meant to be final. If a placeholder-covered
  scenario is otherwise broken (the button doesn't work at all, not just "the color isn't final"),
  that's still a real failure — the placeholder exemption only covers the placeholder itself.
- **Open `owner: user` issues** — anything the feature genuinely can't be tested without (e.g. a
  real payment flow needing a real API key) becomes a noted gap, not a scenario: tell the user what
  can't be verified yet and why, rather than skipping it silently or forcing a pass/fail on
  something that structurally can't be checked.

## Running the walkthrough

Use the `run` skill to launch the project. For web/UI features, drive the shared browser pane
yourself to each scenario — navigate to the right screen, set up any needed state — then tell the
user what to look at or try, and wait for them to confirm what they observe rather than deciding
for them. For CLI/API/non-UI features, run the relevant command/request yourself, show the actual
output, and ask the user to confirm it's what they expect.

For a feature with no interactive surface at all (a library, an internal module, a batch job) —
there's still something to exercise: call the changed function/entry point directly yourself
(a throwaway script or REPL invocation is fine) and show the user the real input/output, rather
than asking them to read code or trust that passing unit tests means the feature works.

Go one scenario at a time. Don't dump the whole script and ask for one blanket "does it all work?"
— that's what leads to missed edge cases. Ask for a verdict per scenario, and let a "no" or "sort
of" prompt follow-up questions about specifically what was wrong before moving on.

## On a full pass

Update `issues/FEATURES.md`: set the feature's `Status` to `qc-passed` and `Last QC` to today's
date.

Then check whether this was the last remaining feature — if every row in `FEATURES.md` is now
`qc-passed`, tell the user the whole project has cleared per-feature QC, and recommend running a
full-project `security-review` (across the whole codebase, not just this feature's diff) before
considering the project release-ready. This is a milestone check for things no single issue or
feature's review would catch on its own — dependency vulnerabilities, misconfigured infra, and
other cross-cutting concerns. Recommend it; don't run it unprompted, since it's a deliberate,
potentially time-consuming pass over the entire codebase rather than a quick per-issue check.

`qc-passed` doesn't mean gap-free, though — a feature can pass with an open `owner: user` issue
still noted as an untested gap, or with `done ⚠️ placeholder` issues inside it. So alongside the
security-review recommendation, scan the whole project (`issues/INDEX.md`) and summarize any
remaining open `owner: user` issues and `⚠️ placeholder` markers across every feature, not just
this one. "Every feature qc-passed" and "actually finished" aren't the same thing here — say so
plainly rather than letting the milestone message read as an all-clear it isn't.

## On failure

For each scenario the user flagged:

- Investigate which issue(s) are responsible, starting with the current feature's own issues —
  that's usually right. If the problem clearly doesn't map to anything in this feature (e.g. the
  root cause is in a shared module or a different feature's work this one depends on), widen the
  search to the rest of `issues/` before asking the user to help narrow it down. Don't force an
  attribution to this feature just because it's the one being tested.
- If it's genuinely ambiguous between two or more issues (in this feature or across features), ask
  the user to help narrow it down rather than guessing.
- Set that issue's `status: in-progress` and append concrete feedback to it (what scenario was
  tested, what was expected per the feature/spec, what actually happened, any detail the user
  gave) — enough that `implement-issue` can pick it back up without re-asking the user what went
  wrong.
- Update `issues/INDEX.md` to reflect the reopened status.

Then set the feature's `Status` in `FEATURES.md` to `qc-failed` and `Last QC` to today's date.
Report a summary: which scenarios failed, which issues were reopened and why. Don't automatically
re-invoke `implement-issue` to fix things — reopening is as far as this skill goes; fixing is a
separate, deliberate next step.
