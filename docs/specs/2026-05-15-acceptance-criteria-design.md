# Design: Acceptance Criteria for Waypoint Milestones

**Date:** 2026-05-15
**Branch:** `acceptance-criteria`
**Plugin version target:** `0.3.0`

---

## Purpose

Add structured Acceptance Criteria (ACs) to Waypoint milestone files. ACs
capture *what success looks like* at the milestone level, in business or
capability terms, in a notation that fits each criterion's nature. Where
possible, each AC is verified automatically by a `verify:` command; where
not, it is explicitly flagged as `uat` for human sign-off.

ACs integrate into the existing Waypoint flow with no new commands:

- `/waypoint:milestone` (via the brainstorming skill) captures ACs during
  milestone definition.
- `/waypoint:phase` reviews and assigns AC verify generation to whichever
  phase fully delivers each AC.
- `/waypoint:done` runs verify commands and gates milestone completion on
  AC status.

This replaces the loose "success signal" topic currently embedded in
brainstorming's Step 3 with a structured artefact that survives
milestone close, and adds a hard machine-checkable gate on completion.

The full feature requirements live in the upstream feature spec at
`waypoint-acceptance-criteria-spec.md` (provided by the user); this
design adapts that spec to the current Waypoint plugin internals and
resolves four points that were open in the upstream document:

1. AC scope and verify generation are per-milestone, not "last phase".
2. AC integration into the brainstorming skill replaces (not augments)
   the existing Success Signal topic.
3. Mid-milestone AC edits are direct file edits, with a lightweight
   prompt in `/waypoint:phase`.
4. The chosen UX scope is "Minimal + AC status summary line" (no new
   `/waypoint:verify` command).

---

## File-level summary

### Created

- `docs/specs/2026-05-15-acceptance-criteria-design.md` — this design.

### Modified

- `plugins/waypoint/skills/waypoint/assets/milestone_skeleton.md` —
  new `## Acceptance Criteria` section between `Done When` and `Phases`;
  one new bullet appended to `Done When` boilerplate.
- `plugins/waypoint/skills/brainstorming/SKILL.md` — Step 3's
  "Success signal" topic replaced with an AC-oriented discussion; Step 5
  gains a fifth validated section (Acceptance Criteria); Step 4 may
  optionally propose a final verification phase; Step 6 writes the AC
  section into the new milestone file; an Appendix with AC format and
  type reference is added; "What this skill must never do" gains one
  bullet forbidding tech/test-tool discussion during AC capture.
- `plugins/waypoint/commands/milestone.md` — adds a one-step preamble
  that ensures the CLAUDE.md AC rules block is present before delegating
  to brainstorming (idempotent append-if-missing).
- `plugins/waypoint/commands/phase.md` — new step 5 (AC review) inserted
  between the existing stub-grep step and plan-proposal step; existing
  steps 5–7 renumber to 6–8.
- `plugins/waypoint/commands/done.md` — phase-completion flow gains a
  step 4a (verify-script presence check for ACs assigned to this phase);
  milestone-completion flow gains steps 5a–5e (pre-flight, run verifies,
  failures, UAT, success).
- `plugins/waypoint/commands/init.md` — appends the new AC rules
  paragraph to the CLAUDE.md rules block written at init time.
- `plugins/waypoint/skills/waypoint/SKILL.md` — Situation 1 rules block
  mirrors the new init output; Situation 4 gains the AC verify-presence
  check; Situation 5 mirrors `/waypoint:done` steps 5a–5e.
- `plugins/waypoint/.claude-plugin/plugin.json` — version bump from
  `0.2.0` to `0.3.0`.
- `README.md` — adds an Acceptance Criteria paragraph in "Key concepts";
  notes in "Getting started" that `/waypoint:done <slug>` runs AC
  verification before closing.

### Unchanged

- `plugins/waypoint/skills/waypoint/assets/roadmap_skeleton.md` — already
  has the `**Stack:**` field used for stack detection.
- `plugins/waypoint/commands/archive.md` — archival is independent of ACs.
- `.claude-plugin/marketplace.json` — no version field; nothing to change.

---

## Milestone file format

### New section — `## Acceptance Criteria`

Inserted between `## Done When` and `## Phases` in `ms-<slug>.md`.
Skeleton boilerplate:

```markdown
## Acceptance Criteria

> Business-level success criteria for this milestone. Each criterion is
> verified automatically where possible; `uat` criteria require human
> sign-off. A milestone cannot be marked complete while any criterion is
> ⬜ untested or ❌ failed (see CLAUDE.md rules).

_(none yet — populated during milestone definition)_
```

After brainstorming finishes, the `_(none yet ...)_` line is replaced
with one or more AC blocks of the form:

```markdown
### AC-<n>: <Short title>
format: <gherkin|user-story|decision-table|ears|sequence|bpmn>
type: <capability|invariant|integration|performance|uat>

<criterion body — format-specific>

verify: `<shell command>` | TODO | manual
status: ⬜ untested | ✅ passed | ❌ failed
```

The six formats and five types, with "Use when" guidance and concrete
examples, are reproduced verbatim from the upstream spec sections
"Acceptance Criteria Formats" and "Criterion Types" — they live in
the brainstorming skill's Appendix (see below), not the milestone
skeleton.

### `Done When` boilerplate addition

Append one bullet to the existing `Done When` boilerplate:

```markdown
- All Acceptance Criteria are ✅ passed (see the Acceptance Criteria section).
```

So the full skeleton `Done When` block becomes:

```markdown
## Done When

<Concrete, verifiable completion criteria.>

- All phases complete.
- Zero `STUB[ms-<slug>]` comments remain in the codebase
  (`grep -r "STUB\[ms-<slug>" src/` returns no results).
- All Acceptance Criteria are ✅ passed (see the Acceptance Criteria section).
```

### Numbering and edit rules

- **Stable numbering.** AC numbers, once assigned, do not shift. Removing
  AC-2 leaves AC-1 and AC-3 in place; the next new AC becomes AC-4.
  Rationale: verify scripts, phase plans, and the Decisions Log can
  reference `AC-N` by name; renumbering would invalidate those refs.
- **Status reset on material edit.** If an AC's body, format, type, or
  verify command is changed after the AC reached `✅ passed`, its status
  resets to `⬜ untested`. Trivial title edits do not reset status.
  The brainstorming and phase prompts enforce this when changes go
  through them; direct file edits are the user's responsibility — the
  rule is restated in the CLAUDE.md rules block so it is salient.

---

## Brainstorming skill changes

### Step 3 — replace Success signal topic

The existing five resolution topics become **four** plus an AC-oriented
discussion. Replace the bullet currently labelled "Success signal" with:

> **Acceptance criteria.** What does success look like when this
> milestone is done? Describe the behaviors, invariants, or workflows
> that must hold — in business or capability terms, not in test-tool
> terms. Push for things that are observable and verifiable. This
> conversation produces the milestone's structured Acceptance Criteria
> (translated in Step 5).

The other four topics (Goal clarity, Beneficiary, Existing foundation,
Scope boundary) are untouched.

### Step 4 — optional final verification phase

Step 4 currently proposes 2–3 phase-decomposition alternatives. Add the
guidance: when ACs are numerous, mutually dependent, or live in a
regulated/safety-critical domain, brainstorming MAY include a final
verification phase as one of the proposed decompositions. This is not
mandatory and most milestones will not need it.

### Step 5 — add Acceptance Criteria as the fifth validated section

Step 5 currently validates four sections in sequence: Goal → Scope/Out
of Scope → Done When → Phases. Add a fifth, presented after Phases is
approved:

> **5. Acceptance Criteria.** Propose 2–4 (occasionally more) criteria
> covering what was discussed in Step 3. For each, pick the format that
> best fits its nature (gherkin, user-story, decision-table, ears,
> sequence, bpmn) and the type (capability, invariant, integration,
> performance, uat). Set `verify: TODO` and `status: ⬜ untested` for
> non-`uat` ACs; for `uat`, set `verify: manual`. The user may push
> back on format, type, or wording; iterate until approved.

The final confirmation line stays *"Ready to write the milestone file?"*.

### Step 6 — write AC blocks into the milestone file

Replace the `_(none yet ...)_` placeholder in the milestone skeleton's
`## Acceptance Criteria` section with the approved AC blocks, in order,
numbered AC-1, AC-2, … `Architecture References`, `Decisions Log`, and
`Deferred Wiring` continue to be populated later, not at brainstorming
time.

### "What this skill must never do" — new bullet

Append:

- Do not discuss verify commands, test tools, or framework choices
  during AC capture. That belongs to phase planning. ACs at
  milestone-definition time are tech-agnostic capability statements
  with `verify: TODO`.

### Appendix — AC formats and types reference

Add a final section at the bottom of `brainstorming/SKILL.md` titled
`## Appendix — Acceptance Criteria formats and types`. Content:

- Reproduce the upstream spec's "Acceptance Criteria Formats" section
  verbatim: gherkin, user-story, decision-table, ears, sequence, bpmn —
  each with its "Use when" guidance and a concrete example.
- Reproduce the upstream spec's "Criterion Types" table verbatim:
  capability, invariant, integration, performance, uat.

This is reference material the skill consults in Step 5 when picking
formats and types; keeping it inline avoids a secondary lookup.

---

## `/waypoint:phase` changes

The phase command currently has 7 numbered steps. Insert a new step 5
between the existing step 4 (stub grep) and existing step 5 (brainstorm
and propose plan). Renumber existing steps 5–7 to 6–8.

### New step 5 — Acceptance Criteria review

1. Read the `## Acceptance Criteria` section of the milestone file.
2. Print a one-line summary: `ACs: N total — X ✅ / Y ⬜ / Z ❌ / W manual`.
3. Prompt the user:
   > *"Based on what you've learned so far, any AC changes needed
   > before planning this phase? (add / edit / remove / none)"*
   Apply edits to the milestone file. New ACs are appended with the
   next free AC-N, `verify: TODO`, `status: ⬜ untested`. Material edits
   to existing ACs reset their status to ⬜ (see the milestone format
   section).
4. For each AC with `verify: TODO`: ask
   > *"Will this phase fully deliver AC-N: \<title\>?"*
   If yes, add an explicit verify-generation task to the phase plan
   (described below). If no, the AC stays `verify: TODO` for a later
   phase.
5. `uat` ACs (`verify: manual`) are never assigned verify-generation
   tasks. They remain `verify: manual` throughout the milestone.

### Verify-generation task shape

When an AC is assigned to this phase, the task added to the plan reads:

> *Generate verify script for AC-N (\<title\>) — \<chosen approach
> from stack detection\>.*

The actual script or test file is written during phase implementation.
The milestone file's `verify:` field is updated at that time, from
`TODO` to the real command.

### Stack detection

In order, stop at the first confident signal:

1. Read `docs/roadmap.md`'s `**Stack:**` field. If populated and
   specific, use it.
2. Else, check whether the user has stated the stack in the current
   conversation.
3. Else, ask the user with a concrete recommendation based on visible
   source files:
   > *"I didn't find a stack in the roadmap. Since your source files
   > look like \<observation\>, I'd suggest \<tool\> — OK to use that?"*

### Verify-strategy mapping

Once the stack is known, pick the verify approach per the table below
(reproduced from the upstream spec):

| AC type | Priority order |
|---|---|
| `capability` / `gherkin` | BDD runner if present (cucumber, behave, pytest-bdd) → otherwise e2e (playwright, cypress, httpx) |
| `capability` / other formats | Integration test in project's primary test runner |
| `invariant` | Property test (hypothesis, fast-check, quickcheck) → if not present, parameterized unit test |
| `integration` | Contract test if framework exists (pact) → otherwise probe script against real/mock service |
| `performance` | Existing bench harness (k6, locust, wrk) → otherwise `time` + threshold assertion in shell |
| `uat` | Always `manual` — no inference |

### Test placement

- If `tests/` or `spec/` exists, read it to infer conventions before
  generating any test files.
- If no AC-style tests exist yet, ask the user:
  > *"Add to the existing test tree or create `tests/ac/`?"*
  Recommend `tests/ac/`.
- If no test infrastructure exists at all, generate a self-contained
  script at `tests/ac/ms-<slug>/verify_ac_<n>.sh`.

### Renumbered existing steps

- Existing step 5 (brainstorm and propose plan) → step 6. Stub-resolution
  tasks still come first; verify-generation tasks come after stubs but
  before the rest of the plan.
- Existing step 6 (architectural decisions) → step 7.
- Existing step 7 (phase row update) → step 8.

No other changes to the existing steps.

---

## `/waypoint:done` changes

### Phase completion flow — step 4a

After the existing step 4 (update phase row to ✅ Complete), insert step 4a:

> 4a. **AC verify-script presence check.** For any AC whose
> verify-generation was assigned to this phase, verify that the script
> or test file now exists and the milestone file's `verify:` field is
> populated (no longer `TODO`). If a verify is still `TODO`, flag the
> user and ask whether the AC was actually delivered. **Do not run the
> verify command at phase completion** — verification runs only at
> milestone completion.

This catches the failure mode where the phase plan said "generate
verify for AC-N" but the work was skipped or forgotten.

### Milestone completion flow — steps 5a–5e

Insert between the existing step 5 (full stub grep) and step 6
(milestone status flip).

> 5a. **Pre-flight AC check.** Print the AC summary line: `ACs: N
> total — X ✅ / Y ⬜ / Z ❌ / W manual`. Scan for any AC with
> `verify: TODO`. If found, **STOP** — list them and prompt the user
> that verify scripts must be generated (likely via a missed
> `/waypoint:phase` step) before the milestone can close. Do not flip
> status.
>
> 5b. **Run verifies.** For each AC with `verify:` set to a real shell
> command (i.e., not `TODO`, not `manual`): run the command sequentially,
> capture its exit code, and update the milestone file's `status:`
> field — `✅ passed` on exit 0, `❌ failed` on any other exit. Sequential
> execution keeps failure output unambiguous.
>
> 5c. **Handle failures.** If any AC ended `❌ failed`: **STOP**. Print
> the failure output for each, print the updated AC summary line, and
> prompt the user to either fix the implementation or — if the criterion
> genuinely cannot be tested automatically — change the AC's type to
> `uat` and document why. Do not flip status. Do not auto-retry; the
> user re-invokes `/waypoint:done` after fixing.
>
> 5d. **Manual / UAT check.** For each AC with `verify: manual`: confirm
> the user has set `status: ✅ passed`. If any is still `⬜ untested`,
> **STOP** and list them, prompting the user to perform the UAT and
> update status. Do not flip status.
>
> 5e. **All passed.** Only when every AC is `✅ passed` proceed to step
> 6 (existing milestone status flip).

The existing step 6 (flip milestone to ✅ Complete, update roadmap row)
and step 7 (suggest archival) are unchanged.

---

## CLAUDE.md rules block — new paragraph

Appended to the existing roadmap rules block by `/waypoint:init` and
re-appended idempotently by `/waypoint:milestone` if missing:

```markdown
### Acceptance Criteria

A milestone cannot be marked complete while any AC has:
- status: ⬜ untested  (run /waypoint:done to trigger verification)
- status: ❌ failed    (fix the failure, or change type to `uat` and document why)
- verify: TODO         (assign and generate verification during the phase that delivers the AC)

UAT-type criteria (verify: manual) must have status set to ✅ passed
explicitly by the user before /waypoint:done will proceed.

Editing an AC's body, format, type, or verify command after it has
passed resets its status to ⬜ untested. Trivial title edits do not.

Do not invent or skip ACs to unblock completion. If a criterion cannot
be tested automatically, change its type to `uat` and document why.
```

This paragraph is added in three places (all kept in lockstep):

1. `plugins/waypoint/commands/init.md` — the rules-block heredoc.
2. `plugins/waypoint/skills/waypoint/SKILL.md` — Situation 1's "CLAUDE.md
   rules block to append" heredoc.
3. `plugins/waypoint/commands/milestone.md` — the idempotent
   append-if-missing logic checks for the literal `### Acceptance
   Criteria` heading inside `CLAUDE.md` before appending.

---

## `/waypoint:milestone` preamble

The command currently delegates to `waypoint:brainstorming` immediately.
Add a one-step preamble:

```
Before delegating: check CLAUDE.md for the heading `### Acceptance
Criteria` inside the Roadmap & Stub Rules section. If missing, append
the AC rules paragraph (see CLAUDE.md rules block above). If present,
do nothing. Then invoke the `waypoint:brainstorming` skill, passing
$ARGUMENTS through.
```

This makes `/waypoint:milestone` self-healing for projects initialised
before v0.3.0 without requiring re-running `/waypoint:init`.

---

## `waypoint:waypoint` skill changes

The natural-language entry points must match the slash-command behaviour.

### Situation 1 — Initialize

Append the AC rules paragraph (above) to the "CLAUDE.md rules block to
append" heredoc.

### Situation 3 — Starting a phase

After the existing stub-grep step, add the AC review step (read AC
section, print summary, prompt for edits, ask per-TODO-AC whether this
phase delivers it, add verify-generation tasks to the plan). Mirrors
`/waypoint:phase` step 5.

### Situation 4 — Finishing a phase

After the existing step 3 (stub bookkeeping), add: *"For any AC whose
verify-generation was assigned to this phase, confirm the script/test
file exists and the milestone file's `verify:` field is no longer
`TODO`."* Mirrors `/waypoint:done` step 4a.

### Situation 5 — Completing a milestone

After the existing zero-stubs grep step, add the same five-step AC
verification sequence (5a–5e) from `/waypoint:done`. The wording is
identical to keep the two entry points in lockstep.

---

## `plugin.json`

```diff
-  "version": "0.2.0",
+  "version": "0.3.0",
```

No other manifest changes.

---

## `README.md`

Two small additions, both in the existing structure:

1. **Key concepts** — add a paragraph between "Stubs" and "Architecture
   docs":

   > **Acceptance Criteria.** Each milestone declares structured
   > acceptance criteria (ACs) in its file — business-level success
   > criteria in whichever format fits each one (gherkin, user-story,
   > decision-table, EARS, sequence diagram, BPMN). Non-UAT criteria are
   > verified automatically by a `verify:` command at milestone
   > completion; UAT criteria require human sign-off. A milestone
   > cannot be marked complete while any AC is untested, failed, or
   > has no verify script.

2. **Getting started** — extend step 4's parenthetical:

   > 4. **`/waypoint:done auth 1`** — marks phase 1 complete, checks for
   >    any new stubs that need to be tracked, prompts for any
   >    architectural decisions made during implementation, and (if any
   >    ACs were assigned to this phase) confirms the verify scripts
   >    exist.

   And add a sentence to step 5:

   > 5. Repeat for subsequent phases. Once every phase is done, run
   >    `/waypoint:done auth` to close out the milestone. This runs all
   >    AC verify commands and refuses to close if any AC is untested,
   >    failed, or still has `verify: TODO`. When several milestones
   >    are complete, run `/waypoint:archive` to move them out of the
   >    active index.

---

## Out of scope

Per the upstream spec, plus a few items specific to this implementation:

- AC inheritance across milestones. Each milestone owns its ACs in full.
- Automatic re-run of verify commands on code change. CI integration is
  the project's responsibility.
- AC versioning or history beyond the milestone file itself. Decisions
  Log entries for AC changes are not auto-generated by this
  implementation — the user may add them by hand.
- A dedicated `/waypoint:verify` command. Considered and rejected — no
  use case beyond `/waypoint:done` justifies it for v0.3.0.
- A dedicated `/waypoint:ac` command for structured AC editing. Direct
  file edit plus the prompt in `/waypoint:phase` is sufficient.
- Changes to `/waypoint:archive`. Archival is unaffected by ACs.
- A migration command for projects with pre-existing milestones that
  pre-date ACs. Such milestones close as before; AC enforcement only
  activates for milestones that contain an `## Acceptance Criteria`
  section.

---

## Acceptance criteria for this design

- `plugins/waypoint/skills/waypoint/assets/milestone_skeleton.md` has a
  `## Acceptance Criteria` section between `Done When` and `Phases`, and
  the new bullet appended to `Done When`.
- `plugins/waypoint/skills/brainstorming/SKILL.md` no longer mentions
  "Success signal" as a Step 3 topic; in its place is the AC-oriented
  bullet described above. Step 5 contains a fifth validated section
  (Acceptance Criteria). Step 6 writes AC blocks into the milestone
  file. An Appendix at the end of the file lists the six AC formats and
  five AC types with examples.
- `/waypoint:milestone` from a fresh project drives end-to-end through
  brainstorming and produces a milestone file with one or more AC
  blocks in the new section.
- `/waypoint:milestone` re-run on a project that already has an active
  roadmap but no AC rules block in `CLAUDE.md` appends those rules
  idempotently and then proceeds normally.
- `/waypoint:phase` prints the AC summary line, prompts for AC edits,
  and for each `verify: TODO` AC asks whether the phase delivers it; on
  yes, a verify-generation task is added to the plan with a concrete
  stack-aware approach.
- `/waypoint:done <slug> <N>` for a non-final phase checks
  verify-script presence for any AC assigned to that phase, and flags
  missing scripts. It does not run verify commands.
- `/waypoint:done <slug>` for the final phase runs all non-manual
  verify commands sequentially, updates AC statuses, refuses to flip
  the milestone if any AC is `⬜`, `❌`, or `verify: TODO`, and prints
  the AC summary line on each block.
- Editing an AC's body/format/type/verify after it has passed resets
  its status to `⬜ untested` (when the edit goes through the
  brainstorming or phase prompts; direct file edits rely on the
  CLAUDE.md rules to remind the user).
- `plugin.json` reflects version `0.3.0`.
- `README.md` describes ACs in "Key concepts" and references AC
  verification in "Getting started".
- All natural-language entry points (`waypoint:waypoint` Situations 3,
  4, and 5) mirror their slash-command equivalents — saying "plan
  ms-X phase 2" or "ms-X is done" produces the same AC behaviour as
  the corresponding slash command.
