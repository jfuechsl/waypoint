Create a new milestone. $ARGUMENTS may contain a slug.

1. Check `CLAUDE.md` for the heading `### Acceptance Criteria` inside
   the `## Roadmap & Stub Rules` section. If the heading is missing
   (typical for projects initialized before plugin v0.3.0), append the
   exact paragraph below to the rules block. If the heading is already
   present, do nothing — this step is idempotent. Skip silently if
   `CLAUDE.md` does not exist.

   Paragraph to append:

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

2. Invoke the `waypoint:brainstorming` skill, passing $ARGUMENTS
   through. The skill owns the full milestone-definition flow: context
   research, scope assessment, capability and scope discussion, phase
   decomposition, incremental validation (including Acceptance
   Criteria), and writing `docs/milestones/ms-<slug>.md` plus the row
   in `docs/roadmap.md`. Do not duplicate any of those steps here.
