# humai

**hum**an + **ai** — a forkable project template for human-led development with AI as planner, pair, and tutor.

You write the production code. OpenCode agents challenge plans, teach at your level, and help close work. Project files—not chat history—carry durable context.

Design reference: [`humai-docs/revised-workflow-plan-v3.md`](./humai-docs/revised-workflow-plan-v3.md)  
Agent rulebook: [`AGENTS.md`](./AGENTS.md)

---

## What you get

| Piece | Role |
|---|---|
| `humai-plans/` | Feature lifecycle by folder location |
| `CONTEXT.md` | Product domain glossary (stub) |
| `humai-teach/` | Craft learning workspaces (empty until Planner scaffolds subjects) |
| `humai-docs/adr/` | Architecture Decision Records when warranted |
| `.opencode/agents/` | **Pair**, **Planner**, **Finalize** |
| `.opencode/commands/` | `/pair-start`, `/nudge`, `/check`, `/deep`, `/show-code` |
| `.opencode/skills/` | Vendored `grill-with-docs`, `teach`, and dependencies |

---

## One-time setup

### 1. Fork or copy this repo

Start a new project from this template. Add your application code alongside the workflow folders—do not remove them.

### 2. ChatGPT for Zed and OpenCode

This workflow assumes a **ChatGPT subscription** used from both:

- **Zed** — edit prediction / inline assist (authenticate in Zed’s UI)
- **OpenCode** — Pair, Planner, Finalize (authenticate in OpenCode’s UI)

No API keys belong in this repository. Auth stays in the editor and OpenCode apps.

Pick whichever ChatGPT-capable model OpenCode and Zed expose for your account; this template does not pin a model ID.

### 3. Open the project in OpenCode

Open this repo (or your fork) as the OpenCode workspace. Custom agents and commands load from `.opencode/`. Vendored skills load from `.opencode/skills/`.

Switch agents with Tab (or your `switch_agent` keybind): **Pair**, **Planner**, **Finalize**. Prefer those over the built-in **Build** agent for this workflow.

### 4. Optional first edits

- Rename or fill the stub in [`CONTEXT.md`](./CONTEXT.md) when you have product terms
- Skim [`humai-plans/README.md`](./humai-plans/README.md) and [`humai-teach/README.md`](./humai-teach/README.md)

---

## Your loop (manual steps)

Agents do not move plan folders for you (except Finalize after you approve archive). **You** own these steps.

### Full path

```text
YOU          write humai-plans/drafts/<slug>/rough-plan.md
YOU          move → humai-plans/grilling/<slug>
PLANNER      grill + architecture fit → refined-plan.md (+ learning map and refresh links)
YOU          review → refined/ → implementing/
YOU          optionally: cd humai-teach/<subject> → teach skill
YOU + PAIR   implement (Zed completion + Pair checkpoints)
YOU          run tests / verify
FINALIZE     outcome → on your approval → humai-plans/done/<year>/
```

Numbered:

1. Copy `humai-plans/_templates/rough-plan.md` to `humai-plans/drafts/<slug>/rough-plan.md` and write it (include learning focus if useful).
2. Move that directory to `humai-plans/grilling/<slug>`.
3. In OpenCode, switch to **Planner**. Point it at the rough plan and start grilling (`grill-with-docs` for codebase work); Planner also checks the simplest architecture fit for known requirements.
4. Review `refined-plan.md` (including Architecture fit, the learning map, relevant completed lessons for refresh, and any new `humai-teach/<subject>/` shells).
5. Move `grilling/<slug>` → `refined/<slug>`, then → `implementing/<slug>` when you accept and coding starts.
6. Switch to **Pair**, run `/pair-start`, write code yourself. Use `/nudge`, `/check`, `/deep`, `/show-code` as needed.
7. Optional formal learning: `cd humai-teach/<subject>` then run the teach skill—never from the repo root.
8. When done: run tests, optionally `/check`, switch to **Finalize**, complete Outcome, approve archive to `humai-plans/done/<year>/`.

### Light path

1. Create `humai-plans/implementing/<slug>/work-notes.md` (freeform; optional Learning blurb).
2. **Pair** → you implement.
3. **Finalize** if useful (may recommend learning follow-ups).

Trivial fixes (typo, lint, obvious one-liner): no plan—just commit.

---

## Agents at a glance

| Agent | You use it for | Edits |
|---|---|---|
| **Planner** | Grill rough plans; assess bounded architecture fit; produce refined plans + learning maps | `humai-plans/**`, `CONTEXT.md`, `humai-docs/adr/**`, `humai-teach/**` |
| **Pair** | Plan-aware hints, teaching, checkpoints while you code | Nothing (read-only) |
| **Finalize** | Outcome, verification notes, archive on approval | Docs allowlist; `humai-teach/` only to append a confirmed learning-record |

Thin Pair commands: `/pair-start`, `/nudge`, `/check`, `/deep`, `/show-code`.

---

## Learning (soft, never a gate)

- Missing lessons never block `refined` → `implementing`.
- Planner may scaffold empty subject shells; you fill them via teach sessions when you want.
- Planner links relevant lessons you have completed in every full refined plan so Pair can use them for refresh; an existing lesson file alone does not count as completion.
- Pair dials teaching from the plan’s learning map and `learning-records/`; lesson completion is not proficiency evidence.
- Prefer a formal teach session when Pair suggests a lesson or you get stuck twice on the same confusion; otherwise learn in-flight with Pair.

---

## Layout

```text
/
├── README.md                 ← you are here (human runbook)
├── AGENTS.md                 ← shared agent law
├── CONTEXT.md                ← product glossary stub
├── opencode.jsonc
├── humai-plans/
├── humai-teach/
├── humai-docs/
│   ├── adr/
│   └── revised-workflow-plan-v3.md
└── .opencode/
    ├── agents/               pair | planner | finalize
    ├── commands/             pair-start | nudge | check | deep | show-code
    └── skills/               grill-with-docs | teach | …
```

---

## Zed + OpenCode

Use Zed for writing and completion; keep OpenCode in a terminal pane or Desktop beside it. Autocomplete you accept is still your code—prefer partial acceptance when learning.

---

## Further reading

- [`humai-plans/README.md`](./humai-plans/README.md) — folder status and templates  
- [`humai-teach/README.md`](./humai-teach/README.md) — subject workspaces and teach invocation  
- [`humai-docs/adr/README.md`](./humai-docs/adr/README.md) — when to write an ADR  
- [`humai-docs/revised-workflow-plan-v3.md`](./humai-docs/revised-workflow-plan-v3.md) — full workflow design  
- [`AGENTS.md`](./AGENTS.md) — rules agents must follow  
