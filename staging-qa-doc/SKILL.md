---
name: staging-qa-doc
description: Generate or update a Claude Doc of manual QA test scenarios covering everything that is on the `staging` branch but not yet on `main` (origin/main...origin/staging). Reads the commits, the code diffs and the tests they add to write precise, step-by-step "Cas N — Titre" scenarios (exact UI labels, routes, expected results in bold, ready-to-paste CSV/data blocks), a prerequisites section and smoke tests per role for technical changes, built for the tester — one-click status dropdown per case (À tester / OK / KO / Bloqué / À retester / Obsolète) in a boxed case header, steps as a table (# / Action / Attendu / Résultat checkbox / Capture) and a linked summary table. On rerun within the same cycle (main base unchanged) it updates the existing doc in place — adds cases for new commits, flags "À retester" cases touched by later fixes, strikes obsolete ones, never touches the user's screenshots or comments; after staging is merged into main it starts a new doc. Use when the user wants a QA / test-scenario document for staging vs main, or invokes /staging-qa-doc. Not the same as the `qc` skill (which tests one issues/ feature against SPEC.md).
---

# Staging QA doc

Produce a living **Claude Doc** that a human uses to manually qualify the state of the `staging`
environment against `main`: one numbered test case per user-visible change, precise enough to be
executed without reading the code, and a place where the tester pastes screenshots, sets statuses
and leaves comments.

This skill is global: it stores no per-project configuration. It infers what it can from the repo
and from the previous QA doc, and asks the user for the rest.

## Hard rules

- Never commit, never push, never modify repository files (including `SPEC.md`). The only output is
  the Claude Doc.
- Never put secrets in the doc: no passwords, tokens, API keys, cookie values, `.env` values,
  connection strings. No real user or patient data. Test e-mail addresses supplied by the user are
  allowed (e.g. `name+test1@domain`).
- When updating an existing doc, **never delete or modify** images/media blocks or comments the user
  added, nor text they wrote. Only add, annotate, or change the parts this skill owns (title date,
  state line, summary table, a case's status dropdown and Résultat checkboxes when
  flagging it « À retester » or « Obsolète », new cases, « À retester » notes, obsolete markers).
  Never change a status the tester set for any other reason.
- Write the doc in the user's conversation language (French by default for this user).

## Step 1 — Git scope

```bash
git fetch origin --quiet
BASE=$(git merge-base origin/main origin/staging)
STAGING=$(git rev-parse origin/staging)
git log --oneline --no-merges $BASE..$STAGING
git diff --stat origin/main...origin/staging
```

Always compare `origin/main...origin/staging` (what is actually deployed), never local branches.
If `origin/staging` has nothing that `origin/main` lacks, say so and stop.

Derive the project **slug** from the origin remote — repo name, lowercased
(`basename -s .git "$(git remote get-url origin)" | tr '[:upper:]' '[:lower:]'`, e.g. `asce`) —
and the `owner/repo` used in the state line.

## Step 2 — Find the current doc and decide: new or update

1. Load the docs tooling (the `anthropic-skills:docs` skill if listed, else the Claude Docs
   connector's `guide` with `topic.index`) before any docs call.
2. Look for the latest QA doc of this project: list the user's docs/artifacts and match titles
   starting with `[<slug>] Scénario de test — staging`; read the most recent candidate and check
   that its **state line** names this repository (the title filters, the state line is the
   authority). Docs from before the slug convention (title without `[<slug>]`, no state line) are
   never auto-matched — the user pastes their link. Show the user what you found and ask them to
   confirm, paste another link, or say "nouveau".
3. Read the confirmed doc's state line (`Dépôt : … · Base main : <sha> · Staging couvert jusqu'à :
   <sha>`).
   - Base in the doc ≠ current `$BASE` → staging was merged into main since: **new cycle → create a
     new doc** (leave the old one untouched as a record).
   - Same base → **update** that doc; the new commits to cover are `<covered sha>..$STAGING`.
   - No doc found / user says "nouveau" → create a new doc.

## Step 3 — Ask the user (every run)

With one `AskUserQuestion` call (plus free text where needed):
- **Staging URL** — prefill from the previous doc, or from the repo (deploy config, `.env.example`,
  docs, README) if found.
- **Test e-mail addresses** to use in data blocks and account prerequisites (e.g. one allowed
  address, one non-allowed address, one on an SSO domain — whatever the cases need).

Do not ask the user to validate the list of cases before writing; write the doc directly.

## Step 4 — Filter commits

Ignore:
- automated build commits (`build: automatic update…`) and merge commits;
- commit + revert pairs (they cancel out — e.g. temporary debug logs then their revert);
- commits that only touch tests, CI, docs/runbooks or tooling scripts with no effect on the running
  staging app.

Keep, as **smoke tests per role**: technical changes with a user-visible blast radius (framework
migration, dependency upgrades, config changes such as cookie/session duration, build/runtime
changes). Group them into one case per role (e.g. "Smoke test <migration> : parcours Apprenant")
listing the routes to visit and the symptoms that would reveal a regression.

## Step 5 — Analyse the changes (read the code)

For each kept change, read the actual diff, not only the commit message. Launch parallel
`Explore`/`general-purpose` subagents per functional area when the diff is large. Extract:
- **exact UI strings** (button labels, headings, error/success messages, helper texts) — quote them
  between « » in the doc;
- **routes/pages** touched (`/login`, `/teacher/exams/[id]`…), and which **roles** can reach them;
- **validation and business rules** (mandatory fields, conditions per role or mode, limits,
  durations, throttling/cooldowns, one-time use, expiry);
- **security-relevant behaviour** worth a manual check (anti-enumeration, blocked flows, ignored
  server-side inputs);
- the **tests added in the diff**: use them to derive edge cases and the expected results.

## Step 6 — Write the doc

Structure (in this order):

1. `# [<slug>] Scénario de test — staging JJ/MM/AAAA` (today's date), e.g.
   `[asce] Scénario de test — staging 30/09/2026`. Use the same string as the doc's name.
   When updating a pre-slug doc, add the `[<slug>]` prefix to its title.
2. Date block + mention of the user (`me`).
3. Intro: « Ce scénario qualifie l'état de staging à la date du JJ/MM/AAAA. »
4. The staging URL as a link.
5. **State line** (owned by this skill, keep it exact so reruns can parse it):
   `Dépôt : <owner/repo> · Base main : <short sha> · Staging couvert jusqu'à : <short sha>`
6. `## Sommaire` — table **Cas | Rôle | Objectif | Commit(s)**, one row per case, the case cell an
   in-doc link to its section. **No status column**: the status lives only in each case header, so
   the tester never has two dropdowns to keep in sync. No progress line anywhere in the doc.
7. `## Pré-requis` — environment access, accounts per role needed, the test addresses and what each
   is for, any durations/limits the tester must know (link validity, cooldowns…).
8. One `## Cas N — <titre fonctionnel>` per change, flat list (no grouping by domain), ordered by
   user journey (auth first, then admin, then per-role flows, then smoke tests, then visual checks).
   No horizontal rule between cases. Each case contains, in order:
   - a **boxed header** (a blockquote, four lines, each its own line):
     `> **Statut** : <dropdown>` / `> **Rôle** : …` / `> **Page(s)** : <links>` / `> **Objectif** : …` —
     the status is a **dropdown chip** of the doc's `Statut` enum, pre-set to « À tester »; pages are
     clickable links to the staging URL; the objective is one sentence (what changed, why tested);
   - optionally **Données**: a bold `**Données**` line, then each ready-to-paste block (CSV, JSON,
     payload) under a short label « CSV A — apprenant sans date », « CSV B — corrigé »…, using the
     user's test addresses (a failing and a corrected version when the case tests validation). The
     steps refer to them by label (« importer le CSV A ») — data never goes inside the table;
   - the **steps table** `| # | Action | Attendu | Résultat | Capture |`, one row per step, one action
     per row (so the tester can say « KO à l'étape 5 »): Action = where to go / what to click / what
     to type; Attendu = the exact message or UI state (several checks of one action go in the same
     cell, separated by « ; »); **Résultat** = an unticked checkbox; **Capture** = an empty cell
     where the dev pastes the screenshot. Negative / edge variants (other role, other mode, repeated
     action, limit exceeded) are extra rows starting with « Variante : ».
   A KO is reported by the tester as a **comment on the row concerned**, plus the case status set to
   KO — no dedicated block. Don't insert placeholder images.

Doc mechanics (Claude Docs connector — read its guide for exact call shapes):
- Create the doc with its skeleton first (title, byline, one pending block per section), then fill
  each section, one call per section.
- Create the **`Statut` enum once per doc**, options in this order: « À tester », « OK », « KO »,
  « Bloqué », « À retester », « Obsolète » (index 0–5), in the same batch as the first case fill;
  every case's status is a `dropdown` chip on that enum (`index: 0` at creation). In update mode,
  reuse the existing enum (read its id off an existing status chip) — never create a second one.
- Boxed header: separate the four blockquote lines with an empty `>` line, otherwise they merge
  into one paragraph.
- Steps table: write it in markdown with the Résultat and Capture cells empty (a `- [ ]` typed in
  a markdown cell stays literal text). Then, off the fill ack's table id, replace each Résultat
  cell (`{"kind":"cell","table":<id>,"row":r,"col":3}`, rows from 1, guard `ifRev`) with a real
  checkbox, `"as":"blocks"`:
  `[{"type":"list","attrs":{"kind":"check"},"content":[{"type":"listItem","attrs":{"checked":false},"content":[{"type":"paragraph"}]}]}]`.
  Many cell replacements fit in one `update` call. Never put a pipe `|` inside a cell's text.
- Write the cases first and the summary table last: its case cells link to each case's heading
  block id (`[3 — Titre](#<heading block id>)`), read off the fill acks.
- Keep the doc private; tell the user they can share it with their organisation themselves.

## Step 7 — Update mode (same cycle)

Read the whole current doc (content + comments) before writing, then apply targeted edits only:
- **New changes** from `<covered sha>..$STAGING` → append new cases (continue numbering, never
  renumber existing ones), status « À tester », and add them to the summary table.
- **Later commits touching an existing case's feature** (e.g. `fix(review): QA …`) → under that
  case's header add a note « À retester après `<sha>` : <what changed, concretely> », add any new or
  changed rows, set its status dropdown to « À retester » and **untick its Résultat checkboxes**
  (put the note as an extra line of the boxed header). Read the comments on that case: if the fix answers a KO comment, say so in the note
  (« corrige le KO signalé à l'étape 5 »).
- **Case made obsolete** (feature reverted or removed) → strike through its title (or prefix
  « [OBSOLÈTE] » if strikethrough isn't supported), add one line explaining why, and set its status
  dropdown to « Obsolète ». Never delete it.
- Update the title date, the state line (`Staging couvert jusqu'à`) and the summary table.
- Leave every image, comment, user-written text and every other status exactly as it is.

## Step 8 — Report

End with one short message: new doc or updated doc, the link, how many cases
were added / flagged « À retester » / marked obsolete, and any change you deliberately left without
a case (and why).
