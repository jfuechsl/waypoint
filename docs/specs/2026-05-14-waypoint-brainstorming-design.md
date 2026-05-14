# Design: `waypoint:brainstorming` skill

**Date:** 2026-05-14
**Branch:** `waypoint-brainstorming-skill`
**Plugin version target:** `0.2.0`

---

## Purpose

Add a dedicated `waypoint:brainstorming` skill that owns the full
milestone-definition conversation. It is invoked from two entry points:

1. The `/waypoint:milestone` slash command.
2. The existing `waypoint:waypoint` skill's Situation 2 ("New milestone"),
   which is triggered by natural-language phrases like "new milestone" or
   "add milestone".

Both entry points reduce to a single instruction: *invoke
`waypoint:brainstorming`*. The new skill handles research, scope
assessment, capability discussion, phase decomposition, incremental
validation, and writing the milestone file plus the roadmap row.

This replaces the current behaviour, where `/waypoint:milestone` and
Situation 2 each ran their own (more abrupt) flow, producing inconsistent
results depending on entry point.

---

## File-level summary

### Created

- `plugins/waypoint/skills/brainstorming/SKILL.md` — the new skill.

### Modified

- `plugins/waypoint/commands/milestone.md` — reduced to a thin
  delegation to the brainstorming skill, passing `$ARGUMENTS` through.
- `plugins/waypoint/skills/waypoint/SKILL.md` — Situation 2 reduced to
  a one-paragraph delegation; trigger phrases in the skill description
  remain so chat-language entry still routes here.
- `plugins/waypoint/.claude-plugin/plugin.json` — version bump from
  `0.1.0` to `0.2.0`.

### Unchanged

- `plugins/waypoint/skills/waypoint/assets/milestone_skeleton.md` — the
  new skill reads from this existing path; no duplication.
- `plugins/waypoint/skills/waypoint/assets/roadmap_skeleton.md` — not
  touched by this skill (it is only used at init time).
- `.claude-plugin/marketplace.json` — already has no version field;
  nothing to change.
- `README.md` — already lists `/waypoint:milestone` correctly and does
  not describe the internal flow; no edit needed.

---

## Skill placement and discovery

- **Path:** `plugins/waypoint/skills/brainstorming/SKILL.md`.
- **Invokable as:** `waypoint:brainstorming`.
- **Discovery:** Claude Code auto-discovers any folder under
  `plugins/<plugin>/skills/` that contains a `SKILL.md`. No manifest
  edit required.
- **Why not nested under the existing skill:** Claude Code's discovery
  model is one folder per skill — there is no nesting. The spec's path
  hint (`skills/waypoint/brainstorming/SKILL.md`) was interpreted as a
  namespace shorthand, not a literal filesystem path. The realised path
  is the sibling form above.

---

## Skill behaviour (the conversational flow)

The skill body is laid out as six numbered steps. The original spec
jumped from Step 3 to Step 5; the numbering is fixed here.

### Step 1 — Research context (silent)

Before asking the user anything, the skill reads existing project
documentation, in order:

1. `docs/roadmap.md` — active milestones, status, descriptions.
2. All active `docs/milestones/ms-*.md` files — goals, scope, phase
   status, decisions log, architecture references.
3. `docs/milestones/archive/index.md` — completed milestone history.
4. Any `docs/architecture/*.md` files — existing architectural
   decisions.
5. `AGENTS.md` or `CLAUDE.md` — project working conventions if present.

No narration of each file read. After reading, produce an internal
summary (not shown to the user) of:

- What the project currently does.
- What gaps or natural next steps exist.
- Which existing milestones are relevant to what the user is about to
  define.

**Pre-init guard.** If `docs/roadmap.md` does not exist, stop
immediately and tell the user: *"This project isn't initialized for
Waypoint yet — run `/waypoint:init` first, then come back."* Do not
proceed in a half-initialized state.

### Step 2 — Scope check

Open with a brief, neutral two-to-three-sentence summary of project
state, then ask the user — in their own words — to describe the
milestone they want to define. One open-ended question.

After the user answers, apply the scope assessment. A milestone is too
large if it:

- Delivers multiple independent capabilities that could each stand
  alone.
- Would naturally be described as "X and Y and Z".
- Cannot be summarised in one crisp sentence.

If too large: do not start refining details. Propose decomposition —
*"This sounds like [A], [B], [C] — split or keep together?"* If the
user splits, ask which sub-milestone they want to define **now**. The
others are mentioned and parked; they re-enter this flow on their own
later `/waypoint:milestone` invocation. The current run focuses on the
single chosen sub-milestone.

### Step 3 — Capability and scope discussion

An open dialogue, not a sequential questionnaire. The goal is shared
clarity on five resolution topics — work through them conversationally,
in whatever order the discussion naturally leads:

- **Goal clarity.** What can the system or user do after this milestone
  that they can't do now? Probe until this can be stated as a single
  crisp capability sentence, not a feature list.
- **Beneficiary.** Who or what benefits — end user, developer, other
  systems? This often sharpens scope naturally.
- **Success signal.** How would you know this milestone is done if you
  came back after a month away? Push for something observable and
  verifiable.
- **Existing foundation.** Which parts of what's already built does
  this milestone build on? Cross-check against context research —
  surface connections the user may not have mentioned.
- **Scope boundary.** What is explicitly out of scope? Things that
  might seem related but won't be tackled here. When the user is
  vague, name adjacent things from context research and ask:
  *"In or out?"*

Treat these as discussion topics, not a checklist. Many will resolve
through conversation without a direct question. Stay on each until the
answer is unambiguous and confirmed, then move on. When a topic is
settled, reflect it back briefly before continuing.

### Step 4 — Phase decomposition

Phases must be:

- **Strictly sequential** — phase N cannot start until phase N-1 is
  complete.
- **Independently shippable** — each phase ends with the system in a
  coherent, testable, committable state.
- **Capability-level** — described by what becomes possible, not how
  it is built.
- **Appropriately sized** — not every small task; not the whole
  milestone in two steps.

Propose **2–3 alternative phase decompositions** and explain the
tradeoff of each. This is the one place where structured options are
mandatory, because the tradeoffs are concrete and worth surfacing.
Example format (use the same shape, adapt the content):

> *"Here are three ways to break this into phases:*
>
> *Option A — Breadth first (3 phases)*
> *Phase 1: Core capability, end-to-end but minimal*
> *Phase 2: Edge cases and error handling*
> *Phase 3: Polish and observability*
> *→ Good if you want something working quickly to validate the approach.*
>
> *Option B — Depth first (2 phases)*
> *Phase 1: Full implementation of the primary flow*
> *Phase 2: Secondary flows and hardening*
> *→ Good if the primary flow is well understood and unlikely to change.*
>
> *Option C — Foundation first (3 phases)*
> *Phase 1: Data model and core abstractions*
> *Phase 2: Business logic on top of phase 1*
> *Phase 3: Integration and exposure*
> *→ Good if other milestones will build on the same foundation."*

Ask the user to pick an option or propose a variant. Then refine the
chosen decomposition: for each phase, confirm Focus, Key Deliverable,
and Handoff (what the next phase can rely on).

### Step 5 — Incremental validation

Present the milestone definition section by section. Wait for explicit
approval of each section before showing the next. If the user wants
changes, make them and re-present that section.

Order:

1. **Goal** (one crisp paragraph).
2. **Scope** (bulleted deliverables) + **Out of Scope** (bulleted
   exclusions).
3. **Done When** (verifiable criteria, including the zero-stubs grep).
4. **Phases table** (all phases with Focus, Key Deliverable, Handoff).

After all four sections are approved, ask once: *"Ready to write the
milestone file?"* Then proceed.

### Step 6 — Write artifacts

Create `docs/milestones/ms-<slug>.md` from the existing skeleton. The
brainstorming skill references it by path relative to its own
`SKILL.md` — `../waypoint/assets/milestone_skeleton.md` — which
resolves to `plugins/waypoint/skills/waypoint/assets/milestone_skeleton.md`
in the plugin source tree. No duplication, no symlink, no inlined
copy. Fill every section from the approved definition:

- **Header.** Status = `Not started`, Created = today's date,
  Completed = `—`.
- **Goal**, **Scope**, **Out of Scope**, **Done When** — from the
  approved definition.
- **Phases table.** All phases with Focus, Key Deliverable, Handoff
  filled in; Design Doc = `TBD`, Plan = `TBD`, Status = `Not started`.
- **Architecture References** — empty (populated during phase planning).
- **Decisions Log** — empty (populated during phase planning and
  completion).
- **Deferred Wiring** — empty (populated during implementation).
- **How to Continue in a New Session** — standard skeleton text.

Append a row to the **Active Milestones** table in `docs/roadmap.md`:

```
| ms-<slug> | <one-sentence description> | Not started | [ms-<slug>.md](milestones/ms-<slug>.md) |
```

Confirm to the user what was written and where. Remind them to run
`/waypoint:phase <slug> 1` when ready to start the first phase.

---

## Slug handling

- If `$ARGUMENTS` carries a slug at invocation time, use it from the
  start.
- If not, propose a slug derived from the agreed capability sentence
  during Step 5 (before writing) and ask the user to confirm or
  override. Rationale: a slug coined from a vague seed phrase is worse
  than one coined from a settled capability.

---

## Conversation rules (preserved from the spec)

These are encoded as bullets inside the new SKILL.md so they remain
salient at runtime:

- **Dialogue over interrogation.** Genuine back-and-forth, not Q&A.
  React, push back, explore implications, offer observations.
- **Stay on a topic until resolved.** Don't move on until the current
  topic is clearly understood and confirmed.
- **One open thread at a time.** Park tangents explicitly.
- **YAGNI ruthlessly.** Default to less. Be willing to argue for a
  tighter scope.
- **Offer structure when it helps, not by default.** Multiple choice
  is a tool, not a template — use only when the answer space is
  bounded and enumerable.
- **Explore alternatives for phase decomposition.** Always 2–3
  options with tradeoffs.
- **Incremental validation.** Reflect back crisp summaries; let the
  user reopen anything at any time.
- **Redirect implementation drift.** If the conversation drifts to
  specific tech, library choice, or low-level implementation:
  *"That's a phase planning decision — let's nail down what this
  milestone delivers first."* High-level conceptual architecture
  (components, rough data flows, system boundaries) is allowed when
  it genuinely helps clarify scope.
- **Decompose before refining.** If the request sounds like several
  independent capabilities, flag this before going deeper.

---

## What this skill must never do

(Verbatim from the original spec, reproduced inside SKILL.md for
runtime salience.)

- Ask about specific implementation technology, library choices, or
  low-level implementation details. Conceptual architecture — high-
  level components, rough data flows, system boundaries — is allowed
  when it aids scoping.
- Write the milestone file before all four definition sections are
  approved.
- Ask more than one question per turn.
- Skip the context research step.
- Proceed with a milestone that should be decomposed into multiple
  milestones.
- Add entries to Architecture References or Decisions Log (those
  belong to phase planning and completion, not milestone definition).

---

## Edits to existing files in detail

### `plugins/waypoint/commands/milestone.md`

Full body replaced with:

```
Create a new milestone. $ARGUMENTS may contain a slug.

Invoke the `waypoint:brainstorming` skill, passing $ARGUMENTS through.
The skill handles research, scope assessment, capability discussion,
phase decomposition, validation, and writing the milestone file and
roadmap row. Do not duplicate any of those steps here.
```

### `plugins/waypoint/skills/waypoint/SKILL.md` — Situation 2

Replace the existing Situation 2 body with:

```
## Situation 2 — New milestone

When the user wants to add a milestone, invoke the
`waypoint:brainstorming` skill. It owns the full milestone-definition
flow: context research, scope assessment, capability discussion,
phase decomposition, incremental validation, and writing
`docs/milestones/ms-<slug>.md` plus the `docs/roadmap.md` row. Do not
duplicate any of those steps here.
```

The skill's frontmatter `description` keeps its current trigger
phrases (including `"new milestone"` and `"add milestone"`) so
chat-language entry still routes here, then delegates.

### `plugins/waypoint/.claude-plugin/plugin.json`

```diff
-  "version": "0.1.0",
+  "version": "0.2.0",
```

No other manifest changes.

---

## New skill file outline

`plugins/waypoint/skills/brainstorming/SKILL.md` — single file, no
assets. Approximate structure:

```
---
name: brainstorming
description: >
  Define a new Waypoint milestone through capability-level dialogue.
  Invoked by /waypoint:milestone and by the waypoint skill's
  "new milestone" situation. Researches existing project state,
  assesses scope, drives a focused discussion through goal/
  beneficiary/success-signal/foundation/boundary, presents 2-3
  phase decomposition options, validates each section
  incrementally, and writes docs/milestones/ms-<slug>.md plus the
  roadmap row.
---

# Waypoint Brainstorming Skill

## When to use
[invoked by /waypoint:milestone and Situation 2 of waypoint:waypoint]

## What this skill must never do
[bulleted list — verbatim from this spec]

## Conversation rules
[bulleted list — verbatim from this spec]

## Step 1 — Research context (silent)
[read order, pre-init guard, internal summary instructions]

## Step 2 — Scope check
[neutral summary, open question, decomposition path]

## Step 3 — Capability and scope discussion
[five resolution topics as conversational discussion]

## Step 4 — Phase decomposition
[mandatory 2-3 alternatives template, refinement after choice]

## Step 5 — Incremental validation
[four sections, approval gate between each]

## Step 6 — Write artifacts
[milestone file from skeleton path, roadmap row, slug rule,
 confirmation + next-step pointer]
```

---

## Out of scope

- Changes to `/waypoint:phase`, `/waypoint:done`, `/waypoint:archive`,
  or `/waypoint:init`.
- Changes to the milestone or roadmap skeletons.
- Adding new triggers, situations, or behaviours not described above.
- Splitting `waypoint:waypoint` further (e.g. extracting Situation 3
  into a `phase` skill). That could follow the same pattern in a
  later milestone but is not part of this work.
- Auto-creating sibling milestones when the user splits an
  over-scoped request — only the chosen sub-milestone is written;
  others are surfaced verbally and parked for a later run.

---

## Acceptance criteria

- `plugins/waypoint/skills/brainstorming/SKILL.md` exists and is
  discoverable as `waypoint:brainstorming`.
- `/waypoint:milestone` invocation drives through the new
  brainstorming flow end-to-end, producing a valid
  `docs/milestones/ms-<slug>.md` and a new row in `docs/roadmap.md`.
- Saying "new milestone" or "add milestone" in chat routes through
  Situation 2 of `waypoint:waypoint`, which delegates to
  `waypoint:brainstorming` — i.e. the same flow runs.
- The skill refuses to proceed if `docs/roadmap.md` does not exist,
  pointing the user to `/waypoint:init`.
- The skill never writes the milestone file before all four
  definition sections (Goal, Scope/Out-of-Scope, Done When, Phases)
  have been individually approved.
- `plugin.json` reflects version `0.2.0`.
