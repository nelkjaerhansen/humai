# Project agent instructions

## Purpose

Human-led AI workflow (**humai**). The developer owns production code.
AI supports planning, understanding, completion, review, and teaching
calibrated to documented learning level.

## Sources of truth (conflict order)

1. Executable code and tests
2. Active plan under `plans/implementing/` (including learning map when present)
3. Accepted ADRs under `docs/adr/`
4. `CONTEXT.md` domain terminology (project language only)
5. `teach/<subject>/` via plan links (learning-records, glossary, lessons)
6. Original rough plan (if present)
7. Current chat history (never durable state)

## Plan lifecycle

`drafts` → `grilling` → `refined` → `approved` → `implementing` → `done` / `cancelled`

Directory location is authoritative. Do not maintain a separate status dashboard.

Cardinality:

- At most one plan in each of: `grilling/`, `refined/`, `approved/`, `implementing/`
- Many allowed in `drafts/`, `done/`, `cancelled/`

`implementing/` accepts `refined-plan.md` (full path) or `work-notes.md` (light path).

**Learning gaps never block plan moves.** Missing lessons or learning-records only change teaching intensity.

Agents may remind the developer that a folder move is appropriate. They must not silently relocate plans. Finalize moves to `done/` or `cancelled/` only after explicit developer approval.

## Active-plan resolution

Inspect `plans/implementing/` only:

- Exactly one plan directory → use it
- None → say so before plan-specific assumptions
- More than one → report the ambiguity
- Prefer `refined-plan.md`; otherwise `work-notes.md`
- If neither is present → report required context is missing

Use the plan’s **Learning map** (full) or **Learning** blurb (light) for Pair teaching intensity. Mention materially missing context only.

## teach/

Committed subject workspaces under `teach/<subject>/`. Broad craft subjects; expanding missions OK.

Always `cd teach/<subject>` (or open that folder as the workspace) before running the teach skill. Never run teach from the repo root.

`learning-records/` = durable proficiency signal (not mere lesson coverage).

- **Planner** may scaffold subject shells, write prior-knowledge records (on developer say-so), and author the plan learning map. Does not author full courses during grilling.
- **Pair** is read-only on `teach/`.
- **Finalize** may recommend lessons/records; may append a learning-record only on explicit developer confirmation.

## CONTEXT.md

Domain glossary only (this product’s language). No feature plans, implementation dumps, or general craft textbooks (those belong under `teach/`).

## ADRs

Create an ADR only when all are true:

1. Meaningfully expensive to reverse
2. A future developer would reasonably wonder why it was chosen
3. Real alternatives and trade-offs were considered

## Completion

Move to `done/` only when criteria/todos are checked, verification is done, deviations are recorded, and warranted ADRs are handled or explicitly rejected. Learning follow-ups are optional recommendations.

Trivial no-plan commits do not use Finalize.

## Role boundaries (summary)

| Role | Edits production code | Edits docs / teach |
|---|---|---|
| Pair | No | No |
| Planner | No | `plans/**`, `CONTEXT.md`, `docs/adr/**`, `teach/**` |
| Finalize | No | Same docs allowlist; `teach/**` only to append a confirmed learning-record |

OpenCode **Build** (full-edit agent) is outside this workflow.
