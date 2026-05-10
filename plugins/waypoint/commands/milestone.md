Create a new milestone. $ARGUMENTS may contain a slug.

1. If no slug in $ARGUMENTS, ask for one. Also ask for a one-sentence goal
   and any known dependencies on other active milestones.
2. Based on the goal and scope, propose a logical sequential phase breakdown:
   - Each phase must leave the system in a coherent, testable, committable state.
   - Each phase has a Handoff note describing what it delivers for the next phase.
   - Explain briefly why you chose these phase boundaries.
   - Get explicit user confirmation before writing anything.
3. Create docs/milestones/ms-<slug>.md with these sections:
   - Header: Status (Not started), Created date, Completed (—)
   - Goal (note any milestone dependencies here explicitly)
   - Scope / Out of Scope
   - Done When — must include: all phases ✅ Complete AND
     `grep -r "STUB\[ms-<slug>" src/` returns zero results
   - Phases table: Phase | Focus | Key Deliverable | Handoff | Design Doc | Plan | Status
   - Architecture References (empty)
   - Decisions Log (empty)
   - Deferred Wiring (empty)
   - How to Continue in a New Session
4. Add a row to docs/roadmap.md:
   | ms-<slug> | <description> | Not started | [ms-<slug>.md](milestones/ms-<slug>.md) |
5. No sequencing is assumed between milestones. Dependencies on other
   milestones belong in the Goal section of this file only.
