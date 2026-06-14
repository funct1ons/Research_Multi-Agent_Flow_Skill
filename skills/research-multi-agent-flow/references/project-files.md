# Project File Conventions

Use these conventions when adding Research Multi-Agent Flow to a repository.

## `AGENTS.md`

Keep this file short and stable. Include project-specific rules, commands, directories, and agent roles. Do not put daily tasks, long research notes, or experiment results here.

Minimum sections:

- Project type
- Workflow files
- Agent roles
- Task card policy
- Validation policy
- High-risk areas
- Experiment policy
- Merge policy

## `tasks/README.md`

Explain how task cards are named and used. Link to or embed the task card template.

## `LOG.md`

Use this as the human-readable research log. Record completed work, unvalidated work, risks, and next steps. Avoid making it an agent transcript.

## `experiments/`

Each important experiment should record:

- copied config or config directory
- exact command
- commit hash
- metrics
- notes

Commit lightweight records. Ignore large raw data, checkpoints, caches, and generated artifacts unless the project explicitly needs them versioned.

## `.gitignore`

Ignore environments, caches, notebooks checkpoints, logs, raw datasets, model checkpoints, and large run outputs. Do not ignore source code, configs, scripts, tests, task cards, lightweight experiment notes, commands, metrics, or commit hashes.
