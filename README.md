<div align="center">

<img src="./assets/logo.svg" alt="Waypoint logo" width="160" height="160">

# Waypoint

**Structured milestone and phase planning for Claude Code.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-compatible-8A4FFF.svg)](https://code.claude.com/)

</div>

## What is Waypoint?

Waypoint is a structured planning layer for Claude Code that organizes work
into **milestones** (independent units of delivery) and **sequential phases**
(ordered steps within a milestone). It gives Claude a consistent way to plan,
execute, and track project work across sessions, instead of relying on ad-hoc
notes scattered through your repo.

Three guarantees set Waypoint apart from a freeform planning doc. First,
phases within a milestone are strictly sequential — Claude refuses to plan
phase N until phase N-1 is marked complete. Second, a milestone cannot be
marked complete while any `STUB[ms-<slug>]` markers still exist in the
codebase. Third, architectural decisions are captured automatically as part
of the planning and phase-completion workflows, so your `docs/architecture/`
folder stays in sync with what was actually built.

## Key concepts

**Milestone.** An independent unit of work — usually something you could
ship, demo, or hand off on its own. Milestones do not assume any sequencing
between each other; if one depends on another, that dependency is recorded
explicitly in its goal section. Each milestone lives in a single file at
`docs/milestones/ms-<slug>.md`.

**Phase.** A sequential step within a milestone. Each phase builds on the
previous one and must be `✅ Complete` before the next begins. Good phase
boundaries are points where the system is in a coherent, testable,
committable state — not arbitrary task buckets.

**Stubs.** When implementation needs to defer a piece of logic, drop a
`// STUB[ms-<slug>/phase-N]: ...` marker at the call site **and** record it
in the milestone's Deferred Wiring table. Both steps are required. Stubs may
only reference the current milestone's slug, and a milestone cannot close
while any of its stubs remain in the code.

**Acceptance Criteria.** Each milestone declares structured acceptance
criteria (ACs) in its file — business-level success criteria in
whichever format fits each one (gherkin, user-story, decision-table,
EARS, sequence diagram, BPMN). Non-UAT criteria are verified
automatically by a `verify:` command at milestone completion; UAT
criteria require human sign-off. A milestone cannot be marked complete
while any AC is untested, failed, or has no verify script.

**Architecture docs.** Living documentation in `docs/architecture/` updated
automatically as part of phase planning and phase completion. Each decision
is logged in the milestone file's Decisions Log, with a link to the
architecture doc it created or amended — so the historical context survives
even after the architecture doc itself evolves.

## File layout

Waypoint creates and maintains this structure inside your project:

```
docs/
  roadmap.md                    # Active milestone index
  milestones/
    ms-<slug>.md                # One file per active milestone
    archive/
      index.md                  # Archived milestone index
      ms-<slug>.md              # Archived milestone files
  architecture/
    <topic>.md                  # Living architecture documentation
```

## Commands

| Command | Arguments | Description |
| :--- | :--- | :--- |
| `/waypoint:init` | — | Initialize roadmap structure in a new project |
| `/waypoint:milestone` | `[slug]` | Create a new milestone and plan its phases |
| `/waypoint:phase` | `[ms-slug] [phase#]` | Start planning a phase |
| `/waypoint:done` | `[ms-slug] [phase#]` | Mark a phase or milestone complete |
| `/waypoint:archive` | — | Archive completed milestones |

## Installation

1. Add the marketplace:

   ```
   /plugin marketplace add github:jfuechsl/waypoint
   ```

2. Install the plugin:

   ```
   /plugin install waypoint
   ```

Commands are available immediately as `/waypoint:init`,
`/waypoint:milestone`, etc.

## Getting started

A natural first run on a new project:

1. **`/waypoint:init`** — sets up `docs/roadmap.md`, `docs/milestones/`,
   `docs/milestones/archive/`, `docs/architecture/`, and appends the
   Waypoint rules block to your `CLAUDE.md`.
2. **`/waypoint:milestone auth`** — creates `docs/milestones/ms-auth.md`,
   proposes a phase breakdown for the milestone, and adds it to the
   roadmap index after you confirm.
3. **`/waypoint:phase auth 1`** — verifies prior phases are complete, runs
   the stub grep, and walks through planning phase 1.
4. **`/waypoint:done auth 1`** — marks phase 1 complete, checks for any
   new stubs that need to be tracked, prompts for any architectural
   decisions made during implementation, and (if any ACs were assigned
   to this phase) confirms the verify scripts exist.
5. Repeat for subsequent phases. Once every phase is done, run
   `/waypoint:done auth` to close out the milestone. This runs all AC
   verify commands and refuses to close if any AC is untested, failed,
   or still has `verify: TODO`. When several milestones are complete,
   run `/waypoint:archive` to move them out of the active index.

## Contributing

Issues and pull requests are welcome. If you have ideas for new situations
the skill should handle, please open an issue at
`https://github.com/jfuechsl/waypoint/issues`.

## License

MIT — see [LICENSE](./LICENSE).
