# Execution Workflow

This workflow is progressive. The agent gathers context before acting, pauses at checkpoints according to mode, and keeps the user informed about decisions that affect code, git, PRs, or task state.

## Mode Selection

First select the role. Task drafting and creation use the `project-manager` [creation workflow](task-creation-workflow.md); they do not automatically enter repository implementation phases. Existing task execution uses `executor`. For combined requests, begin this workflow after requested publication is confirmed, carrying the created task references forward.

Use `interactive` unless the user clearly requested `autonomous` or `review-only`.

Signals for `autonomous` include: "implement end-to-end", "open a PR", "finish the ticket", or equivalent language that explicitly authorizes a broad implementation workflow. Even in autonomous mode, do not bypass safety rules or permission requirements.

Signals for `review-only` include: "review", "analyze", "estimate", "inspect", "triage", or "assess" when the user does not ask for implementation.

## Discovery And Understanding

1. Identify the platform, task reference or collection scope, and requested operation from the prompt, selected app context, or linked URL.
2. Perform the runtime connector handshake in `capability-contract.md`: inspect callable integrations for the named platform and map the required list, search, or read capability. Do not inspect MCP configuration or credentials as a substitute for tool discovery.
3. If callable, invoke the platform's runtime-exposed MCP or connector tool. If it is disconnected, missing, not exposed, or permission-blocked, prompt for the matching connection, installation, restart, or permission fix and stop until it is available. Never recreate the operation with `curl`, direct HTTP, an SDK, browser automation, or custom code using connector credentials.
4. For list or search requests, return the tool-backed results in the requested scope and do not enter repository or implementation phases unless the user also asked for code work.
5. For a known task, read its title, description, acceptance criteria, comments that affect scope, linked artifacts, current status, assignee, labels, and dependencies when available.
6. Summarize implementation tasks in terms of goal, success criteria, constraints, unknowns, and likely risk.
7. Ask the user only for ambiguities that materially affect the objective and cannot be resolved from task or repository context.

The connector handshake is mandatory even when the platform name is explicit. Naming Asana, Linear, Jira, or another tracker identifies where to look; it does not prove that its integration is installed or connected.

## Repository Assessment

Before editing:

- identify repository path, current branch, base branch when discoverable, and worktree cleanliness;
- inspect recent commits and related code;
- find existing implementation that may already satisfy part or all of the task;
- discover local validation commands from package scripts, project files, CI config, and repository conventions;
- classify the task as new functionality, bug fix, incomplete implementation, regression, refactor, or duplicate.

If uncommitted changes exist, classify them as likely related, likely unrelated, or unclear. Preserve unrelated changes and do not include them in commits without explicit authorization.

## Implementation Planning

For non-trivial tasks, present or internally maintain a short plan covering:

- files or subsystems likely to change;
- behavior to add, fix, or preserve;
- validation to run;
- task sync or PR actions expected later;
- known degraded capabilities.

In interactive mode, get confirmation before implementation if the scope is ambiguous, risky, or larger than the user likely expects. In autonomous mode, proceed when the task and repository context are clear.

## Branch Checkpoint

When implementation requires code changes, inspect the current branch and apply the branch policy from `safety-and-checkpoints.md`.

Recommended branch names should be derived from available task identifiers and title text, but never assume a required naming convention unless the repository has one.

## Implementation-Start Gate

After scope and branch checkpoints are resolved, immediately before the first implementation edit (including tests), synchronize the active task to `implementation_started`. Do not defer this to validation, a progress comment, or the final handoff.

1. Inspect the current task state and available workflow transitions through exposed MCP tools. Discover this capability during task discovery so it is ready at implementation start. Map the platform's actual in-progress equivalent; never assume a literal status name or ID. A supported status-field update or section move can implement conceptual `task.transition`.
2. Treat the user's implementation request as authorization for this specific non-final transition in both interactive and autonomous modes, subject to host permissions and explicit instructions not to change the tracker. It does not authorize other field changes, comments, completion, or reopening a completed task.
3. If the task is already in the mapped in-progress state, record `already_in_progress` and avoid a duplicate write. If it is completed/cancelled, or the mapping is ambiguous, resolve that with the user before editing; do not silently reopen or choose a state.
4. Otherwise invoke the transition tool and confirm the resulting state from its response or a subsequent read. Record the result before proceeding. A written plan, internal state label, or comment saying "In Progress" does not perform the transition.
5. If the connector definitively lacks transition support or the user forbids tracker writes, report this before editing and record `unavailable` or `skipped_by_user`. Continue authorized local implementation with degraded sync. Pasted-only task context follows this same unavailable path.
6. If an available transition fails, is denied, or returns an uncertain result, report it and pause implementation for recovery or the user's explicit choice to continue locally. Re-read uncertain results before any retry; do not blindly repeat a write or claim success.

Apply this gate to each task/subtask when its own implementation starts, including after PM creation. Do not move an entire backlog or untouched sibling tasks into progress. On resumption, check the active task again; an already-in-progress task needs no repeated transition. Read-only discovery, planning, review, and PM creation alone do not trigger the gate.

## Implementation

Begin edits only after the implementation-start gate has a confirmed or explicitly disclosed degraded outcome.

Make the smallest coherent code changes that satisfy the task. Follow repository patterns, tests, style, and ownership boundaries. Avoid unrelated refactors. Keep task-to-code traceability in mind: every meaningful change should map back to a task requirement or validation need.

If new information contradicts the plan, pause in interactive mode or proceed conservatively in autonomous mode only when the correction is low-risk and still within task scope.

## Validation

Run available validation in an order appropriate for the project, typically:

1. focused tests for changed behavior;
2. typecheck or static validation;
3. lint when relevant;
4. build when needed;
5. integration or E2E tests when available and proportionate.

Record each result as pass, fail, skipped by choice, unavailable, or blocked. If validation fails, investigate and fix within scope before moving to commit or PR checkpoints.

## Manual Testing

For user-facing features, provide a concise manual test checklist when automated validation does not fully cover the behavior or when the task explicitly requires manual QA.

If the user reports failure, treat it as a new validation failure: investigate, fix, rerun relevant validation, and ask for retest when needed.

## Review, Commit, Push, PR

Before committing, review the diff and summarize:

- changed files and behavioral impact;
- validation results;
- any skipped or unavailable checks;
- suggested commit message.

Then follow checkpoint policy for commit, push, and PR creation. PR descriptions should include task link or ID when available, summary, validation, risks, and follow-ups.

## Task Synchronization

Use `task-state-and-sync.md` to add comments, update fields, or transition semantic state when supported and allowed. If a task update is unavailable or not authorized, report that truthfully.

This handoff phase does not replace the implementation-start transition, which must have been handled before edits.

## Completion

Final summaries must distinguish:

- implementation completed;
- validation completed and results;
- branch, commit, push, and PR references confirmed by tools;
- task comments or transitions confirmed by task tools;
- skipped, unavailable, blocked, or user-pending actions.
