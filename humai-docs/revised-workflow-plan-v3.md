# Human-Led AI Development Workflow

**Status:** Revised workflow, version 3  
**Primary objective:** Keep the developer in control of implementation while using AI for completion, planning clarification, contextual discussion, and teaching—calibrated to the developer’s documented level of understanding.

**Changes from v2:** Integrates a committed `humai-teach/` tree (Matt Pocock’s teach skill) as soft tutor context. Learning is never a hard gate on implementation. Planner curates a per-plan learning map, including relevant completed lessons for refresh; Pair dials teaching intensity from that map and `learning-records/`; Finalize may recommend lessons or records when gaps appear.

---

## 1. Executive summary

This workflow deliberately avoids autonomous AI implementation as its default.

The development loop is:

```text
rough-plan.md
→ grilling session (Planner)
  (may scaffold humai-teach/<subject>/ shells + learning map in refined-plan)
→ refined-plan.md
→ human-led implementation with an AI pair programmer
  (optional formal teach sessions in humai-teach/<subject>/)
→ Finalize (outcome + archive; may recommend lessons / learning-records)
```

A lighter path exists for small work: drop `work-notes.md` directly into `humai-plans/implementing/` and use the same Pair agent. Light path may include a tiny optional learning blurb; Planner does not scaffold `humai-teach/` unless the work is later promoted to a full plan. Finalize may still flag knowledge gaps.

### Roles and hosts

| Role | Host | Responsibility |
|---|---|---|
| **Autocomplete / inline assist** | Zed | Predicts or answers about code the developer is already writing |
| **Planner** | OpenCode | Challenges a rough plan, resolves ambiguity, produces a refined plan; may update domain docs; scaffolds teach subject shells and writes the plan’s learning map, including relevant completed lessons for refresh |
| **Pair programmer** | OpenCode | Reads project and current work, teaches, hints, redirects; calibrates depth from the plan’s learning map and linked learning-records; may refresh from linked completed lessons; does not edit production code or `humai-teach/` |
| **Teach sessions** | OpenCode (teach skill) | Formal lessons, glossaries, and learning-records inside `humai-teach/<subject>/` (developer invokes after `cd` into that subject) |
| **Finalize** | OpenCode | Verifies completion with the developer, records outcome, archives the plan; may recommend lessons/learning-records; may append a learning-record only on explicit developer confirmation |
| **Models** | ChatGPT subscription | Used from Zed and OpenCode |

Durable state lives in repository files—not in AI chat history.

Main sources of context:

```text
AGENTS.md
active plan under humai-plans/implementing/
  (including its learning map, when present)
CONTEXT.md
relevant ADRs
humai-teach/<subject>/ via links in the active plan
  (especially learning-records/ and relevant completed lessons)
current code
recent changes or Git diff
lint and test output
```

Sessions may be discarded and restarted at any time without losing the project’s actual state.

**North star:** Agents suggest the next helpful thing and adjust help to the developer’s level. The developer freely chooses what to do next—like a personal tutor on call, not a locked curriculum.

---

## 2. Core decisions

### 2.1 Lightweight Markdown plans

Ordinary Markdown plans—not a specification framework.

- A **rough plan** captures the developer’s initial thinking.
- A **refined plan** is the agreement used during full-lifecycle implementation.
- **Work notes** support a light path without grilling or templates.

```text
humai-plans/
├── drafts/
├── grilling/
├── refined/
├── implementing/
├── done/
└── cancelled/
```

No generated manifests, schemas, or lifecycle tooling are required for v1.

### 2.2 Folder location is the authoritative status

Do not maintain a separate status dashboard.

| Location | Meaning | Cardinality |
|---|---|---|
| `humai-plans/drafts/` | Rough ideas being formed | Many allowed |
| `humai-plans/grilling/` | Actively being grilled | At most one |
| `humai-plans/refined/` | Grilling done; awaiting human review | At most one |
| `humai-plans/implementing/` | Accepted and in active implementation (Pair’s focus) | At most one |
| `humai-plans/done/` | Completed and verified | Many (organize by year) |
| `humai-plans/cancelled/` | Deliberately abandoned | Many (organize by year) |

### 2.3 Exactly one active implementation

`humai-plans/implementing/` contains no more than one plan directory.

Pair must behave as follows:

- If there is one plan under `implementing/`, use it.
- If there is none, say so before making plan-specific assumptions.
- If there is more than one, report the ambiguity.
- Accept either `refined-plan.md` (full path) or `work-notes.md` (light path).
- If neither is present, report that required context is missing.

Mention **materially missing context** only—not every theoretically useful piece of information.

### 2.4 Who moves plans

| Transition | Who |
|---|---|
| `drafts` → `grilling` | Developer, when starting Planner |
| Write `refined-plan.md` while in `grilling/` | Planner (docs only) |
| `grilling` → `refined` | Developer, when grilling is done and review should be cold |
| `refined` → `implementing` | Developer, when accepting the plan and coding starts |
| Create light work directly in `implementing/` | Developer (skips earlier stages) |
| `implementing` → `done` / `cancelled` | Finalize, only after explicit developer approval |

Agents may remind the developer that a move is appropriate. They must not silently relocate plans.

Mid-lifecycle moves are manual in v1. Deterministic scripts (`plan start`, `plan done`, …) may be added later if friction appears.

**Learning is not a lifecycle gate.** Missing lessons or learning-records never block `refined` → `implementing`. They only change how aggressively Pair (and suggestions from Finalize) teach.

### 2.5 ADRs are exceptional

Create an Architecture Decision Record only when all are true:

1. Meaningfully expensive to reverse.
2. A future developer would reasonably wonder why it was chosen.
3. Real alternatives and trade-offs were considered.

`CONTEXT.md` is a domain glossary, not an implementation plan. Normal implementation choices stay in the refined plan, work notes, or code.

### 2.6 Teaching lives under `humai-teach/` (soft tutor context)

Committed in the same project repository:

```text
humai-teach/
├── README.md                 # invocation rules (cd into subject before /teach)
├── c-language/               # example subject = full teach workspace
│   ├── MISSION.md
│   ├── RESOURCES.md
│   ├── NOTES.md
│   ├── GLOSSARY.md           # craft terms for this subject (when earned)
│   ├── learning-records/
│   ├── lessons/
│   ├── reference/
│   └── assets/
├── raylib/
└── …
```

Rules:

- Each `humai-teach/<subject>/` is a **full teach-skill workspace** (one mission per workspace).
- **Subject grain:** broad craft buckets (`c-language`, `raylib`, `game-architecture`). Missions and out-of-scope inside each keep focus on what *this project* needs next. Missions **may expand** as the project demands new skills in that craft. Split a new subject only when the craft clearly differs.
- **Proof of level:** `learning-records/` are the durable signal (demonstrated understanding or stated prior knowledge—not mere lesson coverage).
- **Completed lessons:** file existence alone does not establish completion. Planner links a lesson as completed only when the developer confirms it or a teaching record explicitly establishes completion. These links support refresh; they are not proof of level.
- **No global teach dashboard.** The per-plan learning map in `refined-plan.md` (or a light blurb in `work-notes.md`) is the handoff artifact. Subject folders accumulate curriculum over time.
- **Invocation:** always `cd humai-teach/<subject>` (or open that folder as the workspace) before running the teach skill. Running teach from the repo root will mistreat the project root as the teaching workspace.
- **`CONTEXT.md` vs teach glossaries:** strict split. `CONTEXT.md` = *this product’s* language (entities, rules, mechanics). `humai-teach/<subject>/GLOSSARY.md` (and reference docs) = *general craft* (language, library, patterns). Do not copy teach glossaries into `CONTEXT.md`. Rarely promote a term into `CONTEXT.md` only when it has become project domain language.

---

## 3. Repository structure

```text
/
├── AGENTS.md
├── CONTEXT.md
│
├── humai-plans/
│   ├── README.md
│   ├── _templates/
│   │   ├── rough-plan.md
│   │   └── refined-plan.md
│   │
│   ├── drafts/
│   ├── grilling/
│   ├── refined/
│   ├── implementing/
│   ├── done/
│   │   └── 2026/
│   └── cancelled/
│       └── 2026/
│
├── humai-teach/
│   ├── README.md
│   └── <subject>/            # teach-skill workspaces (committed)
│
├── humai-docs/
│   ├── adr/
│   └── revised-workflow-plan-v3.md
│
└── .opencode/                    # or equivalent OpenCode config locations
    ├── agents/
    │   ├── pair.md
    │   ├── planner.md
    │   └── finalize.md
    └── (commands / prompts as needed)
```

Skills such as `grill-with-docs` and `teach` live where OpenCode discovers them (for example `.agents/skills/` or Claude-compatible skill paths). The teach skill may be installed globally; subject state still lives in this repo under `humai-teach/<subject>/`.

Preserve `rough-plan.md` through the lifecycle when it exists. `refined-plan.md` is authoritative for full-path work; `work-notes.md` is authoritative for light-path work.

---

## 4. Plan templates

### 4.1 Rough plan (short)

Save as `humai-plans/_templates/rough-plan.md`.

```markdown
---
title:
created:
author:
---

# Rough plan: <title>

## Problem

What problem am I trying to solve?

## Desired outcome

What should become possible or behave differently?

## Initial approach

How do I currently think I might implement this?
This is a hypothesis, not a commitment.

## Risks and uncertainties

What am I unsure about?

## Learning focus (optional)

What do I want to learn or take slowly?
Familiarity claims are welcome even if incomplete
(e.g. "I know structs; I do not know pointers well").
You may not know what you do not know—Planner will expand this.

## Questions for grilling

-
```

Keep it short. Omit long “affected areas” checklists unless already known. Uncertainty belongs in the document.

### 4.2 Refined plan (layered)

Save as `humai-plans/_templates/refined-plan.md`.

A human should be able to re-read the **must-read** block in about five minutes. Prefer links to code, `CONTEXT.md`, and ADRs over pasted explanations.

```markdown
---
title:
created:
refined:
branch:
---

# Refined plan: <title>

<!-- MUST READ -->

## Objective

A concise description of the result this work must produce.

## Desired behaviour

Observable behaviour and important scenarios.

## Scope

What this implementation includes.

## Out of scope

What this implementation deliberately excludes.

## Decisions

### Decision: <name>

**Choice:**

**Reason:**

**Alternatives considered:**

**Consequences:**

Repeat only for decisions important to implementing this feature.

## Acceptance criteria

- [ ]

<!-- WORKING -->

## Implementation slices

### Slice 1: <small vertical result>

- [ ] Implementation
- [ ] Relevant tests
- [ ] Manual or runtime verification

### Slice 2: <small vertical result>

- [ ] Implementation
- [ ] Relevant tests
- [ ] Manual or runtime verification

<!-- REFERENCE / SKIM -->

## Context

How the relevant part of the system currently works.
Link to code, CONTEXT.md terms, and ADRs instead of duplicating them.

## Risks

| Risk | Mitigation |
|---|---|
| | |

## Open questions

No unresolved blocking questions when implementation begins.
Non-blocking uncertainties may remain here explicitly.

## Possible ADRs

List decisions that may warrant an ADR. Do not create one automatically.

## Learning map

Planner-authored handoff for Pair. Keep short—no syllabus dump.
Not a gate; implementation may proceed regardless.

### Familiarity (optional)

Developer claims and depth, e.g. high / partial / none for named topics.
Enough for Pair to dial down immediately even before records exist.

### Subjects

Links to `humai-teach/<subject>/` workspaces relevant to this plan.

### Relevant completed lessons

Direct links to existing lessons the developer has completed that are relevant and useful to refresh for this plan, or `None`.
Completion must be developer-confirmed or explicit in a teaching record; file existence alone is not completion or proof of proficiency.

### Relevant learning-records

Links to existing records Pair should treat as floor for teaching intensity.

### Suggested lessons or topics (optional)

Plan-scoped next steps (links into `lessons/` when they exist, or topic names for a future teach session).

### In-flight learning

If the developer prefers to learn with Pair during implementation, say so explicitly here.

## Outcome

Complete this section during Finalize before archiving.

### Result

What was implemented?

### Deviations

How did the implementation differ from the plan, and why?

### Verification

Which tests, checks, or manual scenarios confirmed the result?

### Follow-up work

What was intentionally left for later?

### Learning follow-ups (optional)

Lessons or learning-records Finalize recommends after this work.
```

Implementation slices should be small, observable vertical results—not layers such as “create types,” then “create service,” then “create UI.”

### 4.3 Light path: `work-notes.md`

No template required. Create a slug under `humai-plans/implementing/` and add a freeform `work-notes.md`.

Suggested shape (optional, not enforced):

```markdown
---
kind: light
title:
created:
---

# Work notes: <title>

## Goal

## Constraints

## Todos

- [ ]

## Open questions

## Learning (optional)

Short familiarity / subject hints only. No Planner scaffolding on the light path.

## Notes
```

The developer updates todos while iterating. Pair reads them; Pair does not write them.

Use this path for small intentional changes where full grilling would be overkill. Trivial fixes (typo, lint, obvious one-liner) need no plan at all—just commit.

Follow-ups on work just completed: prefer appending a follow-up slice or bouncing the plan briefly `done` → `implementing`, rather than starting a full new feature lifecycle.

If a light change reveals a real knowledge gap, either learn in-flight with Pair, run a teach session, or promote the work to a full plan so Planner can build a proper learning map. Finalize may recommend lessons or records when closing light work.

---

## 5. Phase 1: planning (developer)

1. Create a directory under `humai-plans/drafts/`.
2. Copy the rough-plan template (or write a short rough plan).
3. Write the rough plan yourself—including learning focus when relevant.
4. Include uncertainty rather than hiding it.
5. Stop once there is enough substance for a useful grilling session.

Do not create a refined plan in this phase. AI autocomplete may help with wording; the developer supplies the underlying understanding.

---

## 6. Phase 2: grilling and refinement (Planner)

### 6.1 OpenCode Planner agent

Planner is a primary OpenCode agent with:

- Permission to edit documentation paths: `humai-plans/**`, `CONTEXT.md`, `humai-docs/adr/**`, **`humai-teach/**`**
- Production code edits denied
- A short role prompt that defers shared workflow rules to `AGENTS.md`
- Instruction to load the **`grill-with-docs`** skill for codebase-related feature grilling
- Use **`grill-me`** (or equivalent) only for ideas that need no codebase test

Do not paste a long planning-only preamble into every chat. The Planner agent prompt already encodes boundaries.

### 6.2 Starting grilling

1. Move the plan: `humai-plans/drafts/<slug>` → `humai-plans/grilling/<slug>`
2. Switch to the Planner agent in OpenCode
3. Point it at the rough plan and begin grilling (one question at a time via the skill)

### 6.3 During grilling

Planner:

- Explores the codebase instead of asking what code can answer
- Checks domain language in `CONTEXT.md`
- Tests assumptions with concrete scenarios
- Updates `CONTEXT.md` only with durable terminology
- Offers ADRs sparingly when criteria are met
- Does not implement production code
- **Learning (soft tutor prep):**
  - Spots subjects the plan touches where the developer may lack coverage
  - Offers a short prior-knowledge check for flagged subjects; on the developer’s say-so, may write prior-knowledge `learning-records/` and/or record familiarity in the learning map
  - Scaffolds missing **subject shells** under `humai-teach/<subject>/` (at least a project-grounded `MISSION.md`, and empty `lessons/` / `learning-records/` as needed)—not full HTML lesson courses
  - Inspects existing lessons in relevant subjects and, in every full refined plan, links directly relevant lessons the developer has completed for refresh; writes `None` when there are none
  - Treats a lesson as completed only when the developer confirms it or a teaching record explicitly establishes completion; file existence alone is not completion or proof of proficiency
  - Writes the structured **Learning map** into `refined-plan.md` with concrete links (subjects, relevant completed lessons, relevant records, suggested lessons/topics)
  - Does not treat missing lessons as blocking questions

When grilling is complete, Planner writes or updates `refined-plan.md` in the same directory using the repository template (must-read block first; keep it human-skimmable).

### 6.4 After grilling

1. Developer reviews `refined-plan.md` (including the learning map)
2. Ensure no blocking questions remain (learning gaps are not blocking)
3. Move: `grilling` → `refined`
4. When accepting and coding starts: `refined` → `implementing`
5. Optionally, before or between slices: `cd humai-teach/<subject>` and run the teach skill for suggested topics

---

## 7. Phase 3: implementation

### 7.1 Activating work

**Full path:** move `refined/<slug>` → `implementing/<slug>`, create or switch to the branch, open OpenCode on **Pair**, run a thin start command or prompt that rehydrates from `refined-plan.md` (including the learning map).

**Light path:** create `implementing/<slug>/work-notes.md` directly, then start Pair the same way.

### 7.2 How implementation should feel

```text
write a small piece of code
→ inspect autocomplete rather than accepting blindly
→ run or inspect feedback
→ ask Pair when useful
→ optionally take a formal teach lesson when suggested or when stuck twice
→ continue
```

The developer decides when interaction is useful. Pair is not a continuous commentator.

### 7.3 Interaction surfaces

| Surface | Host | Use for |
|---|---|---|
| Tab / edit prediction | Zed | Reducing typing |
| Inline assist | Zed | Tiny local questions about selected code |
| Pair session | OpenCode | Plan-aware discussion, trade-offs, teaching, checkpoints |
| Teach session | OpenCode + teach skill | Formal lessons / glossary / learning-records in `humai-teach/<subject>/` |

Keep one Pair session for a coherent slice. Start a fresh one when changing slices, features, or when old discussion contaminates context.

### 7.4 Production code ownership

The developer writes production code.

OpenCode **Build** (or equivalent full-edit agent) is not part of this workflow. It may remain installed for rare out-of-band trivial agent edits; those are outside the defined loop.

Zed completions the developer accepts are still developer-driven.

### 7.5 Formal teach vs Pair-in-flight

Two learning paths:

1. **Formal:** `cd humai-teach/<subject>/`, run the teach skill → lessons, glossary, learning-records.
2. **In-flight:** Pair teaches/hints while implementing.

**Heuristic:** prefer a formal teach session when Pair’s checkpoint suggestion points at a lesson, **or** the same confusion appears twice. Otherwise Pair-in-flight is enough. Never required before coding.

---

## 8. OpenCode agents

### 8.1 Pair

**Permissions:** read/search allowed (including `humai-teach/**`); edit denied; destructive/terminal writes denied by default.

**Prompt focus (role-only):**

- Follow `AGENTS.md`
- Resolve active plan from `humai-plans/implementing/` only
- Support both `refined-plan.md` and `work-notes.md`
- **Learning calibration:** treat the plan’s learning map (or light Learning blurb) as authoritative for this feature; follow its links into `humai-teach/` (especially `learning-records/` and relevant completed lessons). Do not independently rescan all of `humai-teach/` unless the map is missing and learning focus is clearly material
- Use linked completed lessons as refresh material when relevant or requested. Completion alone is not proof of proficiency; learning-records remain the durable level signal
- Default: at most four short sentences
- Prefer one useful question or hint over a complete solution
- If the approach is sound: confirm briefly and stop
- If the developer is wrong or lacking knowledge: intervene with teaching, hinting, suggesting, criticizing, or explaining as fits—**intensity scaled by familiarity claims and learning-records** (few/no records + learning focus → teach-heavy; solid records → normal/brief)
- Honour learning focus / learning map with a slower, more deliberate style when indicated
- **Checkpoint tutor:** at `pair-start`, `check`, or slice boundaries, may offer **at most one** next learning step (lesson link or micro-concept) when material; otherwise stay quiet
- Do not provide full implementations unless explicitly asked (e.g. show-code)
- Distinguish repository facts from assumptions
- Mention missing context only when material
- Never write under `humai-teach/**`

**Thin commands** (optional, OpenCode prompts/commands):

| Command | Purpose |
|---|---|
| `pair-start` | Rehydrate: active plan, slice if known, top acceptance criterion or current todo, learning map / intensity posture, one material link if present (prefer a completed lesson when refresh would help), material gaps, ready? |
| `nudge` | One Socratic hint; no code; max two sentences |
| `check` | Single most consequential issue vs plan + supplied context; or “No material concern found.” May include one learning next-step if material |
| `deep` | Temporarily lift brevity for a detailed explanation; still read-only |
| `show-code` | Smallest code example for this response only |

Use `check` at semantic checkpoints (after choosing an approach, finishing a slice, before declaring done)—not after every save.

### 8.2 Planner

**Permissions:** allow edits under `humai-plans/**`, `CONTEXT.md`, `humai-docs/adr/**`, **`humai-teach/**`**; deny production code edits.

**Prompt focus:** planning-only role, docs hygiene, load `grill-with-docs` for feature work, follow `AGENTS.md`, build learning map + subject shells as in §6.3, and link relevant completed lessons for refresh—not full course authorship during grilling.

### 8.3 Finalize

**Permissions:** docs allowlist similar to Planner for plans / `CONTEXT.md` / ADRs; **`humai-teach/**` only to append a learning-record when the developer explicitly confirms** something non-obvious stuck (or prior knowledge should be recorded). Recommendations may also be written into the plan Outcome.

**Behaviour:**

1. Read the active plan under `implementing/`
2. Confirm acceptance criteria / todos and remaining gaps with the developer
3. Help complete Outcome (full) or a short done note (light)
4. Offer ADR only if criteria met
5. **Learning follow-ups:** may recommend lessons and/or learning-records when knowledge gaps showed up (full path or light path)—suggestions only, not a gate
6. On explicit approval: move to `humai-plans/done/<year>/` (or `cancelled/`)
7. Summarize what changed

Finalize closes the loop. It is not a second `check`. Mid-work concern spotting stays with Pair’s `check`.

Trivial no-plan commits do not use Finalize.

---

## 9. Instruction architecture

### 9.1 Single durable rulebook

| Concern | Location |
|---|---|
| Shared workflow law (folders, cardinality, plan kinds, ADR criteria, CONTEXT.md rules, humai-teach/ rules, sources of truth) | **`AGENTS.md` only** |
| Pair / Planner / Finalize personality and tool boundaries | **OpenCode agent prompts** (short; “follow AGENTS.md”) |
| One-shot verbs | Thin OpenCode commands that do not restate lifecycle rules |
| Grilling interview loop | **`grill-with-docs` skill** (loaded on demand) |
| Formal lessons in a subject workspace | **`teach` skill** (run only with cwd = `humai-teach/<subject>/`) |

Do not triple-copy the same paragraphs across `AGENTS.md`, agent prompts, and commands.

### 9.2 Suggested `AGENTS.md` outline

```markdown
# Project agent instructions

## Purpose
Human-led AI workflow. Developer owns production code.
AI supports planning, understanding, completion, review, and teaching
calibrated to documented learning level.

## Sources of truth (conflict order)
1. Executable code and tests
2. Active plan under humai-plans/implementing/
   (including learning map when present)
3. Accepted ADRs under humai-docs/adr/
4. CONTEXT.md domain terminology (project language only)
5. humai-teach/<subject>/ via plan links (learning-records, glossary, lessons)
6. Original rough plan (if present)
7. Current chat history (never durable state)

## Plan lifecycle
(drafts → grilling → refined → implementing → done/cancelled)
Directory location is authoritative.
At most one plan in grilling, refined, and implementing.
implementing accepts refined-plan.md (full) or work-notes.md (light).
Learning gaps never block plan moves.

## Active-plan resolution
(inspect implementing/; require exactly one; read refined-plan or work-notes;
 use learning map / Learning blurb for Pair intensity)

## humai-teach/
Committed subject workspaces. Broad craft subjects; expanding missions OK.
cd humai-teach/<subject> before running the teach skill.
learning-records/ = durable proficiency signal.
Lesson file existence does not prove completion; completion must be confirmed or recorded.
Planner may scaffold shells + write records (prior knowledge), and every full learning map
links relevant completed lessons for refresh or says None.
Pair: read-only on humai-teach/; may refresh from completed lessons linked by the plan.
Finalize: may recommend; may append a record only on explicit confirmation.

## CONTEXT.md
Domain glossary only (this product). No feature plans, implementation dumps,
or general C/raylib textbooks (those belong under humai-teach/).

## ADRs
Only when hard to reverse, surprising without context, and a real trade-off.

## Completion
Move to done only when criteria/todos checked, verification done,
deviations recorded, and warranted ADRs handled or explicitly rejected.
Learning follow-ups are optional recommendations.
```

Pair-specific brevity and teaching rules live in the Pair agent file, not duplicated here at length.

---

## 10. Completing and archiving

When the developer believes work is finished:

1. Run relevant automated tests (developer)
2. Perform manual verification from the plan (developer)
3. Optionally Pair `check` once on final changes
4. Invoke **Finalize**
5. Complete Outcome / done note
6. Decide ADR warrant
7. Optionally capture learning follow-ups (recommended lessons/records; append a record only if the developer confirms)
8. Approve archive → `humai-plans/done/<year>/<slug>`

Cancelled plans get a short cancellation note, then move to `humai-plans/cancelled/<year>/<slug>`. Do not delete cancelled plans automatically.

Routine checklist items (tests, moves, outcome prompts) should be driven by Finalize over time; v1 still expects the developer to run tests and approve the move.

---

## 11. Session survival

| Knowledge | Durable location |
|---|---|
| Project-wide AI behavior | `AGENTS.md` |
| Domain vocabulary (this product) | `CONTEXT.md` |
| Craft vocabulary / lessons / level proof | `humai-teach/<subject>/` |
| Lasting architecture decisions | `humai-docs/adr/` |
| Initial feature thinking | `rough-plan.md` |
| Agreed full implementation + learning map | `refined-plan.md` |
| Light active work | `work-notes.md` |
| Work status | Plan directory |
| Actual system state | Code and tests |
| Temporary discussion | Chat / OpenCode session |

A new Pair session reconstructs context from:

```text
AGENTS.md
→ humai-plans/implementing/
→ refined-plan.md or work-notes.md
→ learning map / Learning blurb → follow humai-teach/ links
→ CONTEXT.md
→ relevant ADRs
→ code
→ current changes and diagnostics
```

---

## 12. Assumptions and gotchas

### 12.1 Pair cannot see everything automatically

Supply state explicitly: selection, diffs, lint, pasted test output or logs when relevant.

### 12.2 Save before asking about current work

Do not assume the agent has a perfect live view of unsaved buffers.

### 12.3 Keep Pair prompts short

Read-only, active-plan aware, short by default, one issue at a time, teach before dumping solutions, state material missing context. Everything else lives in repository artifacts.

### 12.4 Do not let `CONTEXT.md` become a dump

Growing glossary noise is a warning sign. Feature behaviour belongs in plans; rationale in ADRs; general craft learning belongs under `humai-teach/`.

### 12.5 Completion can still remove you from the loop

Accepting large autocomplete without understanding recreates autonomous implementation. Prefer partial acceptance.

### 12.6 Do not over-automate plan moves initially

Introduce deterministic scripts only after real repetitive friction.

### 12.7 Do not run the teach skill from the repo root

Always `cd humai-teach/<subject>` first. Document this in `humai-teach/README.md`.

### 12.8 Do not turn learning into a gate

Suggested lessons and empty subject shells are tutor fuel. The developer may skip them and learn in-flight with Pair. Pair should turn teaching up when records are thin—not refuse to help with implementation.

### 12.9 Do not let Planner author full courses during grilling

Scaffold shells and a short learning map. Real lessons (HTML, citations, ZPD) belong in teach-skill sessions.

---

## 13. Trial and evaluation

Use this workflow on one medium-sized feature (full path) and a few light-path changes before expanding.

Observe:

- Did grilling uncover assumptions early?
- Was the refined must-read block usable while coding?
- Did `work-notes.md` suffice for small work without ceremony?
- Did Pair steer/teach usefully without writing code?
- Did the learning map calibrate Pair without blocking progress?
- Did links to relevant completed lessons make refresh easy without being mistaken for proficiency evidence?
- Did Planner subject shells stay thin (not accidental courses)?
- Did formal teach sessions help when used, without becoming mandatory?
- Did Finalize learning recommendations feel useful?
- Did Finalize make archiving reliable?
- Did folder status stay obvious with at-most-one in-flight plans?
- Did Zed + OpenCode feel like one workspace (editor + terminal agent)?

After the trial, change only parts that caused measurable friction.

---

## 14. Minimum viable setup checklist

### Repository

- [ ] Add `AGENTS.md` (shared law only, including humai-teach/ rules)
- [ ] Add `CONTEXT.md`
- [ ] Add `humai-plans/` with the six lifecycle directories
- [ ] Add rough and refined templates (including Learning map on refined)
- [ ] Add `humai-docs/adr/`
- [ ] Document light-path `work-notes.md` convention
- [ ] Add `humai-teach/README.md` (cd-into-subject rule)
- [ ] Allow empty `humai-teach/` until Planner scaffolds the first subject

### OpenCode

- [ ] Configure ChatGPT / OpenAI models via subscription or provider settings
- [ ] Create **Pair** agent (edit denied; teaching-oriented prompt; read teach via plan links)
- [ ] Create **Planner** agent (docs allowlist including `humai-teach/**`; load `grill-with-docs`)
- [ ] Create **Finalize** agent/command (learning follow-ups; record append only on confirm)
- [ ] Add thin commands: pair-start, nudge, check, deep, show-code
- [ ] Install `grill-with-docs` (and optionally `grill-me`) where OpenCode can load skills
- [ ] Ensure the **teach** skill is available; document subject cwd requirement

### Zed

- [ ] Configure ChatGPT subscription / models for edit prediction and inline assist
- [ ] Prefer Vim motions or preferred keybindings
- [ ] Keep OpenCode available in a terminal pane (or OpenCode Desktop beside Zed)

### First feature (full path)

- [ ] Write `rough-plan.md` in `drafts/` (optional Learning focus)
- [ ] Move to `grilling/`; run Planner + `grill-with-docs`
- [ ] Review learning map, relevant completed-lesson refresh links, and any new `humai-teach/<subject>/` shells
- [ ] Produce and review `refined-plan.md` through `refined/`
- [ ] Move to `implementing/` when accepted; Pair with pair-start
- [ ] Optionally take suggested teach lessons; or learn in-flight
- [ ] Implement with Zed completion + deliberate Pair checkpoints
- [ ] Finalize → `done/` (optional learning follow-ups)

### First light change

- [ ] Create `implementing/<slug>/work-notes.md` (optional Learning blurb)
- [ ] Pair against it
- [ ] Finalize or simply finish if trivial enough that no plan was needed

---

## 15. Final workflow

```text
YOU
write humai-plans/drafts/<feature>/rough-plan.md
(optional learning focus / familiarity)

        ↓

YOU
move to humai-plans/grilling/

        ↓

OPENCODE PLANNER + grill-with-docs
challenge assumptions
inspect the codebase
resolve terminology
identify edge cases
spot learning subjects; prior-knowledge check
scaffold humai-teach/<subject>/ shells when needed
find relevant completed lessons for refresh
write refined-plan.md (including Learning map)
optionally update CONTEXT.md / ADR / learning-records

        ↓

YOU
review → refined/ → implementing/
(optional: cd humai-teach/<subject> → teach skill)

        ↓

OPENCODE PAIR
pair-start (plan + learning map → intensity + relevant refresh)
read-only
teach / hint / redirect
one next learning step at checkpoints when material
refined-plan or work-notes aware

        ↓

YOU (+ Zed completion / inline)
write the code
use Pair checkpoints as needed
formal teach when suggested or stuck twice

        ↓

OPENCODE FINALIZE
outcome, verification notes, optional ADR
optional learning follow-ups (lessons / records)
on approval → humai-plans/done/
```

Light path:

```text
YOU write humai-plans/implementing/<slug>/work-notes.md
  (optional Learning blurb)
→ OPENCODE PAIR
→ YOU implement
→ OPENCODE FINALIZE (if useful; may recommend learning follow-ups)
```

Central principle:

> Project files carry the context. The AI conversation consumes that context but does not own it.

Teaching corollary:

> `humai-teach/` carries craft knowledge and proof of level. Plans carry the learning map for the current feature. Neither locks the developer out of implementation.
