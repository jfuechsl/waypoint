---
name: waypoint
description: >
  Project roadmap and milestone planning for Claude Code projects. Use this
  skill whenever the user wants to set up a project roadmap, create or plan a
  milestone or phase, archive milestones, or manage architecture documentation.
  Triggers on phrases like "set up waypoint", "new milestone", "plan phase",
  "start phase", "add milestone", "archive milestone", "architecture decision",
  or "initialize project structure".
---

# Waypoint Skill

Manages a slug-based, file-per-milestone roadmap with stub tracking and living
architecture documentation. Detect which situation applies and jump to it.

---

## File Layout

```
docs/
  roadmap.md                        # Active milestone index
  milestones/
    ms-<slug>.md                    # One file per active milestone
    archive/
      index.md                      # Archived milestone index
      ms-<slug>.md                  # Archived milestone files
  architecture/
    <topic>.md                      # Living architecture documentation
```

Skeletons are at:
- `assets/roadmap_skeleton.md`
- `assets/milestone_skeleton.md`

---

## Situation 1 — New project: initialize roadmap

When the user wants to set up a roadmap for a new project:

1. Ask for project name and stack if not known.
2. Ask the user to describe the milestones they have in mind (rough is fine).
3. Create the directory structure:
   ```
   docs/milestones/
   docs/milestones/archive/
   docs/architecture/
   ```
4. Copy `assets/roadmap_skeleton.md` → `docs/roadmap.md`. Fill in the header.
5. For each milestone the user described, create `docs/milestones/ms-<slug>.md`
   from `assets/milestone_skeleton.md` and add a row to the index in
   `docs/roadmap.md`.
6. Create `docs/milestones/archive/index.md` with the archived index header
   (empty table).
7. Append the **CLAUDE.md rules block** (below) to `CLAUDE.md` (create if
   missing). Tell the user what was added and why.
8. Confirm with the user before writing any files.

**CLAUDE.md rules block to append:**

```markdown
## Roadmap & Stub Rules

### Phase sequencing within a milestone
Phases are strictly sequential. Do not start phase N until phase N-1 is
✅ Complete. Each phase builds on the previous — review the prior phase's
Handoff note before planning the next one.

### Starting any milestone phase
Before planning, grep for stubs scoped to the current milestone:

    grep -r "STUB\[ms-<slug>" src/

Add every hit as an explicit task: locate → implement → wire → delete comment.
Never begin planning without running this grep first.

### Deferring logic during implementation
Both steps are required — doing only one is a bug.

1. Add an inline comment at the stub site:
   `// STUB[ms-<slug>/phase-N]: description of what needs to be implemented`
2. Add a row to the **Deferred Wiring** table in `docs/milestones/ms-<slug>.md`.

Stubs MUST only be scoped to the current milestone's slug. Cross-milestone
stubs are not allowed.

### Completing a milestone
A milestone CANNOT be marked complete while any STUB[ms-<slug>] comment
exists in the codebase. Run the grep to verify zero results before marking
complete.

### Adding a new milestone
1. Create `docs/milestones/ms-<slug>.md` from the milestone skeleton.
2. Add a row to the active table in `docs/roadmap.md`.
No sequencing is assumed between milestones unless explicitly noted in the
milestone file's Goal section.

### Archiving milestones
Only when manually triggered (or suggested by Claude):
1. Move `docs/milestones/ms-<slug>.md` → `docs/milestones/archive/ms-<slug>.md`.
2. Move the milestone's row from `docs/roadmap.md` → `docs/milestones/archive/index.md`.

### Architecture documentation
When a decision with lasting impact is made:
1. Create or update the relevant `docs/architecture/<topic>.md`.
2. Add a row to the **Decisions Log** in the milestone file describing the
   decision and linking to the updated architecture doc.
3. Ensure the architecture doc is listed in the milestone's **Architecture
   References** section.

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

---

## Situation 2 — New milestone

When the user wants to add a milestone, invoke the
`waypoint:brainstorming` skill. It owns the full milestone-definition
flow: context research, scope assessment, capability and scope
discussion, phase decomposition, incremental validation, and writing
`docs/milestones/ms-<slug>.md` plus the row in `docs/roadmap.md`. Do
not duplicate any of those steps here.

---

## Situation 3 — Starting a phase

When the user says something like "plan ms-auth phase 2" or "start the next
phase":

1. Identify the milestone slug and phase number. Read the milestone file.
2. Verify all prior phases are `✅ Complete`. Phases are strictly sequential —
   do not plan or start phase N until phase N-1 is complete. If a prior phase
   is not complete, stop and flag it.
3. Review the previous phase's **Handoff** note in the Phases table to
   understand what this phase builds on.
4. Run the stub grep for this milestone:
   ```
   grep -r "STUB\[ms-<slug>" src/
   ```
5. Report what was found. Each hit becomes an explicit task prepended to the
   plan: locate stub → implement real logic → wire → delete comment.
5a. Acceptance Criteria review (mirrors `/waypoint:phase` step 5):
    - Read the milestone file's `## Acceptance Criteria` section.
    - Print: `ACs: N total — X ✅ / Y ⬜ / Z ❌ / W manual`.
    - Ask the user: *"Any AC changes needed before planning this
      phase? (add / edit / remove / none)"* Apply edits — new ACs get
      `verify: TODO`, `status: ⬜ untested`; material edits to a
      previously passed AC reset its status to ⬜.
    - For each AC with `verify: TODO`, ask: *"Will this phase fully
      deliver AC-N?"* If yes, add a verify-generation task to the
      plan (stack detection → strategy mapping → test placement; see
      `commands/phase.md` step 5 for the full mapping).
6. Proceed with normal planning (brainstorm → plan → execute). Order
   tasks: stub resolution first, then verify-script generation for any
   ACs assigned to this phase, then the rest of the plan.
7. After the plan is agreed: check if any decisions made during planning have
   lasting architectural impact (e.g. choice of library, data model, API
   contract, infrastructure pattern). For each that does:
   - Create or update the relevant `docs/architecture/<topic>.md`.
   - Add it to the milestone's **Architecture References** if not already listed.
   - Log the decision in the milestone's **Decisions Log**.
8. Update the phase's row in the milestone's **Phases** table:
   - Set **Design Doc** and **Plan** to their filenames/links if available.
   - Set **Status** to `🔄 In progress`.

---

## Situation 4 — Finishing a phase

When the user says a phase is done:

1. Update the phase's row in the milestone's **Phases** table:
   - Set **Design Doc** and **Plan** to their filenames/links (if still TBD).
   - Set **Status** to `✅ Complete`.
2. Check whether any new stubs were introduced during this phase:
   - Each stub must have a `// STUB[ms-<slug>/phase-N]: …` comment in code.
   - Each stub must have a row in the milestone's **Deferred Wiring** table.
   - Stubs must NOT reference other milestones' slugs.
3. If any stubs are missing from the **Deferred Wiring** table, add them now
   and flag to the user.
4. Ask: *"Were any architectural decisions made during implementation of this
   phase?"* For each yes:
   - Create or update the relevant `docs/architecture/<topic>.md`.
   - Add it to the milestone's **Architecture References** if not already listed.
   - Add a row to the milestone's **Decisions Log** with date, decision summary,
     impact, and a link to the updated doc.
   Do not skip this step — implementation often produces decisions that planning
   did not anticipate.
4a. AC verify-script presence check (mirrors `/waypoint:done` step
    4a). For any AC whose verify-generation was assigned to this
    phase, confirm the script/test file exists and the milestone
    file's `verify:` field is no longer `TODO`. If a verify is still
    `TODO`, flag the user and ask whether the AC was actually
    delivered. Do not run the verify command at phase completion —
    verification runs only at milestone completion.

---

## Situation 5 — Completing a milestone

When the user says a milestone is done:

1. Run the stub grep to verify zero stubs remain:
   ```
   grep -r "STUB\[ms-<slug>" src/
   ```
   If any results are found, **stop**. The milestone cannot be marked complete.
   Report the remaining stubs and ask the user how to proceed.
1a. Pre-flight AC check (mirrors `/waypoint:done` step 5a). Print
    `ACs: N total — X ✅ / Y ⬜ / Z ❌ / W manual`. Scan for any AC
    with `verify: TODO`. If found, STOP — list them and prompt the
    user that verify scripts must be generated before the milestone
    can close. Do not flip status.
1b. Run verifies. For each AC with `verify:` set to a real shell
    command: run sequentially, capture exit code, update `status:`
    — `✅ passed` on exit 0, `❌ failed` otherwise.
1c. Handle failures. If any AC ended `❌ failed`, STOP. Print the
    failure output, print the updated AC summary line, prompt the
    user to fix or change the AC's type to `uat` and document why.
    Do not flip status. Do not auto-retry.
1d. Manual / UAT check. For each AC with `verify: manual`: confirm
    the user has set `status: ✅ passed`. If any is still
    `⬜ untested`, STOP and prompt the user to perform the UAT.
1e. All passed. Only when every AC is `✅ passed` proceed to step 2.
2. If zero stubs: update the milestone file header — set **Status** to
   `✅ Complete` and record the completion date.
3. Update the milestone's row in `docs/roadmap.md` — set Status to
   `✅ Complete`.
4. Once a logical set of milestones is complete, suggest archiving them to
   keep the active index manageable.

---

## Situation 6 — Archiving milestones

Only when manually triggered (or after Claude suggests it and user confirms):

1. For each milestone to archive:
   - Move `docs/milestones/ms-<slug>.md` →
     `docs/milestones/archive/ms-<slug>.md`.
   - Move the row from the active table in `docs/roadmap.md` →
     `docs/milestones/archive/index.md` (append to its table).
2. Confirm with the user before moving any files.

---

## Situation 7 — Architecture documentation

When an architectural or significant implementation decision is made:

1. Determine whether it updates an existing architecture doc or requires a
   new one. Ask the user if unclear.
2. Create or update `docs/architecture/<topic>.md`:
   - New doc: use a clear title, date, and structured sections (Context,
     Decision, Rationale, Consequences).
   - Update: append a dated entry to the doc's **Change Log** section
     describing what changed and why. Do not overwrite previous decisions.
3. In the current milestone file:
   - Add or verify the doc is listed in **Architecture References**.
   - Add a row to **Decisions Log**: what was decided, what it impacts, and
     a link to the updated architecture doc.
4. This ensures historical context is preserved in the milestone file even
   after the architecture doc evolves.
