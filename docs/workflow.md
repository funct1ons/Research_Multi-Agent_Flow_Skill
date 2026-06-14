# Research Multi-Agent Flow

This workflow coordinates Codex and OpenCode in a personal research repository without letting either agent become the project owner.

## Core Idea

Use files and Git as the coordination layer:

- `AGENTS.md` stores stable project rules.
- `tasks/*.md` stores concrete task cards.
- The implementation agent writes a handoff note.
- Codex reviews task cards, handoff notes, and diffs.
- Git records the facts.
- `experiments/` records research outputs and commit hashes.

## Default Loop

1. Use Codex to turn a research goal into one small task card.
2. Review the task card as the human research owner.
3. Create a short-lived branch or worktree.
4. Give the task card to OpenCode for implementation.
5. Require OpenCode to update the handoff section.
6. Inspect `git status` and `git diff`.
7. Ask Codex for read-only review.
8. Send bounded follow-up work back to OpenCode if needed.
9. Commit and squash-merge only after validation.
10. Record important experiment results with config, command, commit hash, metrics, and notes.

## Minimal Project Files

Add these files to downstream research repositories:

```text
AGENTS.md
tasks/
LOG.md
experiments/
.gitignore
```

Use `docs/` for longer workflow notes or paper-specific policies.

## Example

Goal: add dropout ablation configs.

Codex creates `tasks/YYYY-MM-DD_add-dropout-ablation.md` with:

- allowed files: `configs/`, `tests/test_config.py`
- forbidden files: `src/model.py`, `src/data.py`, `paper/`
- acceptance criteria: add dropout `0.0`, `0.1`, `0.3` configs and keep non-dropout baseline settings unchanged
- validation: config tests and a dry-run command

OpenCode implements the task and fills the handoff. The human checks the diff. Codex reviews whether only dropout changed, whether seeds are explicit, and whether a multi-seed follow-up is needed. The human decides whether to merge.
