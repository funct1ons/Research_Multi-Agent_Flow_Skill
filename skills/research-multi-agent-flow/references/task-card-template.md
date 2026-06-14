# Task Card Template

Use this template for `tasks/YYYY-MM-DD_short-name.md`.

```markdown
# Task: <task name>

## Owner agent

OpenCode

## Status

planned

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
```

Keep each task card small. If the allowed files list becomes broad, split the task.
