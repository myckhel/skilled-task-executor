# Safety And Checkpoints

This workflow must protect user work, external systems, and task history. Checkpoints are deliberate pauses before meaningful state changes unless the user has explicitly authorized an autonomous path and the action is low-risk.

## Connector Credential Protection

Use task-platform integrations only through MCP, app, or connector tools exposed by the runtime. Never read connector configuration, environment variables, secret stores, keychains, authentication caches, or logs to obtain an access token. Never copy or pass such credentials to `curl`, a generic HTTP client, an SDK, browser automation, custom scripts, source files, or chat.

If the required tool is not callable, fails authentication, or lacks permission, stop the platform operation and route the user to the supported connection, installation, restart, or permission flow. A configured server is not evidence that its tools are available, and direct API access is not a fallback for a missing MCP capability.

## Worktree Protection

Before editing, inspect git status when git is available.

If uncommitted changes exist:

- preserve them;
- determine whether they are related to the task;
- do not overwrite, stage, commit, discard, reset, or clean unrelated changes without explicit authorization;
- do not use `git reset --hard` or `git clean -fd` unless the user explicitly requested that exact destructive operation.

If unrelated changes are present, continue only when the task can be implemented without touching or staging them. Otherwise pause and explain the conflict.

## Branch Checkpoint

Before creating or switching branches in interactive mode, show:

- current branch;
- whether the worktree is clean;
- recommended branch name when clear;
- options to create the recommended branch, use another branch, continue on current branch, or stop.

In autonomous mode, branch creation is allowed only when the user explicitly authorized end-to-end implementation and the worktree state is safe. Do not switch away from a branch with unrelated uncommitted work unless explicitly authorized.

## Implementation Checkpoint

Pause before implementation when:

- task requirements conflict or are materially ambiguous;
- linked artifacts are inaccessible and required;
- the apparent change is much larger than the task wording suggests;
- implementation would touch sensitive, destructive, security-critical, billing, legal, medical, or permission-sensitive behavior;
- existing code appears to already satisfy the task.

## Validation Checkpoint

Do not mark validation complete until actual commands or tools confirm results. If tests cannot run because dependencies, services, credentials, fixtures, or permissions are missing, report them as blocked or unavailable.

For user-facing changes, provide a manual test checklist when automated coverage is incomplete or unavailable.

## Commit Checkpoint

Before committing in interactive mode, summarize:

- changed files and behavioral summary;
- validation results;
- skipped, unavailable, or failing checks;
- exact suggested commit message.

In autonomous mode, committing is allowed only when the user explicitly authorized end-to-end implementation or committing. Never include unrelated changes in the commit.

Use conventional commits when they fit the repository and task:

- `feat(scope): summary`
- `fix(scope): summary`
- `refactor(scope): summary`
- `test(scope): summary`
- `docs(scope): summary`

## Push And PR Checkpoints

Before pushing in interactive mode, identify the branch and remote. In autonomous mode, push only when the user explicitly authorized push or PR creation.

Before creating a PR, summarize:

- source branch and target branch;
- PR title;
- PR body;
- validation status;
- task reference when available.

If `pr.create` is unavailable, prepare the title and body and stop there. Never claim a PR exists without a returned URL, number, or provider reference.

## Task Update And Transition Checkpoints

Task comments, field updates, and transitions are external state changes. In interactive mode, ask before writing unless the user specifically requested task synchronization.

In autonomous mode, progress comments and appropriate non-final transitions are allowed when the user asked to complete the tracker workflow, but final completion still requires confidence that the task is actually complete. Merging, closing, or marking done requires explicit authorization unless the user clearly requested that final outcome.

Always inspect available statuses or transitions before moving a task. Do not infer platform status names.

## Hard Stops

Stop and ask for user input when:

- required task content is missing;
- no required task list/search/read capability or acceptable pasted task details are available;
- platform access would require extracting credentials or bypassing a connector tool;
- destructive git cleanup is needed;
- unrelated user changes block implementation;
- credentials, permissions, or external approvals are required;
- the next action would merge, delete, close, submit, or transmit sensitive data without explicit authorization.
