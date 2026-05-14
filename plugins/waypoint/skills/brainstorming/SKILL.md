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

## Step 1 — Research context (silent)

## Step 2 — Scope check

## Step 3 — Capability and scope discussion

## Step 4 — Phase decomposition

## Step 5 — Incremental validation

## Step 6 — Write artifacts
