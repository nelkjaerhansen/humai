# plans/

Plan location is the authoritative status. Do not maintain a separate dashboard.

| Location | Meaning | Cardinality |
|---|---|---|
| `drafts/` | Rough ideas being formed | Many |
| `grilling/` | Actively being grilled | At most one |
| `refined/` | Grilling done; awaiting human review | At most one |
| `approved/` | Human accepted; coding not started | At most one |
| `implementing/` | Active implementation (Pair’s focus) | At most one |
| `done/<year>/` | Completed and verified | Many |
| `cancelled/<year>/` | Deliberately abandoned | Many |

## Full path

1. Copy `plans/_templates/rough-plan.md` into `drafts/<slug>/rough-plan.md` and write it yourself.
2. Move `drafts/<slug>` → `grilling/<slug>` and run **Planner** + `grill-with-docs`.
3. Review `refined-plan.md` → move to `refined/` → `approved/` → `implementing/` when coding starts.
4. Use **Pair** against the active plan. When finished, run **Finalize** → `done/<year>/`.

## Light path

Create `implementing/<slug>/work-notes.md` directly (no template required). Use Pair. Finalize if useful.

Trivial fixes need no plan—just commit.

## Templates

- `_templates/rough-plan.md`
- `_templates/refined-plan.md`

Preserve `rough-plan.md` through the lifecycle when it exists. `refined-plan.md` is authoritative for full-path work; `work-notes.md` for light-path work.

You move plans between folders. Agents may remind you; only Finalize relocates after your explicit approval (to `done/` or `cancelled/`).
