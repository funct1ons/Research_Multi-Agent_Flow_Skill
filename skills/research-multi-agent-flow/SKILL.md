---
name: research-multi-agent-flow
description: Use when managing personal research codebases with Codex, execution agents, Git branches/worktrees, task cards, handoff notes, experiment records, or multi-agent coding/review workflows.
---

# Research Multi-Agent Flow

## Overview

Use this skill to keep personal research projects reproducible while coordinating Codex and execution agents through files and Git. The core rule is: stable policy belongs in `AGENTS.md`, concrete work belongs in `tasks/*.md`, execution handoff belongs in the task card, research facts belong in `experiments/`, and the human user owns final research and merge decisions.

## Roles

| Actor | Default responsibility |
| --- | --- |
| Human user | Research priorities, task scope approval, experiment semantics, merge decisions, paper claims |
| Codex | Planning, task decomposition, task cards, architecture reasoning, read-only review, experiment-semantics checks, results analysis, documentation |
| Execution agent | Local implementation, tests, config/script edits, lint/type/test repair loops, handoff notes. This may be OpenCode, Reasonix, a low-cost Codex subagent, or another local coding agent. |
| Git | Branch/worktree isolation, diff review, commit history, tags, experiment commit hashes |

When asked to review, default to read-only review. Do not edit code during review unless the user explicitly asks for implementation.

## Workflow

1. Clarify the goal enough to make one small task.
2. Create or update a task card in `tasks/*.md`.
3. Confirm the task has a valid status, branch or worktree, allowed files, forbidden files, acceptance criteria, validation commands, review checklist, and handoff section.
4. Move the task from `planned` to `in_progress` when one execution agent starts executing the task card.
5. Require the execution agent to fill the handoff section before moving the task to `implemented`.
6. Inspect `git status` and `git diff` before trusting any agent report.
7. Use Codex to review the task card, handoff, and diff without modifying code.
8. If review finds issues, move the task to `changes_requested` and create a follow-up task card or send a bounded fix request back to the execution agent.
9. Move the task to `ready_to_merge` only after review and validation are acceptable; let the human user decide whether to commit, squash-merge, tag, defer, or mark it `done`.
10. If results or paper claims are affected, record config, command, commit hash, metrics, and notes under `experiments/`.

Use `blocked` when progress needs a human decision, external resource, or task split. Use `abandoned` only when the task is intentionally dropped. Only the human user can mark a task `done`.

Codex remains the default planner and reviewer. If a low-cost Codex subagent is used as the execution agent, the reviewing Codex instance must rely on the task card, handoff, validation output, and `git diff`, not on trust in the subagent's report.

## Task Sizing

Prefer small tasks:

- Fix one reproducibility bug.
- Add one experiment configuration family.
- Add one regression test.
- Create one table or plotting script.
- Refactor one bounded module without behavior changes.

Split broad requests such as "improve the training system", "clean up all experiments", "optimize everything", or "make the paper better" into task cards before implementation.

## Review Checklist

During Codex review, check:

- Did the implementation satisfy the task card?
- Were forbidden files changed?
- Did modified files stay within the allowed scope?
- Did the agent silently change baseline experiment semantics?
- Were validation commands run, and are results reported clearly?
- Are tests or dry-runs sufficient for the risk level?
- Were results invented, hard-coded, or written without source experiment records?
- Are paper-facing claims supported by recorded metrics?
- Should remaining risk become a follow-up task instead of expanding the current task?

High-risk areas require review before merge: data processing, training loops, evaluation scripts, metrics, baseline configs, experiment configs used for paper results, paper tables, and paper figures.

## Experiment Policy

For important experiments, require an experiment directory containing:

- `config.yaml` or copied configs
- `command.txt`
- `commit.txt` from `git rev-parse HEAD`
- `metrics.json` or `metrics.csv`
- `notes.md`

Do not invent results. Do not edit paper tables or figures unless the source experiment directory is specified.

## Resources

- Read `references/task-card-template.md` when creating a task card.
- Read `references/project-files.md` when adding `AGENTS.md`, `tasks/README.md`, `LOG.md`, or experiment-record conventions to a repository.
- Read `references/review-prompts.md` when the user asks for execution-agent handoff, Codex diff review, or follow-up task prompts.
