# `waypoint:brainstorming` Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and wire up a new `waypoint:brainstorming` skill that owns
the full milestone-definition conversation, replacing the ad-hoc flows
inside `/waypoint:milestone` and Situation 2 of the existing
`waypoint:waypoint` skill.

**Architecture:** One new SKILL.md at
`plugins/waypoint/skills/brainstorming/SKILL.md`, two thin delegation
edits (one slash command, one situation inside the existing skill), and
a plugin version bump. No new assets — the new skill reads the existing
milestone skeleton at `../waypoint/assets/milestone_skeleton.md`. No code,
no tests-as-code; "tests" are shell assertions that the markdown/JSON
contains the right structural and content markers.

**Tech Stack:** Markdown (SKILL.md, command, skeleton) + JSON
(`plugin.json`). Verification via `grep`, `python3 -c 'import json'`,
and file existence checks.

**Branch:** `waypoint-brainstorming-skill` (already created and checked
out; spec lives at `docs/specs/2026-05-14-waypoint-brainstorming-design.md`).

**Reference spec:** Read it before starting — every task references it.
Path: `docs/specs/2026-05-14-waypoint-brainstorming-design.md`.

---

## File map

**Create:**
- `plugins/waypoint/skills/brainstorming/SKILL.md` — the new skill.

**Modify:**
- `plugins/waypoint/commands/milestone.md` — replace body with thin delegation.
- `plugins/waypoint/skills/waypoint/SKILL.md` — replace Situation 2 body with thin delegation.
- `plugins/waypoint/.claude-plugin/plugin.json` — bump `version` to `0.2.0`.

**Untouched (intentionally):**
- `plugins/waypoint/skills/waypoint/assets/milestone_skeleton.md`
- `plugins/waypoint/skills/waypoint/assets/roadmap_skeleton.md`
- `plugins/waypoint/skills/waypoint/assets/archive_index_skeleton.md`
- `.claude-plugin/marketplace.json`
- `README.md`

---

## Task 1: Scaffold new skill file (frontmatter + section headings)

**Files:**
- Create: `plugins/waypoint/skills/brainstorming/SKILL.md`

- [ ] **Step 1: Confirm the target directory does not exist yet**

Run:
```bash
test ! -e plugins/waypoint/skills/brainstorming && echo "absent" || echo "present"
```
Expected: `absent`

- [ ] **Step 2: Create the directory**

Run:
```bash
mkdir -p plugins/waypoint/skills/brainstorming
```
Expected: no output.

- [ ] **Step 3: Write the SKILL.md scaffold (frontmatter + headings only)**

Write file `plugins/waypoint/skills/brainstorming/SKILL.md` with exactly this content:

````markdown
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

## Conversation rules

## Step 1 — Research context (silent)

## Step 2 — Scope check

## Step 3 — Capability and scope discussion

## Step 4 — Phase decomposition

## Step 5 — Incremental validation

## Step 6 — Write artifacts
````

- [ ] **Step 4: Verify scaffold structure**

Run:
```bash
grep -c "^## " plugins/waypoint/skills/brainstorming/SKILL.md
```
Expected: `8` (one `##` for each of: Never do, Conversation rules, Step 1, Step 2, Step 3, Step 4, Step 5, Step 6).

Run:
```bash
grep -q "^name: brainstorming$" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

- [ ] **Step 5: Commit**

```bash
git add plugins/waypoint/skills/brainstorming/SKILL.md
git commit -m "Scaffold brainstorming skill (frontmatter + section headings)"
```

---

## Task 2: Fill "What this skill must never do"

**Files:**
- Modify: `plugins/waypoint/skills/brainstorming/SKILL.md`

- [ ] **Step 1: Verify the section exists and is empty**

Run:
```bash
awk '/^## What this skill must never do$/,/^## /' plugins/waypoint/skills/brainstorming/SKILL.md | sed '1d;$d' | tr -d '[:space:]' | wc -c
```
Expected: `0` (no content between this heading and the next).

- [ ] **Step 2: Replace the empty section with the full body**

Replace the line `## What this skill must never do` (and the blank lines that follow it before the next `##`) with:

````markdown
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
````

- [ ] **Step 3: Verify the section is populated**

Run:
```bash
grep -q "Ask more than one question per turn" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

Run:
```bash
grep -q "Skip the context research step" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

- [ ] **Step 4: Commit**

```bash
git add plugins/waypoint/skills/brainstorming/SKILL.md
git commit -m "Add 'What this skill must never do' to brainstorming skill"
```

---

## Task 3: Fill "Conversation rules"

**Files:**
- Modify: `plugins/waypoint/skills/brainstorming/SKILL.md`

- [ ] **Step 1: Replace the empty `## Conversation rules` section with the full body**

Replace the empty section with:

````markdown
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
````

- [ ] **Step 2: Verify content present**

Run:
```bash
grep -c "^- \*\*" plugins/waypoint/skills/brainstorming/SKILL.md
```
Expected: at least `9` (nine bolded rule names — there will be more bullets later from other sections, but at this point exactly 9 from this section plus 0 from elsewhere = 9).

Run:
```bash
grep -q "Decompose before refining" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add plugins/waypoint/skills/brainstorming/SKILL.md
git commit -m "Add conversation rules to brainstorming skill"
```

---

## Task 4: Fill "Step 1 — Research context (silent)"

**Files:**
- Modify: `plugins/waypoint/skills/brainstorming/SKILL.md`

- [ ] **Step 1: Replace the empty Step 1 section with the full body**

Replace the empty section with:

````markdown
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
````

- [ ] **Step 2: Verify pre-init guard and read order present**

Run:
```bash
grep -q "Pre-init guard" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

Run:
```bash
grep -q "docs/roadmap.md" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add plugins/waypoint/skills/brainstorming/SKILL.md
git commit -m "Add Step 1 (research context) to brainstorming skill"
```

---

## Task 5: Fill "Step 2 — Scope check"

**Files:**
- Modify: `plugins/waypoint/skills/brainstorming/SKILL.md`

- [ ] **Step 1: Replace the empty Step 2 section with the full body**

Replace the empty section with:

````markdown
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
````

- [ ] **Step 2: Verify scope assessment language present**

Run:
```bash
grep -q "Want to split them, or keep them together" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

Run:
```bash
grep -q "Cannot be summarised in one crisp sentence" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add plugins/waypoint/skills/brainstorming/SKILL.md
git commit -m "Add Step 2 (scope check) to brainstorming skill"
```

---

## Task 6: Fill "Step 3 — Capability and scope discussion"

**Files:**
- Modify: `plugins/waypoint/skills/brainstorming/SKILL.md`

- [ ] **Step 1: Replace the empty Step 3 section with the full body**

Replace the empty section with:

````markdown
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
````

- [ ] **Step 2: Verify all five topics named**

Run:
```bash
for t in "Goal clarity" "Beneficiary" "Success signal" "Existing foundation" "Scope boundary"; do
  grep -q "\*\*$t\.\*\*" plugins/waypoint/skills/brainstorming/SKILL.md \
    && echo "ok: $t" || echo "MISSING: $t"
done
```
Expected: five `ok:` lines, no `MISSING:` lines.

- [ ] **Step 3: Commit**

```bash
git add plugins/waypoint/skills/brainstorming/SKILL.md
git commit -m "Add Step 3 (capability discussion) to brainstorming skill"
```

---

## Task 7: Fill "Step 4 — Phase decomposition"

**Files:**
- Modify: `plugins/waypoint/skills/brainstorming/SKILL.md`

- [ ] **Step 1: Replace the empty Step 4 section with the full body**

Replace the empty section with:

````markdown
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
````

- [ ] **Step 2: Verify all three example options present**

Run:
```bash
for opt in "Breadth first" "Depth first" "Foundation first"; do
  grep -q "$opt" plugins/waypoint/skills/brainstorming/SKILL.md \
    && echo "ok: $opt" || echo "MISSING: $opt"
done
```
Expected: three `ok:` lines.

Run:
```bash
grep -q "Strictly sequential" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add plugins/waypoint/skills/brainstorming/SKILL.md
git commit -m "Add Step 4 (phase decomposition) to brainstorming skill"
```

---

## Task 8: Fill "Step 5 — Incremental validation"

**Files:**
- Modify: `plugins/waypoint/skills/brainstorming/SKILL.md`

- [ ] **Step 1: Replace the empty Step 5 section with the full body**

Replace the empty section with:

````markdown
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
````

- [ ] **Step 2: Verify all four validation sections named and slug rule present**

Run:
```bash
for sec in "Goal" "Scope" "Done When" "Phases table"; do
  grep -q "$sec" plugins/waypoint/skills/brainstorming/SKILL.md \
    && echo "ok: $sec" || echo "MISSING: $sec"
done
```
Expected: four `ok:` lines.

Run:
```bash
grep -q "Slug resolution" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add plugins/waypoint/skills/brainstorming/SKILL.md
git commit -m "Add Step 5 (incremental validation) to brainstorming skill"
```

---

## Task 9: Fill "Step 6 — Write artifacts"

**Files:**
- Modify: `plugins/waypoint/skills/brainstorming/SKILL.md`

- [ ] **Step 1: Replace the empty Step 6 section with the full body**

Replace the empty section with:

````markdown
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
````

- [ ] **Step 2: Verify skeleton path reference and next-step pointer present**

Run:
```bash
grep -q "../waypoint/assets/milestone_skeleton.md" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

Run:
```bash
grep -q "/waypoint:phase" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

Run:
```bash
grep -q "Active Milestones" plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add plugins/waypoint/skills/brainstorming/SKILL.md
git commit -m "Add Step 6 (write artifacts) to brainstorming skill"
```

---

## Task 10: Replace `/waypoint:milestone` command body with delegation

**Files:**
- Modify: `plugins/waypoint/commands/milestone.md`

- [ ] **Step 1: Show current content of the command (for diff awareness)**

Run:
```bash
wc -l plugins/waypoint/commands/milestone.md
```
Expected: `25` (the current file is 25 lines).

- [ ] **Step 2: Overwrite the file with the delegation body**

Replace the entire contents of `plugins/waypoint/commands/milestone.md` with exactly:

```markdown
Create a new milestone. $ARGUMENTS may contain a slug.

Invoke the `waypoint:brainstorming` skill, passing $ARGUMENTS through.
The skill owns the full milestone-definition flow: context research,
scope assessment, capability and scope discussion, phase decomposition,
incremental validation, and writing `docs/milestones/ms-<slug>.md`
plus the row in `docs/roadmap.md`. Do not duplicate any of those
steps here.
```

- [ ] **Step 3: Verify delegation present and old phase-breakdown text removed**

Run:
```bash
grep -q "waypoint:brainstorming" plugins/waypoint/commands/milestone.md && echo OK
```
Expected: `OK`

Run:
```bash
grep -q "Propose a logical sequential phase breakdown" plugins/waypoint/commands/milestone.md && echo "STALE" || echo "clean"
```
Expected: `clean` (old language is gone).

Run:
```bash
wc -l plugins/waypoint/commands/milestone.md
```
Expected: roughly `7` (significantly shorter than the previous 25).

- [ ] **Step 4: Commit**

```bash
git add plugins/waypoint/commands/milestone.md
git commit -m "Delegate /waypoint:milestone to brainstorming skill"
```

---

## Task 11: Replace Situation 2 body in `waypoint:waypoint` skill

**Files:**
- Modify: `plugins/waypoint/skills/waypoint/SKILL.md` (lines 116–133, the Situation 2 block)

- [ ] **Step 1: Confirm Situation 2 is at the expected location**

Run:
```bash
grep -n "^## Situation 2" plugins/waypoint/skills/waypoint/SKILL.md
```
Expected: `116:## Situation 2 — New milestone` (or similar — the heading line).

Run:
```bash
grep -n "^## Situation 3" plugins/waypoint/skills/waypoint/SKILL.md
```
Expected: a higher line number — the next situation, marking the end of Situation 2's body.

- [ ] **Step 2: Replace the Situation 2 block**

Replace everything from the `## Situation 2 — New milestone` heading up to (but not including) the `---` separator that precedes `## Situation 3`, with exactly:

```markdown
## Situation 2 — New milestone

When the user wants to add a milestone, invoke the
`waypoint:brainstorming` skill. It owns the full milestone-definition
flow: context research, scope assessment, capability and scope
discussion, phase decomposition, incremental validation, and writing
`docs/milestones/ms-<slug>.md` plus the row in `docs/roadmap.md`. Do
not duplicate any of those steps here.

```

(Note: keep the blank line at the end so the existing `---` separator
on the following line is preserved.)

- [ ] **Step 3: Verify delegation present and old enumeration removed**

Run:
```bash
awk '/^## Situation 2/,/^## Situation 3/' plugins/waypoint/skills/waypoint/SKILL.md | grep -q "waypoint:brainstorming" && echo OK
```
Expected: `OK`

Run:
```bash
awk '/^## Situation 2/,/^## Situation 3/' plugins/waypoint/skills/waypoint/SKILL.md | grep -q "Ask: slug, one-sentence goal" && echo "STALE" || echo "clean"
```
Expected: `clean`

Run:
```bash
awk '/^## Situation 2/,/^## Situation 3/' plugins/waypoint/skills/waypoint/SKILL.md | wc -l
```
Expected: roughly `10` lines (the heading + the new body + the `---` separator), down from ~20.

- [ ] **Step 4: Verify other situations unchanged**

Run:
```bash
grep -c "^## Situation " plugins/waypoint/skills/waypoint/SKILL.md
```
Expected: `7` (Situations 1 through 7 all still present).

- [ ] **Step 5: Commit**

```bash
git add plugins/waypoint/skills/waypoint/SKILL.md
git commit -m "Delegate Situation 2 (new milestone) to brainstorming skill"
```

---

## Task 12: Bump plugin version to 0.2.0

**Files:**
- Modify: `plugins/waypoint/.claude-plugin/plugin.json`

- [ ] **Step 1: Verify current version**

Run:
```bash
python3 -c "import json; print(json.load(open('plugins/waypoint/.claude-plugin/plugin.json'))['version'])"
```
Expected: `0.1.0`

- [ ] **Step 2: Edit the version field**

In `plugins/waypoint/.claude-plugin/plugin.json`, change:

```json
  "version": "0.1.0",
```

to:

```json
  "version": "0.2.0",
```

(Only this field. Do not touch anything else in the file.)

- [ ] **Step 3: Verify the bump and JSON validity**

Run:
```bash
python3 -c "import json; print(json.load(open('plugins/waypoint/.claude-plugin/plugin.json'))['version'])"
```
Expected: `0.2.0`

Run:
```bash
python3 -c "import json; json.load(open('plugins/waypoint/.claude-plugin/plugin.json'))" && echo "valid JSON"
```
Expected: `valid JSON`

- [ ] **Step 4: Commit**

```bash
git add plugins/waypoint/.claude-plugin/plugin.json
git commit -m "Bump plugin version to 0.2.0"
```

---

## Task 13: Final integration validation

**Files:** (read-only sweep across modified files)

No commit in this task — it is a verification pass to catch any
regression from earlier tasks. Each step is a single shell check.

- [ ] **Step 1: New skill file is present and well-formed**

Run:
```bash
test -f plugins/waypoint/skills/brainstorming/SKILL.md && echo OK
```
Expected: `OK`

Run:
```bash
grep -c "^## Step " plugins/waypoint/skills/brainstorming/SKILL.md
```
Expected: `6` (Step 1 through Step 6).

Run:
```bash
grep -c "^- \*\*" plugins/waypoint/skills/brainstorming/SKILL.md
```
Expected: at least `18` (9 conversation rules + 5 capability topics + 4 phase requirements = 18; "What this skill must never do" uses plain `- ` bullets, not `- **`).

- [ ] **Step 2: Brainstorming skill frontmatter is valid YAML-ish**

Run:
```bash
head -15 plugins/waypoint/skills/brainstorming/SKILL.md | grep -q "^name: brainstorming$" && echo OK
```
Expected: `OK`

Run:
```bash
head -15 plugins/waypoint/skills/brainstorming/SKILL.md | grep -q "^description: >$" && echo OK
```
Expected: `OK`

- [ ] **Step 3: Both entry points reference the new skill**

Run:
```bash
grep -q "waypoint:brainstorming" plugins/waypoint/commands/milestone.md && echo "command OK"
```
Expected: `command OK`

Run:
```bash
grep -q "waypoint:brainstorming" plugins/waypoint/skills/waypoint/SKILL.md && echo "skill OK"
```
Expected: `skill OK`

- [ ] **Step 4: Old per-flow language is gone**

Run:
```bash
grep -q "Propose a logical sequential phase breakdown" plugins/waypoint/commands/milestone.md && echo "STALE" || echo "clean"
```
Expected: `clean`

Run:
```bash
awk '/^## Situation 2/,/^## Situation 3/' plugins/waypoint/skills/waypoint/SKILL.md | grep -qE "(Ask: slug, one-sentence goal|Based on the goal and scope, propose)" && echo "STALE" || echo "clean"
```
Expected: `clean`

- [ ] **Step 5: Plugin version is `0.2.0` and JSON parses**

Run:
```bash
python3 -c "import json; v=json.load(open('plugins/waypoint/.claude-plugin/plugin.json'))['version']; assert v=='0.2.0', v; print('version OK')"
```
Expected: `version OK`

- [ ] **Step 6: Skeleton path resolves**

Run:
```bash
test -f plugins/waypoint/skills/waypoint/assets/milestone_skeleton.md && echo OK
```
Expected: `OK`

(This is the file the brainstorming skill points to via
`../waypoint/assets/milestone_skeleton.md` — confirming it still
exists in its original location.)

- [ ] **Step 7: Branch state summary**

Run:
```bash
git log --oneline main..HEAD | wc -l
```
Expected: `14` commits ahead of `main` — the design spec, this plan, and one commit per task (Tasks 1–12). Task 13 makes no commit.

Run:
```bash
git status
```
Expected: `nothing to commit, working tree clean`

Run:
```bash
git diff --stat main..HEAD -- plugins/ | tail -10
```
Expected (order may vary): four touched files —
- `plugins/waypoint/.claude-plugin/plugin.json` (1 insertion, 1 deletion)
- `plugins/waypoint/commands/milestone.md` (rewrite, ~7 lines)
- `plugins/waypoint/skills/brainstorming/SKILL.md` (new file)
- `plugins/waypoint/skills/waypoint/SKILL.md` (Situation 2 shrunk)

- [ ] **Step 8: No commit**

Validation pass only. Stop here and report results to the user.

---

## Acceptance criteria (mirrored from spec)

- `plugins/waypoint/skills/brainstorming/SKILL.md` exists and is
  discoverable as `waypoint:brainstorming`.
- `/waypoint:milestone` invocation drives through the new
  brainstorming flow end-to-end.
- "New milestone" or "add milestone" in chat routes through
  Situation 2, which delegates to `waypoint:brainstorming`.
- The skill refuses to proceed if `docs/roadmap.md` does not exist,
  pointing the user to `/waypoint:init`.
- The skill never writes the milestone file before all four
  definition sections (Goal, Scope/Out of Scope, Done When, Phases)
  have been individually approved.
- `plugin.json` reflects version `0.2.0`.
