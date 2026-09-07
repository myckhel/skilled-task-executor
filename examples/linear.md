# Linear Example

This example illustrates compatible behavior with a Linear issue tool. It is not a built-in Linear adapter.

## User Request

```text
Use $skilled-task-executor to pick up LIN-248 and open a PR when it is ready.
```

## Capability Mapping

Possible mapping:

- `task.read`: read the Linear issue by key or URL.
- `task.comment`: add progress or PR-ready updates.
- `task.transition`: move the issue to a workflow state that matches `implementation_started` or `ready_for_review`.
- `pr.create`: create a pull request through an available code-hosting tool.

## Expected Behavior

Because the user asked to open a PR, the agent may treat the request as autonomous for low-risk implementation, validation, commit, push, and PR preparation. It still stops for ambiguity, destructive actions, unrelated user changes, unavailable permissions, or merge/completion actions.
