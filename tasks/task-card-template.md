# Task: <task name>

## Owner agent

OpenCode

## Status

planned

Allowed values:

- `planned`: Task card exists, but implementation has not started.
- `in_progress`: The implementation agent is actively working on the task.
- `implemented`: Implementation is finished and the handoff is filled, but review has not happened.
- `needs_review`: Waiting for Codex or human review.
- `changes_requested`: Review found required changes.
- `ready_to_merge`: Review and validation are acceptable; waiting for the human merge decision.
- `done`: The task has been merged, closed, or explicitly accepted by the human user.
- `blocked`: Progress needs a human decision, external resource, or task split.
- `abandoned`: The task was intentionally dropped.

Status transitions:

- `planned` -> `in_progress`
- `in_progress` -> `implemented`
- `implemented` -> `needs_review`
- `needs_review` -> `changes_requested`
- `changes_requested` -> `in_progress`
- `needs_review` -> `ready_to_merge`
- `ready_to_merge` -> `done`
- Any non-terminal status -> `blocked`
- `blocked` -> `planned` or `in_progress`
- Any non-terminal status -> `abandoned`

Rules:

- Only the implementation agent should move `in_progress` to `implemented`, and only after filling the Handoff section.
- Only review can move a task to `ready_to_merge`.
- Only the human user can move a task to `done`.
- `done` and `abandoned` are terminal unless the human user explicitly reopens the task.

## Branch

<type>/<short-name>

## Goal

<One sentence describing the desired outcome.>

## Background

<Why this task matters, what files or behavior are relevant, and what should not change.>

## Allowed files

- `<path>`

## Forbidden files

- `<path>`

## Acceptance criteria

- <Observable requirement>
- <Observable requirement>

## Validation commands

```bash
<command>
```

## Review required

Codex review required before merge.

Focus on:

- Whether the task card was satisfied.
- Whether modified files stayed in scope.
- Whether experiment semantics changed.
- Whether validation is sufficient.
- Whether follow-up tasks are needed.

## Handoff

To be filled by the implementing agent.

### Files changed

- 

### What changed

- 

### Commands run

```bash

```

### Validation result

Not run yet.

### Assumptions

- 

### Risks

- 

### Suggested next step

- 
