---
name: brainstorming
description: >
  Define a new Waypoint milestone through capability-level dialogue.
  Invoked by /waypoint:milestone and by the waypoint skill's
  "new milestone" situation. Researches existing project state,
  assesses scope, drives a focused discussion through goal,
  beneficiary, success signal, foundation, and boundary, presents
  2-3 phase decomposition options, validates each section
  incrementally, and writes docs/milestones/ms-<slug>.md plus the
  roadmap row.
---

# Waypoint Brainstorming Skill

Owns the full milestone-definition conversation for Waypoint. Invoked
from `/waypoint:milestone` and from Situation 2 ("New milestone") of
the `waypoint:waypoint` skill. The flow is six numbered steps:
research → scope check → capability discussion → phase decomposition
→ incremental validation → write artifacts.

---

## What this skill must never do

- Ask about specific implementation technology, library choices, or
  low-level implementation details. Conceptual architecture —
  high-level components, rough data flows, system boundaries — is
  allowed when it aids scoping.
- Write the milestone file before all four definition sections
  (Goal, Scope/Out of Scope, Done When, Phases table) are individually
  approved.
- Ask more than one question per turn.
- Skip the context research step.
- Proceed with a milestone that should be decomposed into multiple
  milestones.
- Add entries to **Architecture References** or **Decisions Log** — those
  belong to phase planning and completion, not milestone definition.

## Conversation rules

- **Dialogue over interrogation.** This is a back-and-forth discussion,
  not a structured Q&A. React to what the user says, push back,
  explore implications, offer observations. The user should feel like
  they are thinking alongside the agent, not answering a form.
- **Stay on a topic until it is resolved.** Do not move to the next
  topic until the current one is clearly understood and the user has
  confirmed it — explicitly or by a response that makes it unambiguous.
  If something is ambiguous, dig in: ask a follow-up, offer a reframe,
  surface a tension. Never drop a topic mid-air to move the process
  along.
- **One open thread at a time.** Do not introduce a new topic while
  another is still unresolved. If the user raises something tangential,
  acknowledge it and park it: *"Good point — let's come back to that
  once we've settled X."*
- **YAGNI ruthlessly.** If something isn't clearly necessary for this
  milestone, push it out of scope. Default to less. Be willing to
  argue for a tighter scope if the user is over-scoping.
- **Offer structure when it helps, not by default.** Multiple choice
  options are a tool, not a template. Use them when the space of
  reasonable answers is bounded and enumerable. Avoid them when the
  question is genuinely open or when options would artificially
  constrain the user's thinking.
- **Explore alternatives for phase decomposition.** This is the one
  place where offering 2–3 structured options is always appropriate,
  because the tradeoffs between decomposition strategies are concrete
  and worth making explicit.
- **Incremental validation.** Once a topic is settled, reflect back a
  crisp summary of what was agreed and get confirmation before moving
  on. This is a checkpoint, not a form submission — the user can
  reopen anything at any time.
- **Redirect implementation drift.** If the conversation moves toward
  specific tech choices, library selection, or low-level
  implementation details, redirect: *"That's a phase planning
  decision — let's nail down what this milestone delivers first."*
  Conceptual architecture (high-level components, rough data flows,
  major system boundaries) is allowed when it genuinely helps clarify
  scope or justify phase decomposition. The test: does it help define
  *what* gets built, or is it deciding *how*? The former is fine; the
  latter is not.
- **Decompose before refining.** If the milestone sounds like multiple
  independent capabilities, flag this before going deeper on any of
  them.

## Step 1 — Research context (silent)

Before asking the user anything, read the project's existing
documentation. Do not narrate each file read.

Read in this order:

1. `docs/roadmap.md` — active milestones, status, descriptions.
2. All active `docs/milestones/ms-*.md` files — goals, scope, phase
   status, decisions log, architecture references.
3. `docs/milestones/archive/index.md` — completed milestone history.
4. Any `docs/architecture/*.md` files — existing architectural
   decisions.
5. `AGENTS.md` or `CLAUDE.md` — project working conventions if present.

After reading, produce an internal summary (do not show the user):

- What the project currently does.
- What gaps or natural next steps exist.
- Which existing milestones are relevant to what the user is about
  to define.

**Pre-init guard.** If `docs/roadmap.md` does not exist, stop
immediately and tell the user:

> *"This project isn't initialized for Waypoint yet — run
> `/waypoint:init` first, then come back."*

Do not proceed in a half-initialized state. Do not offer to run
`/waypoint:init` — that is a separate, deliberate user action.

**Slug from arguments.** If `$ARGUMENTS` carried a slug at invocation
time, note it now and use it throughout the conversation (e.g. when
referring to the milestone before it is written). If no slug was
passed, leave slug resolution for Step 5.

## Step 2 — Scope check

Open with a brief, neutral two-to-three sentence summary of what was
found in context research:

> *"I've reviewed the project. Here's where things stand: [2–3
> sentences on current state]. Now let's define the new milestone."*

Then ask the user to describe the milestone in their own words (one
open-ended question — not a multi-part one).

After their answer, apply the scope assessment. A milestone is too
large if it:

- Delivers multiple independent capabilities that could each stand
  alone.
- Would naturally be described as "X and Y and Z".
- Cannot be summarised in one crisp sentence.

If too large: do not start refining details. Help the user decompose:

> *"This sounds like it might cover [A], [B], and [C] — these could
> be separate milestones. Want to split them, or keep them together
> and explain why?"*

If the user chooses to split, ask which sub-milestone they want to
define **now**. The others are surfaced verbally and parked — they
re-enter this flow on their own later `/waypoint:milestone` run. The
current invocation focuses on the single chosen sub-milestone only.
Do not auto-create files for parked sub-milestones.

## Step 3 — Capability and scope discussion

An open dialogue, not a sequential questionnaire. The goal is to
reach shared clarity on five resolution topics — work through them
conversationally, in whatever order the discussion naturally leads:

- **Goal clarity.** What can the system or user do after this
  milestone that they can't do now? Probe until this can be stated as
  a single crisp capability sentence, not a feature list.
- **Beneficiary.** Who or what benefits — end user, developer, other
  systems? This often sharpens scope naturally.
- **Success signal.** How would you know this milestone is done if
  you came back after a month away? Push for something observable
  and verifiable.
- **Existing foundation.** Which parts of what's already built does
  this milestone build on? Cross-check against context research —
  surface connections the user may not have mentioned.
- **Scope boundary.** What's explicitly out of scope? Things that
  might seem related but won't be tackled here. When the user is
  vague, name adjacent things from context research and ask directly:
  *"In or out?"*

Treat these as discussion topics, not a checklist. Many will resolve
naturally through conversation without needing a direct question.
Stay on each topic until the answer is unambiguous and confirmed,
then move on. When a topic is settled, reflect it back briefly before
continuing:

> *"So the goal is X, and Y is explicitly out — does that capture it?"*

## Step 4 — Phase decomposition

The most important output of brainstorming. Phases must be:

- **Strictly sequential** — phase N cannot start until phase N-1 is
  complete.
- **Independently shippable** — each phase ends with the system in a
  coherent, testable, committable state.
- **Capability-level** — described by what becomes possible, not by
  how it is built.
- **Appropriately sized** — not too fine-grained (every small task)
  and not too coarse (the whole milestone in two steps).

Propose **2–3 alternative phase decompositions** and explain the
tradeoff of each. This is the one place where structured options are
mandatory — the tradeoffs are concrete and worth making explicit.
Use this shape (adapt the content to the milestone):

> *"Here are three ways to break this into phases:*
>
> *Option A — Breadth first (3 phases)*
> *Phase 1: Core capability, end-to-end but minimal*
> *Phase 2: Edge cases and error handling*
> *Phase 3: Polish and observability*
> *→ Good if you want something working quickly to validate the
> approach.*
>
> *Option B — Depth first (2 phases)*
> *Phase 1: Full implementation of the primary flow*
> *Phase 2: Secondary flows and hardening*
> *→ Good if the primary flow is well understood and unlikely to
> change.*
>
> *Option C — Foundation first (3 phases)*
> *Phase 1: Data model and core abstractions*
> *Phase 2: Business logic on top of phase 1*
> *Phase 3: Integration and exposure*
> *→ Good if other milestones will build on the same foundation."*

Ask the user to pick an option or propose a variant. Then refine the
chosen decomposition: for each phase, confirm **Focus**, **Key
Deliverable**, and **Handoff** (what the next phase can rely on).

## Step 5 — Incremental validation

Present the full milestone definition section by section. Wait for
explicit approval of each section before showing the next. If the
user wants changes, make them and re-present that section.

Present in this order, one at a time:

1. **Goal** — one crisp paragraph.
2. **Scope** (bulleted deliverables) + **Out of Scope** (bulleted
   exclusions).
3. **Done When** — verifiable criteria. Always includes both: *all
   phases complete* AND *zero `STUB[ms-<slug>]` comments remain
   (`grep -r "STUB\[ms-<slug>" src/` returns no results)*.
4. **Phases table** — all phases with **Focus**, **Key Deliverable**,
   **Handoff**.

After all four sections are approved, confirm once:

> *"Ready to write the milestone file?"*

Then proceed to Step 6.

**Slug resolution.** If `$ARGUMENTS` carried a slug at invocation
time, use it from the start. If not, derive a candidate slug from the
agreed Goal sentence at this point (before writing) and ask the user
to confirm or override. A slug coined from a vague seed phrase is
worse than one coined from a settled capability.

## Step 6 — Write artifacts

Use the existing milestone skeleton — do not duplicate it, do not
inline it. From this skill's directory, the skeleton resolves at
`../waypoint/assets/milestone_skeleton.md` (full source path:
`plugins/waypoint/skills/waypoint/assets/milestone_skeleton.md`).

Create `docs/milestones/ms-<slug>.md` from the skeleton. Fill every
section from the approved definition:

- **Header.** Status = `Not started`, Created = today's date,
  Completed = `—`.
- **Goal**, **Scope**, **Out of Scope**, **Done When** — from the
  approved definition. Done When must include the zero-stubs grep
  criterion (already in the skeleton boilerplate; verify it is
  retained).
- **Phases table.** All phases with **Focus**, **Key Deliverable**,
  **Handoff** filled in; **Design Doc** = `TBD`, **Plan** = `TBD`,
  **Status** = `Not started`.
- **Architecture References** — leave the placeholder row
  (`_(none yet)_`). Populated later during phase planning.
- **Decisions Log** — leave the placeholder row (`_(none yet)_`).
  Populated later during phase planning and completion.
- **Deferred Wiring** — leave the placeholder row (`_(none)_`).
  Populated during implementation.
- **How to Continue in a New Session** — keep the skeleton text;
  substitute `<slug>` references with the milestone slug.

Append a row to the **Active Milestones** table in `docs/roadmap.md`:

```
| ms-<slug> | <one-sentence description> | Not started | [ms-<slug>.md](milestones/ms-<slug>.md) |
```

Confirm to the user what was written and where. End with the
follow-up pointer:

> *"Milestone `ms-<slug>` written to
> `docs/milestones/ms-<slug>.md` and added to the roadmap. When
> ready, run `/waypoint:phase <slug> 1` to start planning phase 1."*
