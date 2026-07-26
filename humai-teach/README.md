# humai-teach/

Craft learning workspaces for this project. Each `humai-teach/<subject>/` directory is a full **teach-skill** workspace (one mission per workspace).

## Invocation rule

Always enter the subject before running the teach skill:

```bash
cd humai-teach/<subject>
```

Then invoke `/teach` (or load the `teach` skill).

**Do not run the teach skill from the repository root.** That treats the project root as the teaching workspace and will create the wrong files.

All other workflow skills (`/planner`, `/pair`, `/finalize`, grill skills) run from the **repository root**.

## Subject grain

Use broad craft buckets (`c-language`, `raylib`, `game-architecture`, …). Missions may expand as the project needs new skills in that craft. Split a new subject only when the craft clearly differs.

## Who creates what

- **planner** scaffolds thin subject shells during grilling when a plan needs them (`MISSION.md`, empty `lessons/` / `learning-records/`, etc.)—not full courses. It also links relevant completed lessons from these workspaces in every full refined plan so they can be refreshed.
- **You** run formal teach sessions inside a subject when you want lessons, glossary entries, or learning-records.
- **pair** may teach in-flight while coding; it never writes under `humai-teach/`.
- **finalize** may recommend lessons or records; it appends a learning-record only if you explicitly confirm.

## Proof of level

`learning-records/` are the durable signal of demonstrated understanding (or stated prior knowledge)—not mere lesson coverage.

An existing lesson file does not mean you completed it. Planner records a lesson as completed only when you confirm it or a teaching record explicitly establishes completion; completed lesson links are refresh resources, not proficiency evidence.

## Glossaries

- `CONTEXT.md` (repo root) = this **product’s** language
- `humai-teach/<subject>/GLOSSARY.md` = **general craft** terms for that subject

Do not copy teach glossaries into `CONTEXT.md`.
