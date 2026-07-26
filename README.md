# humai

**hum**an + **ai** — a forkable project template for human-led development with AI as planner, pair, and tutor.

You write the production code. Skills challenge plans, teach at your level, and help close work. Project files—not chat history—carry durable context.

The workflow is **IDE- and model-agnostic**: use whatever editor and agent can load project skills from `.agents/skills/`.

Shared law: [`AGENTS.md`](./AGENTS.md)

---

## What you get

| Piece | Role |
|---|---|
| `humai-plans/` | Feature lifecycle by folder location |
| `CONTEXT.md` | Product domain glossary (stub) |
| `humai-teach/` | Craft learning workspaces (empty until planner scaffolds subjects) |
| `humai-docs/adr/` | Architecture Decision Records when warranted |
| `.agents/skills/` | Workflow skills: pair, planner, finalize, grill, teach, … |

Thin Pair verbs are also skills: `/pair-start`, `/nudge`, `/check`, `/deep`, `/show-code` (each invokes `/pair`).

---

## One-time setup

### 1. Fork or copy this repo

Start a new project from this template. Add your application code alongside the workflow folders—do not remove them.

### 2. Use any capable editor / agent

Open this repo in your preferred tool. Skills live under [`.agents/skills/`](./.agents/skills/). Auth and model choice stay in your tool—nothing secret belongs in this repository.

Some tools load skills from other paths. If yours does not discover `.agents/skills/` automatically, point it at that folder (symlink or config)—without pinning a particular IDE.

### 3. Optional first edits

- Rename or fill the stub in [`CONTEXT.md`](./CONTEXT.md) when you have product terms
- Skim [`humai-plans/README.md`](./humai-plans/README.md) and [`humai-teach/README.md`](./humai-teach/README.md)

---

## Your loop (manual steps)

Skills do not move plan folders for you (except **finalize** after you approve archive). **You** own these steps.

### Full path

```text
YOU          write humai-plans/drafts/<slug>/rough-plan.md
YOU          move → humai-plans/grilling/<slug>
/planner     grill + architecture fit → refined-plan.md (+ learning map and refresh links)
YOU          review → refined/ → implementing/
YOU          optionally: cd humai-teach/<subject> → /teach
YOU + /pair  implement (your editor + Pair checkpoints)
YOU          run tests / verify
/finalize    outcome → on your approval → humai-plans/done/<year>/
```

Numbered:

1. Copy `humai-plans/_templates/rough-plan.md` to `humai-plans/drafts/<slug>/rough-plan.md` and write it (include learning focus if useful).
2. Move that directory to `humai-plans/grilling/<slug>`.
3. From the **repo root**, run `/planner`. Point it at the rough plan; it loads `grill-with-docs` for codebase work and checks the simplest architecture fit for known requirements.
4. Review `refined-plan.md` (including Architecture fit, the learning map, relevant completed lessons for refresh, and any new `humai-teach/<subject>/` shells).
5. Move `grilling/<slug>` → `refined/<slug>`, then → `implementing/<slug>` when you accept and coding starts.
6. From the repo root, run `/pair-start`, write code yourself. Use `/nudge`, `/check`, `/deep`, `/show-code` as needed.
7. Optional formal learning: `cd humai-teach/<subject>` then run `/teach`—never from the repo root.
8. When done: run tests, optionally `/check`, run `/finalize`, complete Outcome, approve archive to `humai-plans/done/<year>/`.

### Light path

1. Create `humai-plans/implementing/<slug>/work-notes.md` (freeform; optional Learning blurb).
2. `/pair` → you implement.
3. `/finalize` if useful (may recommend learning follow-ups).

Trivial fixes (typo, lint, obvious one-liner): no plan—just commit.

---

## Skills at a glance

| Skill | You use it for | Edits |
|---|---|---|
| **planner** | Grill rough plans; assess bounded architecture fit; produce refined plans + learning maps | `humai-plans/**`, `CONTEXT.md`, `humai-docs/adr/**`, `humai-teach/**` |
| **pair** | Plan-aware hints, teaching, checkpoints while you code | Nothing (read-only) |
| **finalize** | Outcome, verification notes, archive on approval | Docs allowlist; `humai-teach/` only to append a confirmed learning-record |
| **teach** | Formal lessons in a subject workspace | That `humai-teach/<subject>/` only |
| **grill-with-docs** / **grill-me** | Interview loops (planner loads these) | Glossary/ADRs via domain-modeling when using grill-with-docs |

Thin Pair skills: `/pair-start`, `/nudge`, `/check`, `/deep`, `/show-code` → each invokes `/pair`.

Full cwd and edit table: [`AGENTS.md`](./AGENTS.md).

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
├── AGENTS.md                 ← shared workflow law + skill index
├── CONTEXT.md                ← product glossary stub
├── humai-plans/
├── humai-teach/
├── humai-docs/
│   └── adr/
└── .agents/
    └── skills/               pair | planner | finalize | teach | grill-* | …
```

---

## Further reading

- [`humai-plans/README.md`](./humai-plans/README.md) — folder status and templates  
- [`humai-teach/README.md`](./humai-teach/README.md) — subject workspaces and teach invocation  
- [`humai-docs/adr/README.md`](./humai-docs/adr/README.md) — when to write an ADR  
- [`AGENTS.md`](./AGENTS.md) — rules skills must follow  
