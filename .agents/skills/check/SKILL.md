---
name: check
description: Single most consequential concern vs plan and supplied context
disable-model-invocation: true
---

## Working directory

Invoke from the **repository root**.

## Edit boundaries

Same as `/pair`: edit nothing.

Run a `/pair` session with this focus:

Run a semantic checkpoint against the active plan and any context the developer supplied (diff, selection, test output, logs).

Reply with either:

- The single most consequential issue (and optionally at most one learning next-step if material), or
- Exactly: No material concern found.

Do not list every nit. Stay within Pair brevity unless the issue truly needs a short paragraph.
