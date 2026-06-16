# AGENTS.md

## Project Type

This repository packages the `research-multi-agent-flow` skill and supporting project templates for personal research codebases. Prioritize concise instructions, reproducibility, minimal diffs, clear experiment semantics, and traceable results.

## Workflow

Use the Research Multi-Agent Flow for non-trivial changes.

Project files:

- `skills/research-multi-agent-flow/` contains the reusable Codex skill.
- `tasks/` contains task cards and templates.
- `docs/workflow.md` explains the human-facing workflow.
- `LOG.md` records project progress and release notes.

## Agent Roles

Codex is primarily used for planning, task-card creation, documentation, skill review, repository maintenance, and high-risk reasoning. Codex should default to read-only review when asked to review.

The execution agent may be OpenCode, Reasonix, a low-cost Codex subagent, or another local coding agent. Execution agents are used for local implementation, tests, config/script edits, and bounded repair loops. Execution agents must follow task cards strictly when a task card exists.

When a Codex subagent is used as the execution agent, the planning/reviewing Codex instance must review only from task cards, handoff notes, validation output, and `git diff`, not from trust in the subagent's report.

## Task Card Policy

Concrete work should be described in `tasks/*.md` when it is more than a trivial edit. If a task card exists, follow its goal, allowed files, forbidden files, acceptance criteria, validation commands, and handoff requirements.

If implementation requires files outside the allowed list, stop and report why before editing them.

## Validation

Before claiming completion, run relevant validation. For this repository, use:

```powershell
python C:\Users\admin\.codex\skills\.system\skill-creator\scripts\quick_validate.py skills\research-multi-agent-flow
git status --short --branch
```

## Merge Policy

Do not make substantial changes directly on `main` in normal downstream research projects. In this repository, initialize and maintain a clean `main` history suitable for publishing to GitHub. Only the human user decides when to push or publish.
