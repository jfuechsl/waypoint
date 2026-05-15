# ms-<slug> — <Milestone Name>

**Status:** Not started
**Created:** <date>
**Completed:** —

---

## Goal

<What this milestone achieves and why it matters. If it depends on another
milestone, note it here explicitly.>

## Scope

- <deliverable>
- <deliverable>

## Out of Scope

- <thing deferred to ms-other or future work>

## Done When

<Concrete, verifiable completion criteria.>

- All phases complete.
- Zero `STUB[ms-<slug>]` comments remain in the codebase
  (`grep -r "STUB\[ms-<slug>" src/` returns no results).
- All Acceptance Criteria are ✅ passed (see the Acceptance Criteria section).

---

## Acceptance Criteria

> Business-level success criteria for this milestone. Each criterion is
> verified automatically where possible; `uat` criteria require human
> sign-off. A milestone cannot be marked complete while any criterion is
> ⬜ untested or ❌ failed (see CLAUDE.md rules).

_(none yet — populated during milestone definition)_

---

## Phases

> Phases are strictly sequential. Phase N must be ✅ Complete before phase N+1
> begins. Each phase's Handoff describes what it delivers for the next phase.

| Phase | Focus | Key Deliverable | Handoff | Design Doc | Plan | Status |
|---|---|---|---|---|---|---|
| 1 | <focus> | <deliverable> | <what phase 2 can rely on> | TBD | TBD | Not started |
| 2 | <focus> | <deliverable> | <what phase 3 can rely on / final state> | TBD | TBD | Not started |

---

## Architecture References

> Architecture docs relevant to this milestone.
> Update this list when a new doc is created or an existing one is updated
> as a result of decisions made here.

| Doc | Relevance |
|---|---|
| _(none yet)_ | |

---

## Decisions Log

> Architectural and significant implementation decisions made during this
> milestone. Each entry should capture enough context to understand why the
> decision was made, even after the milestone is archived.

| Date | Decision | Impact | Architecture Doc Updated |
|---|---|---|---|
| _(none yet)_ | | | |

---

## Deferred Wiring

> Stubs introduced within this milestone's phases.
> ALL rows must be resolved before this milestone can be marked complete.
> Stubs must be scoped to this milestone's slug only: `STUB[ms-<slug>/phase-N]`.

| Location | STUB Tag | Description | Resolve in Phase |
|---|---|---|---|
| _(none)_ | | | |

---

## How to Continue in a New Session

1. Read `docs/roadmap.md` for the project overview.
2. Read this file (`docs/milestones/ms-<slug>.md`).
3. Read the latest completed phase's plan document (see Phases table).
4. Read `AGENTS.md` for working-tree policy and Definition of Done.
5. Run `grep -r "STUB\[ms-<slug>" src/` and review open stubs.
6. Ask the user to confirm which phase to work on next.
