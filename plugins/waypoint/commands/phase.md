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
5. Acceptance Criteria review:
   a. Read the milestone file's `## Acceptance Criteria` section.
   b. Print a one-line summary: `ACs: N total — X ✅ / Y ⬜ / Z ❌ / W manual`.
   c. Ask: *"Based on what you've learned so far, any AC changes needed
      before planning this phase? (add / edit / remove / none)"* Apply
      edits to the milestone file. New ACs get the next free AC-N,
      `verify: TODO`, `status: ⬜ untested`. Material edits (body,
      format, type, verify command) to a previously ✅ passed AC reset
      its status to ⬜ untested.
   d. For each AC with `verify: TODO`, ask: *"Will this phase fully
      deliver AC-N: <title>?"* If yes, add an explicit
      verify-generation task to the phase plan (see step 6). If no,
      leave the AC as `verify: TODO` for a later phase.
   e. `uat` ACs (`verify: manual`) are never assigned
      verify-generation tasks.

   Stack detection for verify generation: read `docs/roadmap.md`'s
   `**Stack:**` field first; if absent or vague, use what the user has
   stated in conversation; if still unclear, ask the user with a concrete
   recommendation based on visible source files.

   Verify-strategy mapping by AC type:
   - `capability` / gherkin: BDD runner if present (cucumber, behave,
     pytest-bdd) → otherwise e2e (playwright, cypress, httpx).
   - `capability` / other formats: integration test in the project's
     primary test runner.
   - `invariant`: property test (hypothesis, fast-check, quickcheck) →
     otherwise parameterized unit test.
   - `integration`: contract test if framework exists (pact) →
     otherwise probe script against real/mock service.
   - `performance`: existing bench harness (k6, locust, wrk) →
     otherwise `time` + threshold assertion in shell.
   - `uat`: always `manual` — no inference.

   Test placement: if `tests/` or `spec/` exists, read it to infer
   conventions before generating. If no AC-style tests exist yet, ask
   the user whether to add to the existing tree or create `tests/ac/`
   (recommend `tests/ac/`). If no test infrastructure exists at all,
   generate a self-contained script at
   `tests/ac/ms-<slug>/verify_ac_<n>.sh`.

6. Brainstorm and propose a plan. Order tasks: stub resolution first,
   then verify-script generation for any ACs assigned to this phase
   (each task labelled "Generate verify script for AC-N (<title>) —
   <chosen approach>"), then the rest of the plan. Get user
   confirmation before finalizing.
7. After the plan is agreed, check for architectural decisions:
   Were any decisions made during planning that have lasting impact (e.g.
   library choice, data model, API contract, infrastructure pattern)?
   For each yes:
   - Create or update docs/architecture/<topic>.md.
     New doc: sections — Context, Decision, Rationale, Consequences.
     Existing doc: append a dated entry to its Change Log section.
   - Add the doc to the milestone's Architecture References (if not listed).
   - Add a row to the milestone's Decisions Log:
     Date | Decision | Impact | Architecture Doc Updated.
8. Update the phase row in the milestone's Phases table:
   - Set Design Doc and Plan to filenames/links (or TBD if not yet written).
   - Set Status to 🔄 In progress.
