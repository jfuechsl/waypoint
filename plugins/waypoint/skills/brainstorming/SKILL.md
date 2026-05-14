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
  roadmap row. Triggers on phrases like "new milestone",
  "add milestone", or invocation from /waypoint:milestone.
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

## Step 2 — Scope check

## Step 3 — Capability and scope discussion

## Step 4 — Phase decomposition

## Step 5 — Incremental validation

## Step 6 — Write artifacts
