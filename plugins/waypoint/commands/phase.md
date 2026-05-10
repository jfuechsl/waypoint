Start planning a phase. $ARGUMENTS format: [ms-slug] [phase-number]
Both are optional if the milestone and phase are unambiguous from context.

1. Read docs/roadmap.md and the relevant milestone file. Identify the milestone
   slug and phase number from $ARGUMENTS or context. If still ambiguous, ask.
2. Verify all prior phases are ✅ Complete. If any prior phase is not complete,
   stop and report which phase is blocking — do not proceed.
3. If this is not phase 1, review the previous phase's Handoff note to
   understand what this phase builds on.
4. Run the stub grep:

       grep -r "STUB\[ms-<slug>" src/

   Report all results. Each hit is a mandatory task prepended to the plan:
   locate stub → implement real logic → wire → delete comment.
5. Brainstorm and propose a plan. Stub resolution tasks come first.
   Get user confirmation before finalizing.
6. After the plan is agreed, check for architectural decisions:
   Were any decisions made during planning that have lasting impact (e.g.
   library choice, data model, API contract, infrastructure pattern)?
   For each yes:
   - Create or update docs/architecture/<topic>.md.
     New doc: sections — Context, Decision, Rationale, Consequences.
     Existing doc: append a dated entry to its Change Log section.
   - Add the doc to the milestone's Architecture References (if not listed).
   - Add a row to the milestone's Decisions Log:
     Date | Decision | Impact | Architecture Doc Updated.
7. Update the phase row in the milestone's Phases table:
   - Set Design Doc and Plan to filenames/links (or TBD if not yet written).
   - Set Status to 🔄 In progress.
