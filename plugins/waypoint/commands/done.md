Mark a phase complete. If it was the last phase, complete the milestone too.
$ARGUMENTS format: [ms-slug] [phase-number]
Both are optional if unambiguous from context.

Read the milestone file. Identify the slug and phase number.

---

## Phase completion

1. Verify the phase implementation is done (tests pass, Definition of Done met).
2. Check stubs: run `grep -r "STUB\[ms-<slug>/phase-<N>" src/`
   Every result must have a matching row in the milestone's Deferred Wiring
   table. If any are missing, add them now and flag to the user.
   Stubs must only reference this milestone's slug — flag any cross-milestone
   stubs immediately as bugs.
3. Ask: "Were any architectural decisions made during implementation of this
   phase?" For each yes:
   - Create or update docs/architecture/<topic>.md.
     New doc: sections — Context, Decision, Rationale, Consequences.
     Existing doc: append a dated entry to its Change Log section.
   - Add the doc to the milestone's Architecture References (if not listed).
   - Add a row to the milestone's Decisions Log:
     Date | Decision | Impact | Architecture Doc Updated.
   Do not skip this step — implementation often surfaces decisions that
   planning did not anticipate.
4. Update the phase row in the Phases table:
   - Set Design Doc and Plan links if still TBD.
   - Set Status to ✅ Complete.
4a. AC verify-script presence check. For any AC whose
    verify-generation was assigned to this phase, confirm the
    script/test file exists and the milestone file's `verify:` field
    is no longer `TODO`. If a verify is still `TODO`, flag the user
    and ask whether the AC was actually delivered. Do not run the
    verify command at phase completion — verification runs only at
    milestone completion.

---

## Milestone completion (last phase only)

If all phases are now ✅ Complete, proceed:

5. Run the full stub grep:

       grep -r "STUB\[ms-<slug>" src/

   If any results are found: STOP. The milestone cannot be closed.
   Report the remaining stubs and ask the user how to handle them.
6. If zero results:
   - Set Status to ✅ Complete and record the completion date in the
     milestone file header.
   - Update the milestone's row in docs/roadmap.md to ✅ Complete.
7. If a logical set of milestones is now complete, suggest archiving them
   to keep the active roadmap index manageable.
