---
name: hydrate-memory
description: Use Hydrate's persistent memory. Recall relevant prior context before starting non-trivial work, and save durable decisions, conventions, and user preferences as they are established. On Claude Code the lifecycle hooks already inject and capture automatically; call these tools explicitly when you need memory the hooks did not surface, or on any surface where hooks do not fire.
---

# Hydrate memory — deliberate use

Hydrate stores persistent, local memory across sessions. In Claude Code the
lifecycle hooks inject relevant facts before each prompt and capture the session
on completion, so most recall and capture happen automatically. Call these tools
yourself when you need memory the ambient inject did not surface, or on a surface
where hooks do not fire.

## Recall before you start

At the beginning of a task that could depend on earlier decisions, conventions,
unfinished work, or the user's stated preferences, call:

- `hydrate_recall` with a query describing the topic — returns ranked facts, canon,
  and prior-session context.
- `hydrate_session_resume` (pass the project slug) after a context reset, to pull the
  last distilled session and in-flight state.

Do this once, up front — not repeatedly. If the recall returns nothing relevant,
proceed; do not block on it.

## Capture what's durable

When a decision, convention, constraint, or stable user preference is established
during the work, persist it:

- `hydrate_save_fact` — a durable fact or decision, scoped to the project.
- `hydrate_canon_add` — a binding project rule that should override conflicting
  future requests.

## What NOT to save

Do not save secrets, credentials, tokens, one-off command output, or speculative
conclusions. Save the decision and the reason, not the transcript. When two stored
memories conflict, surface the conflict rather than silently choosing one.
