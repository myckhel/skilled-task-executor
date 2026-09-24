# Jira Example

This example illustrates compatible behavior with a Jira-capable task tool. It is not a Jira integration.

## User Request

```text
Use $skilled-task-executor to review PROJ-123 and tell me what needs to change.
```

## Capability Mapping

PM example: "Create a task and subtasks for password reset in the client portal project." Discover valid issue types, required fields, and project/hierarchy constraints before creating anything. Map the exposed tools to `task.create` and `task.subtask.create`; do not assume every project permits subtasks. Use the [PM workflow](../references/task-creation-workflow.md) for drafts, publication, and hierarchy alternatives.

Possible mapping:

- `task.read`: read Jira issue fields, description, comments, links, and acceptance criteria.
- `task.attachment.read`: inspect linked specs or screenshots when available.
- `task.comment`: optional, only if the user asks to update Jira.

## Expected Behavior

The request is review-only. The agent reads the issue and repository context, reports findings, risks, and likely implementation steps, and does not modify code, git state, PRs, or Jira state.
