# Research Multi-Agent Flow Skill

## Version

Current version: `v0.1.1`

This release adds a Chinese user guide with a full daily workflow example, alongside the core Research Multi-Agent Flow skill, task-card templates, review prompts, project-file conventions, and open-source repository metadata.

`research-multi-agent-flow` is a Codex skill for personal research codebases that use Codex, OpenCode, Git, task cards, and experiment records together.

The workflow keeps responsibilities explicit:

- Codex plans, creates task cards, reviews diffs, checks experiment semantics, and helps with documentation.
- OpenCode implements bounded local tasks, runs tests, and writes handoff notes.
- Git isolates work, exposes diffs, records commits, and binds experiment results to code versions.
- The human user owns research judgment, merge decisions, and paper claims.

## Repository Layout

```text
skills/research-multi-agent-flow/
  SKILL.md
  agents/openai.yaml
  references/

tasks/
  README.md
  task-card-template.md

docs/
  workflow.md
  workflow.zh-CN.md

AGENTS.md
LOG.md
LICENSE
```

## Install

Copy or symlink the skill directory into your Codex skills directory:

```powershell
Copy-Item -Recurse skills\research-multi-agent-flow $env:USERPROFILE\.codex\skills\
```

Then invoke it in Codex with:

```text
Use $research-multi-agent-flow to create a task card for my next research coding task.
```

## Use In A Research Repository

1. Add an `AGENTS.md` that points agents to this workflow.
2. Create `tasks/` for task cards.
3. Use short-lived Git branches or worktrees.
4. Let OpenCode implement task cards and write handoff notes.
5. Ask Codex to review task card, handoff, and diff before merge.
6. Record important experiments under `experiments/` with config, command, commit hash, metrics, and notes.

## Documentation

- [Workflow overview](docs/workflow.md)
- [中文工作流指南](docs/workflow.zh-CN.md)

## Validation

Validate the skill structure with:

```powershell
python C:\Users\admin\.codex\skills\.system\skill-creator\scripts\quick_validate.py skills\research-multi-agent-flow
```

## License

MIT. See [LICENSE](LICENSE).
