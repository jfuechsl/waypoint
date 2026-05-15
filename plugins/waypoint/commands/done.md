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
5a. Pre-flight AC check. Print the AC summary line: `ACs: N total —
    X ✅ / Y ⬜ / Z ❌ / W manual`. Scan for any AC with `verify: TODO`.
    If found, STOP — list them and prompt the user that verify scripts
    must be generated (likely via a missed `/waypoint:phase` step)
    before the milestone can close. Do not flip status.
5b. Run verifies. For each AC with `verify:` set to a real shell
    command (i.e., not `TODO`, not `manual`): run the command
    sequentially, capture its exit code, and update the milestone
    file's `status:` field — `✅ passed` on exit 0, `❌ failed` on any
    other exit.
5c. Handle failures. If any AC ended `❌ failed`: STOP. Print the
    failure output for each, print the updated AC summary line, and
    prompt the user to either fix the implementation or — if the
    criterion genuinely cannot be tested automatically — change the
    AC's type to `uat` and document why. Do not flip status. Do not
    auto-retry; the user re-invokes `/waypoint:done` after fixing.
5d. Manual / UAT check. For each AC with `verify: manual`: confirm
    the user has set `status: ✅ passed`. If any is still
    `⬜ untested`, STOP and list them, prompting the user to perform
    the UAT and update status. Do not flip status.
5e. All passed. Only when every AC is `✅ passed` proceed to step 6.
6. If zero results:
   - Set Status to ✅ Complete and record the completion date in the
     milestone file header.
   - Update the milestone's row in docs/roadmap.md to ✅ Complete.
7. If a logical set of milestones is now complete, suggest archiving them
   to keep the active roadmap index manageable.
