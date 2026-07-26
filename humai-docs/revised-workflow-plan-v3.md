# Human-Led AI Development Workflow

**Status:** Revised workflow, version 3 (skills-only)  
**Primary objective:** Keep the developer in control of implementation while using AI for completion, planning clarification, contextual discussion, and teaching—calibrated to the developer’s documented level of understanding.

**Mechanism:** IDE- and model-agnostic **skills** under `.agents/skills/`. Shared law in `AGENTS.md`. No required host agent or editor.

---

## 1. Executive summary

This workflow deliberately avoids autonomous AI implementation as its default.

The development loop is:

```text
rough-plan.md
→ grilling session (/planner)
  (may scaffold humai-teach/<subject>/ shells + learning map in refined-plan)
→ refined-plan.md
→ human-led implementation with /pair
  (optional formal teach sessions in humai-teach/<subject>/)
→ /finalize (outcome + archive; may recommend lessons / learning-records)
```

A lighter path exists for small work: drop `work-notes.md` directly into `humai-plans/implementing/` and use the same `/pair` skill. Light path may include a tiny optional learning blurb; planner does not scaffold `humai-teach/` unless the work is later promoted to a full plan. Finalize may still flag knowledge gaps.

### Roles (skills)

| Role | Skill | Responsibility |
|---|---|---|
| **Planner** | `/planner` | Challenges a rough plan, resolves ambiguity, checks the simplest architecture fit for known requirements, and produces a refined plan; may update domain docs; scaffolds teach subject shells and writes the plan’s learning map, including relevant completed lessons for refresh |
| **Pair programmer** | `/pair` (+ thin verbs) | Reads project and current work, teaches, hints, redirects; calibrates depth from the plan’s learning map and linked learning-records; may refresh from linked completed lessons; does not edit production code or `humai-teach/` |
| **Teach sessions** | `/teach` | Formal lessons, glossaries, and learning-records inside `humai-teach/<subject>/` (developer invokes after `cd` into that subject) |
| **Finalize** | `/finalize` | Verifies completion with the developer, records outcome, archives the plan; may recommend lessons/learning-records; may append a learning-record only on explicit developer confirmation |

Editor autocomplete / inline assist is whatever the developer prefers; it is outside this skill loop. Model choice is the developer’s.

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

**North star:** Skills suggest the next helpful thing and adjust help to the developer’s level. The developer freely chooses what to do next—like a personal tutor on call, not a locked curriculum.

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

No generated manifests, schemas, or lifecycle tooling are required.

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
| `drafts` → `grilling` | Developer, when starting `/planner` |
| Write `refined-plan.md` while in `grilling/` | `/planner` (docs only) |
| `grilling` → `refined` | Developer, when grilling is done and review should be cold |
| `refined` → `implementing` | Developer, when accepting the plan and coding starts |
| Create light work directly in `implementing/` | Developer (skips earlier stages) |
| `implementing` → `done` / `cancelled` | `/finalize`, only after explicit developer approval |

Skills may remind the developer that a move is appropriate. They must not silently relocate plans.

Mid-lifecycle moves are manual. Deterministic scripts (`plan start`, `plan done`, …) may be added later if friction appears.

**Learning is not a lifecycle gate.** Missing lessons or learning-records never block `refined` → `implementing`. They only change how aggressively Pair (and suggestions from Finalize) teach.

### 2.5 Architecture is deliberate but bounded

Every full refined plan gets a brief architecture-fit pass grounded in the current system. Planner chooses the simplest structure that supports known requirements, calls out obvious material wins in responsibilities, boundaries, data flow, or dependency direction, and records "no architectural change" when the existing structure is already the best fit.

This is not permission for speculative future-proofing, abstractions without current pressure, or redesigning unrelated areas.

### 2.6 ADRs are exceptional

Create an Architecture Decision Record only when all are true:

1. Meaningfully expensive to reverse.
2. A future developer would reasonably wonder why it was chosen.
3. Real alternatives and trade-offs were considered.

`CONTEXT.md` remains a domain glossary, not an implementation plan.

Considering architecture does not make every choice ADR-worthy. Normal implementation choices stay in the refined plan, work notes, or code.

### 2.7 Teaching lives under `humai-teach/` (soft tutor context)

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

### 2.8 Edit bounds are prompt discipline

Skills declare **working directory** and **edit boundaries** in their `SKILL.md`. There is no required IDE permission sandbox. Review diffs. Production code remains developer-owned.

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
│   ├── drafts/
│   ├── grilling/
│   ├── refined/
│   ├── implementing/
│   ├── done/
│   │   └── <year>/
│   └── cancelled/
│       └── <year>/
│
├── humai-teach/
│   ├── README.md
│   └── <subject>/            # teach-skill workspaces (committed)
│
├── humai-docs/
│   ├── adr/
│   └── revised-workflow-plan-v3.md
│
└── .agents/
    └── skills/               # portable project skills
        ├── pair/
        ├── pair-start/ nudge/ check/ deep/ show-code/
        ├── planner/
        ├── finalize/
        ├── grill-me/ grill-with-docs/ grilling/
        ├── domain-modeling/
        └── teach/
```

Canonical skills path: `.agents/skills/`. Some tools load from other paths; symlink or configure as needed—without pinning an IDE.

Preserve `rough-plan.md` through the lifecycle when it exists. `refined-plan.md` is authoritative for full-path work; `work-notes.md` is authoritative for light-path work.

---

## 4. Plan templates

Authoritative copies live under `humai-plans/_templates/`. Do not diverge this design doc from those files.

- Rough plan: `humai-plans/_templates/rough-plan.md`
- Refined plan: `humai-plans/_templates/refined-plan.md` (must-read block first; learning map; Outcome completed during Finalize)
- Light path: freeform `work-notes.md` under `implementing/<slug>/` (optional Learning blurb; no planner scaffolding)

Implementation slices should be small, observable vertical results—not layers such as “create types,” then “create service,” then “create UI.”

Trivial fixes (typo, lint, obvious one-liner) need no plan at all—just commit.

If a light change reveals a real knowledge gap, either learn in-flight with Pair, run a teach session, or promote the work to a full plan so planner can build a proper learning map.

---

## 5. Phase 1: planning (developer)

1. Create a directory under `humai-plans/drafts/`.
2. Copy the rough-plan template (or write a short rough plan).
3. Write the rough plan yourself—including learning focus when relevant.
4. Include uncertainty rather than hiding it.
5. Stop once there is enough substance for a useful grilling session.

Do not create a refined plan in this phase.

---

## 6. Phase 2: grilling and refinement (`/planner`)

### 6.1 Planner skill

From the **repository root**, run `/planner`. Edit bounds (see `AGENTS.md` and the skill):

- May edit: `humai-plans/**`, `CONTEXT.md`, `humai-docs/adr/**`, `humai-teach/**`
- Must not edit: production code
- Loads **`/grill-with-docs`** for codebase-related feature grilling
- Uses **`/grill-me`** only for ideas that need no codebase test

### 6.2 Starting grilling

1. Move the plan: `humai-plans/drafts/<slug>` → `humai-plans/grilling/<slug>`
2. From repo root, run `/planner`
3. Point it at the rough plan and begin grilling (one question at a time via the grill skill)

### 6.3 During grilling

Planner:

- Explores the codebase instead of asking what code can answer
- Checks domain language in `CONTEXT.md`
- Tests assumptions with concrete scenarios
- Updates `CONTEXT.md` only with durable terminology
- Takes a bounded architecture pass grounded in the existing system
- Avoids speculative abstractions, hypothetical future-proofing, or redesign of unrelated areas
- Offers ADRs sparingly when criteria are met
- Does not implement production code
- **Learning (soft tutor prep):** prior-knowledge check on say-so; scaffold subject shells; link relevant completed lessons for refresh (`None` when none); write structured **Learning map** into `refined-plan.md`
- Does not treat missing lessons as blocking questions

When grilling is complete, write or update `refined-plan.md` in the grilling plan directory using the repository template (must-read block first; keep it human-skimmable).

### 6.4 After grilling

1. Developer reviews `refined-plan.md` (including Architecture fit and the learning map)
2. Ensure no blocking questions remain (learning gaps are not blocking)
3. Move: `grilling` → `refined`
4. When accepting and coding starts: `refined` → `implementing`
5. Optionally, before or between slices: `cd humai-teach/<subject>` and run `/teach` for suggested topics

---

## 7. Phase 3: implementation (`/pair`)

**Full path:** move `refined/<slug>` → `implementing/<slug>`, run `/pair-start` from repo root (rehydrates from `refined-plan.md` including the learning map).

**Light path:** create `implementing/<slug>/work-notes.md` directly, then `/pair` the same way.

The developer writes production code. Pair teaches, hints, and checkpoints; it never edits the repository.

### Thin Pair skills (compose `/pair`)

| Skill | Purpose |
|---|---|
| `pair-start` | Rehydrate: active plan, slice, top criterion/todo, learning posture, material gaps, ready? |
| `nudge` | One Socratic hint; no code; max two sentences |
| `check` | Single most consequential issue vs plan + supplied context; or “No material concern found.” |
| `deep` | Temporarily lift brevity for a detailed explanation; still read-only |
| `show-code` | Smallest code example for this response only |

Use `check` at semantic checkpoints—not after every save.

### Formal teach vs Pair-in-flight

1. **Formal:** `cd humai-teach/<subject>/`, run `/teach`
2. **In-flight:** Pair teaches/hints while implementing

**Heuristic:** prefer a formal teach session when Pair’s checkpoint points at a lesson, or the same confusion appears twice. Otherwise Pair-in-flight is enough. Never required before coding.

---

## 8. Phase 4: completion (`/finalize`)

From the **repository root**, run `/finalize`.

1. Read the active plan under `implementing/`
2. Confirm acceptance criteria / todos and remaining gaps with the developer
3. Help complete Outcome (full) or a short done note (light)
4. Offer ADR only if criteria met
5. **Learning follow-ups:** may recommend lessons and/or learning-records—suggestions only, not a gate
6. On explicit approval: move to `humai-plans/done/<year>/` (or `cancelled/`)
7. Summarize what changed

Finalize closes the loop. It is not a second `/check`. Mid-work concern spotting stays with Pair’s `/check`.

Trivial no-plan commits do not use Finalize.

---

## 9. Instruction architecture

| Concern | Location |
|---|---|
| Shared workflow law (folders, cardinality, plan kinds, ADR criteria, CONTEXT.md rules, humai-teach/ rules, sources of truth, skill index) | **`AGENTS.md`** |
| Role procedures and edit/cwd bounds | **`.agents/skills/*/SKILL.md`** |
| One-shot Pair verbs | Thin skills that invoke `/pair` |
| Grilling interview loop | **`grilling`** / wrappers **`grill-me`**, **`grill-with-docs`** |
| Formal lessons in a subject workspace | **`teach`** (cwd = `humai-teach/<subject>/`) |

Do not triple-copy the same paragraphs across `AGENTS.md` and skill bodies.

Composition:

```text
grill-me            → /grilling
grill-with-docs     → /grilling + /domain-modeling
planner             → /grill-with-docs (or /grill-me) + refined-plan authorship
pair-start|nudge|…  → /pair
```

---

## 10. Skill cwd and edit bounds (summary)

| Skill | Invoke from | May edit | Must not edit |
|---|---|---|---|
| `pair` (+ thin verbs) | repo root | nothing | production code, plans, CONTEXT, ADRs, teach |
| `planner` | repo root | plans, CONTEXT, ADRs, teach shells/records on say-so | production code |
| `finalize` | repo root | docs allowlist; teach append on confirm; archive on approval | production code |
| `grill-me` / `grilling` | repo root | nothing until shared understanding | production code |
| `grill-with-docs` | repo root | via domain-modeling | production code |
| `domain-modeling` | repo root | CONTEXT, ADRs | production code; feature plans |
| `teach` | `humai-teach/<subject>/` | that subject | app code; other subjects; plans (default) |

Authoritative table: `AGENTS.md`.

---

## 11. Evaluation criteria

- Did folder location stay the authoritative plan status?
- Did learning stay soft (never blocking `refined` → `implementing`)?
- Did Pair stay read-only while the developer owned production code?
- Did skills load from `.agents/skills/` in the developer’s preferred tool?
- Was teach always invoked from the subject directory?

---

## 12. Quick loop

```text
YOU          drafts/<slug>/rough-plan.md
YOU          → grilling/<slug>
/planner     → refined-plan.md (+ learning map)
YOU          → refined/ → implementing/
YOU          optional: cd humai-teach/<subject> → /teach
YOU + /pair  implement + checkpoints
YOU          verify
/finalize    → done/<year>/ on approval
```
