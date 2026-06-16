# Review and Handoff Prompts

## Ask An Execution Agent To Implement

```text
Please read `tasks/<task-card>.md` and strictly execute the task card.

Requirements:
- Modify only allowed files.
- Do not modify forbidden files.
- If the task requires files outside the allowed list, stop and explain why.
- Run the validation commands that are available in this environment.
- Fill the Handoff section in the task card when finished.
```

## Ask Codex To Review

```text
Please read `tasks/<task-card>.md` and review the current branch relative to `main`.

Do not modify code. Report:
1. Whether the implementation satisfies the task card.
2. Whether any forbidden or out-of-scope files changed.
3. Experiment-semantics risks.
4. Test or dry-run gaps.
5. Whether you recommend merge, follow-up, or rollback.
6. If follow-up is needed, draft a bounded follow-up task card.
```

## Convert Review Findings To Follow-Up

```text
Convert the review findings into a small execution-agent task card.

Keep scope narrow:
- one goal
- explicit allowed files
- explicit forbidden files
- observable acceptance criteria
- validation commands
```
