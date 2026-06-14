# Contributing

Contributions should keep the skill concise and reusable across personal research repositories.

## Guidelines

- Keep `skills/research-multi-agent-flow/SKILL.md` focused on core workflow rules.
- Put longer templates or optional details in `skills/research-multi-agent-flow/references/`.
- Avoid project-specific research notes in the skill.
- Update `README.md` or `docs/workflow.md` when user-facing behavior changes.
- Run validation before submitting changes.

## Validation

```powershell
python C:\Users\admin\.codex\skills\.system\skill-creator\scripts\quick_validate.py skills\research-multi-agent-flow
git status --short --branch
```

If the Python environment lacks `PyYAML`, install it in your preferred environment or run an equivalent frontmatter and file-structure check before opening a pull request.
