---
name: polish
description: Refine one already-built, already-qc-passed aspect of a project at a time — visual/UX details, layout, positioning, fonts, icons, animation timing, interaction feel, swapping in a finished art asset, or kicking off production of a not-yet-made one (pinning down format/dimensions/file paths/generation pipeline, then handing off to doc-to-issues for a per-variant owner:user + owner:agent issue split) — through the same relentless, multi-round interview style as the `interview` skill, but scoped to a single named aspect instead of a whole new feature. Always launches the running app first (via the `run` skill and any debug/fast-forward tools the project has) and looks at its live current state before asking anything, since visual/UX/feel decisions can't be judged from code alone. Stops at a numbered "Shared Understanding" summary and waits for explicit approval before writing anything. Once approved, updates the relevant existing SPEC.md section in place with the validated decision — no new section, no separate changelog file — then stops and points to `doc-to-issues` to sync the concrete issue; its existing rerun logic already reopens the affected done issue and flips that feature back to not-ready, so this skill never touches issues/INDEX.md or issues/FEATURES.md itself. Refuses a target that isn't an already-built, already-qc-passed aspect and redirects to `interview` instead — this skill is for refining what exists, not designing something new, and stays out of raw numeric/balance tuning (costs, growth rates, thresholds), which belongs to a separate deferred pass. Use whenever the user wants to polish, refine, nail down visual/UX/asset details for, or tune the feel of a specific already-built part of a project — "make the kill counter feel right," "let's polish the login button," "the queueing feels weird, can we fix that" — or invokes /polish explicitly. Requires an existing SPEC.md and issues/ (from doc-to-issues) — if neither exists, point to doc-to-issues (or interview first) instead.
---

# polish

`qc` answers "does this work." This skill answers "is this good" — one already-built aspect at a
time, with the same rigor `interview` brings to a brand-new feature, just pointed at something
that already exists. The output isn't a plan for new work; it's a precise, approved decision about
something already built, written back into `SPEC.md` so it can flow through the same pipeline that
built it in the first place.

## Requires

An existing `SPEC.md` and `issues/` (from `doc-to-issues`). If neither exists, say so and point to
`doc-to-issues` — don't improvise a target from the codebase alone.

## Picking and validating the target

**One aspect per run — this is a hard rule, not a default.** If the user names something that's
really several aspects bundled together ("polish the whole HUD," "make the whole game feel
better"), don't start interviewing yet. Say what you see as the natural sub-aspects and help them
pick one to start with — the same reason `doc-to-issues` insists on small independent issues
applies here: a decision that's actually three decisions stapled together is much harder to reach
real, checkable agreement on, and much harder to hand off cleanly afterward.

If the user already knows what they want to polish, take it as given — don't second-guess a
specific, well-scoped target. If they don't have one in mind, or ask you to suggest candidates,
scan `SPEC.md` for placeholder-flagged or clearly-provisional details ("placeholder," "TBD," a
described-as-temporary color/behavior) inside sections whose feature is `qc-passed` in
`issues/FEATURES.md`, and offer a short menu.

When a named target has a placeholder asset (a ColorRect, a stock icon, a temporary sound) *and*
the project has a documented real-art production track (check `SPEC.md`'s Art Direction /
Open-Deferred sections), always offer "kick off real art production for this" as one of the
sub-aspect choices alongside the usual visual-tuning ones — don't assume tuning the placeholder is
what's wanted just because that's the more common run. If the user's phrasing already signals this
(mentions generating art, an asset pipeline, "real" vs "placeholder," or names the production tool
directly), skip straight to that path instead of offering a menu.

**Scope guard:** the target must correspond to something already built and verified — trace it to
a `done` issue under a feature marked `qc-passed` in `issues/FEATURES.md`. If it doesn't (the
thing doesn't exist yet, or its feature hasn't cleared QC), this isn't a polish job — it's new
work, or work that isn't stable enough to refine yet. Say so plainly and redirect to `interview`
instead of trying to force this skill to cover it.

**What's in scope once you have a valid target:** visual appearance (color, icon, shape), layout
and positioning, typography, animation and transition timing, sound cues if the project has audio,
and interaction feel — how something responds to input, whether it queues or reacts instantly,
whether a state change is jarring or smooth. Also in scope, as two distinct flavors of the same
"swap in real art" work:

- **Swapping in an asset that's already been produced** — placement, scale, timing, and any layout
  knock-on effects once a separate production track delivers a finished sprite/icon/sound. Also
  covers right-sizing the delivered file for runtime: if it's oversized relative to how large it's
  actually ever displayed (a common gap when the asset came from a generation tool defaulting to a
  large square canvas), downscale it to something closer to its actual on-screen footprint before
  wiring it in — a modest headroom for high-DPI screens (roughly 2-3x the max display size) is
  plenty, not the full generation resolution. An oversized runtime asset costs real VRAM/download
  size for no visible gain. If downscaling would destructively overwrite the only copy, keep the
  original delivered file too (see "File path and naming convention" below for where) rather than
  discarding it — a future higher-res need (an app icon, a store listing, marketing) shouldn't
  require regenerating art that already exists.
- **Kicking off production of an asset that doesn't exist yet**, when the project has a documented
  pipeline for making one (check `SPEC.md`'s Art Direction section — style reference, tool
  pipeline, frame/format conventions already fixed project-wide). This is still refining something
  *already built* (the placeholder and its call site exist and are `qc-passed`) — you're pinning
  down the concrete production brief for what replaces it, not designing new gameplay. See "Real
  art production kickoff" under Interview below for what that brief needs to cover.

**What's out of scope:** raw numeric/balance tuning — costs, growth rates, thresholds, difficulty
curves. Projects that defer this kind of tuning to its own pass (check `SPEC.md`'s Open/Deferred
items) should keep it there; don't let a polish run quietly turn into a balance-tuning session.
If the user brings up a number alongside a genuine feel question, separate the two: settle the feel
question here, note the numeric one as a nudge toward that separate pass.

If the target was already polished in an earlier run, treat the current `SPEC.md` wording as the
starting point to revise, not a placeholder to replace from scratch — read it as part of grounding
below, and ask what specifically isn't working now rather than re-asking questions already settled
last time.

## Ground yourself in the live app, not just the code

Visual and feel decisions can't be judged from source alone — launch the actual running app before
asking anything. Use the project's `run` skill if it has one; if the app has debug or fast-forward
tooling (a seeded state, a dev-mode panel, a way to jump past setup), use it to get to the state
where the target aspect is actually visible or exercisable, rather than describing it in the
abstract. Look at (screenshot where useful) the current, real behavior of the target — this is
what grounds every question that follows in what's actually there, not a guess from reading a
scene file or component. If the app can't be launched in this environment, say so and fall back to
reading the relevant source/config directly, same as `interview`'s own grounding step — but treat
that as a fallback, not the default.

## Interview

Same posture as `interview`: keep going in rounds until you're not guessing, not until you've asked
a fixed number of questions. Use `AskUserQuestion` for concrete decision points, batched up to 4
per call; when you have a real recommendation, lead with it and mark it "(Recommended)" rather than
presenting a false-neutral menu. Use free text for anything that doesn't reduce to a menu.

There's no fixed checklist — what's worth asking depends entirely on what's being polished — but
useful lenses for a typical UI/visual aspect include: does it need an icon or symbol at all, what
communicates its state changes, exact position and size relative to what's around it, color and
how it should read against the background, whether a change (a value updating, a new state
appearing) should animate and how, and what stays exactly as-is that the user might otherwise
assume is also up for grabs. For an interaction-feel aspect: what should happen on the triggering
input, what the previous behavior actually was and specifically why it read as wrong, and whether
the fix has any knock-on effect on something adjacent (timing, pacing, difficulty) worth flagging
even if it's not being decided here.

Each answer usually opens another question underneath it — that's the interview working normally,
not scope creep. Keep going until you could describe the finished aspect back to the user without
them needing to correct you.

### Real art production kickoff

When the target is kicking off production of a not-yet-made asset (see "What's in scope" above),
the interview has a fixed set of things it must nail down — the point is a brief concrete enough
that someone can go generate the asset and someone else (or the same person, later) can wire it in
without re-asking you anything:

- **Format and frame count/rate.** Reuse whatever `SPEC.md`'s Art Direction section already fixes
  project-wide (FPS, frames per animation, file format) rather than re-deciding it per target —
  only ask if the project has never pinned this down before.
- **Per-frame/asset dimensions.** If nothing's fixed yet, propose a working *generation* canvas
  size suited to the tool being used (e.g. a square canvas matching a common text-to-image
  default) and note explicitly that it doesn't need to match the current placeholder's on-screen
  size pixel-for-pixel. Don't assume that generation size is also the shipped runtime size, though
  — the wiring issue is expected to downscale it to the asset's actual on-screen footprint first
  (see "Swapping in an asset that's already been produced" above for the target size and how to
  avoid losing the original in the process).
- **File path and naming convention** for where the finished asset(s) land, consistent with the
  project's existing asset directory structure if one is documented — and where a kept original
  lives if the runtime version ends up downscaled from it, when the project has no existing
  convention for that.
- **Generation guidance** — reuse the project's documented pipeline (reference images, tools,
  post-processing steps) if `SPEC.md` already describes one; only design a new pipeline from
  scratch if it doesn't.
- **Fallback behavior**: confirm the current placeholder stays in place for this target (or for
  each variant, see below) until its specific replacement is delivered and wired in — nothing
  should go asset-less mid-transition.
- **If the target naturally has multiple variants** (one animation set per character state, per
  item type, per character/weapon/skin, etc.), get an explicit list of every variant needed — the
  written spec must enumerate each one by name, not just describe the pattern once, so
  `doc-to-issues` can generate one independent issue pair per variant rather than a single
  monolithic one. Also confirm the issue granularity itself: one production+wiring issue pair per
  variant (each ships independently as its art lands) versus one pair covering all variants
  together (simpler brief, but wiring waits on every variant being produced first) — lead with the
  per-variant recommendation, since it matches `doc-to-issues`' own preference for small
  independent issues.

## Stopping

Once genuinely out of open questions, print a numbered **"Shared Understanding"** summary: what's
changing about this aspect, the concrete decision on each point raised, and — just as important —
what's explicitly *not* changing, so nothing gets assumed into scope that was never discussed.

Then **stop**. Don't touch `SPEC.md`, don't generate or touch anything under `issues/`. Ask the
user to confirm the summary or correct it; fold in corrections and re-summarize until they
explicitly approve. This is a hard gate, same as `interview`'s.

## Writing it down

Once — and only once — approved: update the relevant existing `SPEC.md` section **in place** with
the validated decision. Reconcile with what's already written there rather than appending a
disconnected block; if the section currently describes a placeholder, replace that description
with the real one — for a real-art production kickoff, that means describing the placeholder as
today's *fallback* (still true until each variant is wired in) alongside the concrete target spec,
not deleting the placeholder description outright. Don't create a new "Polish" section and don't
create a separate log or changelog file — git history already records what changed and when, and a
second place to look for the current truth is worse than one.

For a multi-variant production kickoff, write out every variant by name (not just the pattern) —
this is what lets `doc-to-issues` generate one issue pair per variant on the next sync instead of
one monolithic issue.

Do not modify `issues/INDEX.md` or `issues/FEATURES.md`, and do not write any issue file yourself.

## Handing off

Tell the user the concrete next step is running `doc-to-issues` to sync — then stop. This is
deliberate, not a shortcut: `doc-to-issues`'s own "Syncing on rerun" behavior already covers this
exact situation without any new logic. When a `done` issue's corresponding spec content changes
incompatibly, `doc-to-issues` marks that issue `stale` (with a note explaining what changed) and
generates a fresh issue for the new decision — which also flips that feature's `FEATURES.md` row
back from `qc-passed` to `not-ready`, so `qc` naturally re-verifies it once the new issue is built.
Reimplementing any part of that here would just be a second, divergent copy of logic that already
exists and is already trusted — this skill's job ends at a validated `SPEC.md`.

For a real-art production kickoff specifically, tell the user what to expect from that sync so the
handoff isn't a black box: `doc-to-issues`' owner classification should read the production brief
as `owner: user` (making/delivering an asset needs the human production track — a Leonardo.ai/etc.
pipeline isn't something the agent can run) paired with an `owner: agent` issue for wiring the
delivered asset into the scene, and one such pair per variant if the spec enumerated several. If
the resulting issues don't come out that way, that's worth flagging back rather than silently
accepting a different split.
