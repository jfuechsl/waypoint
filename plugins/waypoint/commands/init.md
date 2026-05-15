Initialize the roadmap structure for this project.

1. Ask for project name and stack if not already clear from context.
2. Ask the user to describe the milestones they have in mind (rough is fine).
3. Create the directory structure:
   - docs/milestones/
   - docs/milestones/archive/
   - docs/architecture/
4. Create docs/roadmap.md as the active milestone index with columns:
   Slug | Description | Status | File
5. For each milestone described:
   - Propose a sequential phase breakdown. Each phase must leave the system in
     a coherent, testable, committable state. Include a Handoff note per phase
     describing what it delivers for the next phase.
   - Confirm the phase breakdown with the user before writing.
   - Create docs/milestones/ms-<slug>.md with: Goal, Scope, Out of Scope,
     Done When (must include zero-stub grep as a criterion), Phases table
     (Phase | Focus | Key Deliverable | Handoff | Design Doc | Plan | Status),
     Architecture References, Decisions Log, Deferred Wiring, How to Continue.
   - Add a row to docs/roadmap.md.
6. Create docs/milestones/archive/index.md with an empty archived milestone
   table: Slug | Description | Completed | File.
7. Append the following rules block to CLAUDE.md (create if missing):

---
## Roadmap & Stub Rules

### Phase sequencing
Phases within a milestone are strictly sequential. Do not start phase N until
phase N-1 is ✅ Complete. Review the prior phase's Handoff note before planning.

### Starting any milestone phase
Before planning, grep for open stubs scoped to this milestone:

    grep -r "STUB\[ms-<slug>" src/

Each hit is a mandatory task: locate → implement → wire → delete comment.
Never begin planning without running this grep first.

### Deferring logic during implementation
Both steps are required — doing only one is a bug.
1. Add an inline comment: `// STUB[ms-<slug>/phase-N]: description`
2. Add a row to the Deferred Wiring table in docs/milestones/ms-<slug>.md.
Stubs must only reference the current milestone's slug. Cross-milestone stubs
are not allowed.

### Architecture documentation
When a decision with lasting impact is made (during planning or implementation):
1. Create or update docs/architecture/<topic>.md.
2. Log the decision in the milestone's Decisions Log with a link to the doc.
3. Ensure the doc appears in the milestone's Architecture References.

### Completing a milestone
A milestone CANNOT be marked complete while any STUB[ms-<slug>] comment exists.
Run the grep to verify zero results before closing.

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
---

8. Confirm with the user before writing any files.
