# Research Multi-Agent Flow User Guide

This guide explains how to use Research Multi-Agent Flow in a personal research codebase. It is written for the human user who coordinates Codex, execution agents, Git, task cards, and experiment records.

The goal is not to let agents freely edit as much code as possible. The goal is to make every meaningful change bounded, reviewable, reproducible, and traceable to a concrete code version.

## 1. What This Workflow Solves

Personal research repositories are not just software products. They often mix model code, data processing, training scripts, experiment configs, notebooks, paper figures, tables, and partial results.

Common failure modes:

- Code changes continuously, but results are not tied to a specific commit.
- A small bug fix silently changes baseline semantics.
- A notebook produced an important result, but the exact code/config state is lost.
- Multiple agents edit the same repository without a shared protocol.
- A paper table uses a result that has no recorded config, command, or commit hash.

Research Multi-Agent Flow uses files and Git as the coordination layer:

```text
AGENTS.md       stable project rules
tasks/*.md      concrete task cards
execution agent local implementation
handoff         implementation summary and risks
Codex           planning and review
Git             isolation and version history
experiments/    research result records
human user      final research and merge decisions
```

## 2. Roles

### Human User

You remain the research owner. You decide:

- what research question matters
- which task should be done next
- whether experiment semantics are valid
- whether a branch should merge into `main`
- whether a result is strong enough for a paper claim

### Codex

Codex is the default planner and reviewer.

Use Codex for:

- task decomposition
- task-card creation
- diff and handoff review
- experiment-semantics checks
- high-risk reasoning around data processing, training loops, metrics, and paper results
- result summarization and writing support

When Codex is asked to review, it should default to read-only review. It should not edit code unless you explicitly ask it to implement.

### Execution Agent

The execution agent is a role, not a fixed tool. It may be OpenCode, Reasonix, a low-cost Codex subagent, or another local coding agent that can read task cards, edit files, run validation commands, and write handoff notes.

Use execution agents for:

- local implementation
- small bug fixes
- tests
- config generation
- script updates
- lint/type/test repair loops
- bounded follow-up fixes
- handoff notes

Execution agents should not decide experiment design, silently change baseline semantics, or invent results.

If a low-cost Codex subagent is used as the execution agent, the reviewing Codex instance must treat the task card, handoff, validation output, and `git diff` as the source of truth. Do not review based on trust in the subagent's summary.

### Git

Git provides the hard boundary:

- `main` stays stable.
- Each non-trivial task uses a short-lived branch or worktree.
- `git diff` reveals what the agent actually changed.
- commits record accepted states.
- commit hashes bind experiment results to code versions.

## 3. Required Project Files

Minimal downstream setup:

```text
AGENTS.md
tasks/
LOG.md
experiments/
.gitignore
```

Recommended research project layout:

```text
my-research-project/
  AGENTS.md
  LOG.md
  README.md
  .gitignore

  tasks/
    README.md
    2026-06-16_add-dropout-ablation.md

  src/
  configs/
  scripts/
  tests/
  notebooks/

  experiments/
    2026-06-16_dropout_ablation/
      configs/
      command.txt
      commit.txt
      metrics.csv
      notes.md

  paper/
```

## 4. Task Card Status Model

Task cards use a small status model, not a full project-management system.

Statuses:

- `planned`: The task card exists, but implementation has not started.
- `in_progress`: The execution agent is actively working on the task.
- `implemented`: Implementation is finished and handoff is filled, but review has not happened.
- `needs_review`: Waiting for Codex or human review.
- `changes_requested`: Review found required changes.
- `ready_to_merge`: Review and validation are acceptable; waiting for the human merge decision.
- `done`: The task has been merged, closed, or explicitly accepted by the human user.
- `blocked`: Progress needs a human decision, external resource, or task split.
- `abandoned`: The task was intentionally dropped.

Normal transitions:

```text
planned -> in_progress
in_progress -> implemented
implemented -> needs_review
needs_review -> changes_requested
changes_requested -> in_progress
needs_review -> ready_to_merge
ready_to_merge -> done
```

Escape transitions:

```text
any non-terminal status -> blocked
blocked -> planned / in_progress
any non-terminal status -> abandoned
```

Rules:

- Only the execution agent should move `in_progress` to `implemented`, and only after filling handoff.
- Only review can move a task to `ready_to_merge`.
- Only the human user can move a task to `done`.
- `done` and `abandoned` are terminal unless the human user explicitly reopens the task.

## 5. When To Use The Full Workflow

Small edits do not need a full task card.

Lightweight changes:

- typo fixes
- one-line README clarifications
- formatting changes with no behavior impact

Use the full flow for:

- code behavior changes
- training, evaluation, data processing, or metrics changes
- experiment config changes
- paper tables or figures
- changes that affect baseline comparability
- any task where an execution agent implements and Codex reviews

Rule of thumb: if a change affects experiment semantics or touches more than one important file, create a task card.

## 6. Daily Workflow: From Morning To Merge

This is how to use the workflow during a normal research day.

### Step 1: Return To A Stable Starting Point

Start from `main`:

```bash
git checkout main
git status
```

You want:

```text
nothing to commit, working tree clean
```

Run a small check:

```bash
pytest
```

or:

```bash
python scripts/run_train.py --config configs/debug.yaml
```

This does not need to be a full training run. The goal is to confirm that `main` is basically usable.

### Step 2: Use Codex To Plan The Day

Do not start by asking an agent to edit code. First ask Codex to split the goal into small tasks.

Example prompt:

```text
Please split today's research goal into 2 small tasks suitable for an execution agent.

Each task should include:
- Goal
- Background
- Allowed files
- Forbidden files
- Acceptance criteria
- Validation commands
- Review checklist
```

Codex output should become task cards under `tasks/*.md`.

Review the task card before implementation:

- Is the task too broad?
- Are allowed files too broad?
- Are forbidden files clear?
- Are acceptance criteria observable?
- Are validation commands realistic?

### Step 3: Create A Task Branch

For example:

```bash
git checkout -b exp/dropout-ablation
```

Rules:

- one task, one branch
- one branch, one execution agent editing at a time
- Codex may review the branch, but should not edit it during review

### Step 4: Have The Execution Agent Implement The Task Card

Prompt the execution agent clearly:

```text
Please read `tasks/2026-06-16_add-dropout-ablation.md` and strictly execute the task card.

Requirements:
- Modify only allowed files.
- Do not modify forbidden files.
- If the task requires files outside the allowed list, stop and explain why.
- Run the validation commands available in this environment.
- Fill the Handoff section when finished.
```

The task status moves:

```text
planned -> in_progress
```

After implementation and handoff:

```text
in_progress -> implemented
```

### Step 5: Inspect The Diff Yourself

Do not trust the agent summary first. Inspect the repository:

```bash
git status
git diff
```

Check:

- Did it modify only allowed files?
- Did it modify forbidden files?
- Did it perform a broad refactor?
- Did it change baseline semantics?
- Did it edit paper results?
- Did it generate large files?
- Did it hard-code or invent results?

If the diff is clearly out of scope, ask the execution agent to explain or revert before review.

### Step 6: Ask Codex For Read-Only Review

Move the task:

```text
implemented -> needs_review
```

Ask Codex:

```text
Please read the task card and Handoff, then review the current branch relative to main.

Do not modify code. Report:
1. Whether the implementation satisfies the task card.
2. Whether any out-of-scope files changed.
3. Experiment-semantics risks.
4. Missing validation.
5. Whether you recommend merge.
6. If not, draft a follow-up task card.
```

If Codex finds required changes:

```text
needs_review -> changes_requested
changes_requested -> in_progress
```

If review and validation are acceptable:

```text
needs_review -> ready_to_merge
```

### Step 7: Commit

After review and validation:

```bash
git add .
git commit -m "exp: add dropout ablation configs"
```

Recommended commit types:

- `fix:` bug fix
- `feat:` feature
- `exp:` experiment-related change
- `test:` tests
- `refactor:` refactor
- `paper:` paper-related change
- `docs:` documentation

### Step 8: Merge Back To Main

For personal research projects, squash merge is usually cleaner:

```bash
git checkout main
git merge --squash exp/dropout-ablation
git commit -m "exp: add dropout ablation configs"
```

After merge, the human user may move:

```text
ready_to_merge -> done
```

If you decide not to merge yet, keep the task at `ready_to_merge` and explain why in the task card or `LOG.md`.

### Step 9: Record Experiment Impact

If the task affects results, record the code version:

```bash
mkdir -p experiments/2026-06-16_dropout_ablation
git rev-parse HEAD > experiments/2026-06-16_dropout_ablation/commit.txt
```

Recommended experiment directory:

```text
experiments/2026-06-16_dropout_ablation/
  configs/
  command.txt
  commit.txt
  metrics.csv
  notes.md
```

Example `notes.md`:

```markdown
# Dropout ablation

## Goal

Compare dropout values 0.0, 0.1, and 0.3 under the baseline setup.

## Code version

See `commit.txt`.

## Command

See `command.txt`.

## Notes

This experiment only creates and validates configs. Full training still needs to be run.

## Risks

Need 3-seed average before using the result in a paper claim.
```

### Step 10: End-Of-Day Wrap-Up

Check status:

```bash
git status
```

Update `LOG.md`:

```markdown
## 2026-06-16

### Done

- Added dropout ablation configs.
- Ran config validation and dry-run.
- Merged task into main.

### Not validated yet

- Full training not run.
- No 3-seed average yet.

### Risks

- Need to confirm dropout is actually applied in model construction.

### Next

- Run dropout values 0.0, 0.1, 0.3 with seeds 1, 2, 3.
- Generate summary table.
```

Push when ready:

```bash
git push
```

## 7. Complete Example: Add Dropout Ablation Configs

Goal: compare `dropout = 0.0, 0.1, 0.3`.

Codex creates a task card:

````markdown
# Task: Add dropout ablation configs

## Execution agent

OpenCode

## Status

planned

## Branch

exp/dropout-ablation

## Goal

Add config files for dropout ablation based on the current baseline.

## Background

We want to compare dropout values 0.0, 0.1, and 0.3 while keeping all other key baseline settings unchanged.

## Allowed files

- `configs/`
- `tests/test_config.py`

## Forbidden files

- `src/model.py`
- `src/data.py`
- `scripts/run_train.py`
- `paper/`

## Acceptance criteria

- Add `configs/ablation_dropout_0.0.yaml`
- Add `configs/ablation_dropout_0.1.yaml`
- Add `configs/ablation_dropout_0.3.yaml`
- Keep all non-dropout key settings aligned with baseline.
- Add or update config loading tests if needed.
- Do not generate or invent experiment results.

## Validation commands

```bash
pytest tests/test_config.py
python scripts/run_train.py --config configs/ablation_dropout_0.1.yaml --dry-run
```

## Review required

Codex review required before merge.

Focus on:

- Whether only dropout differs from baseline.
- Whether seed settings are explicit.
- Whether config names are clear.
- Whether 3-seed follow-up is needed.
````

The execution agent implements the task and fills handoff. You inspect the diff and confirm it did not modify `src/model.py`, `src/data.py`, `scripts/run_train.py`, or `paper/`. Codex reviews the task. After review and validation, you commit and squash-merge. After real experiment runs, record config, command, commit, metrics, and notes under `experiments/`.

## 8. Common Mistakes

### Mistake 1: Making `AGENTS.md` Too Long

`AGENTS.md` should contain stable rules. Do not put daily tasks, long paper background, or experiment logs there.

### Mistake 2: Making Task Cards Too Broad

"Improve the whole training system" is not a good task. Split it into small tasks such as "add config loader tests", "add dropout ablation configs", or "fix dry-run argument parsing".

### Mistake 3: Letting Codex Edit During Review

Review should default to read-only. If review finds issues, create a follow-up task card and send it to an execution agent.

### Mistake 4: Recording Results Without Commit Hashes

Important experiments must record:

```bash
git rev-parse HEAD
```

Without this, you cannot later prove which code version produced a paper result.

### Mistake 5: Treating The Execution Agent As The Research Owner

The execution agent executes task cards. It should not decide whether an experiment supports a paper claim. Codex can help review, but you own the research judgment.

## 9. Minimal Version

If you are just starting, follow these rules:

1. Keep `main` stable.
2. Create a task card for every non-trivial task.
3. Use a short-lived branch for each task.
4. Have one execution agent implement the task and write handoff.
5. Inspect `git diff` yourself.
6. Ask Codex for read-only review.
7. Commit and merge only after validation.
8. Record commit hashes for important experiments.

That is the minimum viable Research Multi-Agent Flow.
