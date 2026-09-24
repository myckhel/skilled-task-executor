---
name: skilled-task-executor
description: Draft, create, break down, search, review, or execute software-development tasks using a tool-agnostic workflow. Use for PM task planning, ticket creation, or requests involving a task platform or tracker item; do not use for ordinary coding requests without task-management context.
metadata:
  short-description: Plan, create, and execute tracker tasks
---

# Skilled Task Executor

Use this skill to plan or create task-management items, access a task platform, or work from an existing task, ticket, or issue.

The central rule is: **the skill owns the workflow; connected tools own the integration.** Do not hardcode platform names, status names, branch names, MCP tool names, source-control providers, CI providers, or test frameworks unless the user or repository makes them explicit.

## Roles And Operating Modes

- `project-manager`: Route "create a task", "break this feature into tickets", or "act as PM" here. Read [references/task-creation-workflow.md](references/task-creation-workflow.md) before drafting or publishing tasks. Inspect relevant repository context without code or git mutations; begin implementation only if also requested.
- `executor`: Existing task discovery, review, and implementation behavior. Ordinary coding requests do not imply task creation.

Roles describe the work; modes govern execution. PM role selection alone does not authorize publication. Draft/plan requests remain unpublished; explicit, sufficiently scoped create requests authorize the requested publication without a redundant confirmation.

- `interactive`: Default. Pause at state-changing checkpoints unless the action is already explicitly authorized. In particular, resolved task creation requests do not need a second approval. Existing git, PR, and task-sync checkpoints still apply.
- `autonomous`: Use only when the user explicitly asks for end-to-end execution. Continue through authorized low-risk phases, but still pause for ambiguity, destructive actions, missing capabilities, external transmission that needs approval, merges, or permission-sensitive actions.
- `review-only`: Use when the user asks to review, inspect, analyze, estimate, or assess a task without implementation. Do not modify code, git state, PRs, or task state.

## Required Start

Drafting from supplied context requires no connector. For live task-platform data or publication:

1. Identify the role, mode, requested platform, scope, and operation: draft, create, list, search, read, review, or execute.
2. Inspect the runtime's callable tools, apps, MCPs, or connector registry. Do not infer availability from the platform name, a configuration file, stored credentials, or this skill's examples.
3. Map the matching connected integration to the required conceptual capability and invoke the runtime-exposed MCP or connector tool to fulfill the user's objective. Do not stop after describing which tool could be used.
4. If the integration exists but is disconnected, disabled, or unauthenticated, ask the user to connect or sign in to it.
5. If the integration is absent, use the host's connector or plugin discovery and installation-suggestion mechanism when available. Recommend the integration for the platform the user named and prompt the user to install or enable it. Do not install anything without the authorization required by the host.
6. If no installation mechanism exists, name the missing platform capability and ask the user to install a compatible connector. Pasted task details are a fallback for reading or implementing a known task, but they cannot satisfy a request to list or search live platform data.

Live operations require matching capabilities: `task.list` or `task.search` for collections, `task.read` or a complete readable search record for a known item, and `task.create` for publication (`task.subtask.create` for children). Read/search is not a prerequisite for creating into a resolved destination. If access is missing, stop live operations and provide setup recovery; PM work can still return unpublished drafts. Never fabricate platform data or substitute public web browsing unless explicitly requested.

An MCP configuration entry or access token is not a callable capability. Never read, extract, copy, reveal, or reuse connector credentials to call the platform with `curl`, an HTTP client, an SDK, browser automation, or custom code. If the MCP is configured but its tools are not exposed or a tool call fails, report that connector state and use the host's connection or installation flow; do not bypass the MCP.

Read [references/capability-contract.md](references/capability-contract.md) when capability availability, platform mapping, or graceful degradation matters.

## Workflow

For implementation work, follow the progressive workflow in [references/execution-workflow.md](references/execution-workflow.md):

1. Discover the platform connector and use it to list, search, or read the requested task data.
2. Understand requirements, acceptance criteria, constraints, linked artifacts, and ambiguity.
3. Inspect the repository, current branch, git status, recent related work, likely files, and validation commands.
4. Produce or confirm an implementation plan before edits when the task is non-trivial.
5. Use the branch checkpoint policy before creating or changing branches.
6. Immediately before the first implementation edit, run the implementation-start gate: inspect current task state and available transitions, invoke the MCP tool to move the active task to the platform's in-progress equivalent, and confirm success. An implementation request authorizes this specific transition in interactive and autonomous modes unless the user forbids tracker changes. If already in-progress, record that without another write. Resolve failures or explicitly report unavailable sync before editing; never silently skip this gate. See [references/execution-workflow.md](references/execution-workflow.md#implementation-start-gate).
7. Implement only within the task scope while preserving unrelated user changes.
8. Run available validation and record truthful results.
9. Offer manual testing when automated validation is insufficient or the feature is user-facing.
10. Review the diff before commit.
11. Use checkpoints for commit, push, PR creation, task sync, and completion.

Use [references/safety-and-checkpoints.md](references/safety-and-checkpoints.md) for approval boundaries and unsafe actions.

## Task Sync

Maintain a trace from task requirements to code changes, validation, git state, PR state, and task updates. Use semantic task states rather than platform-specific statuses. Never claim a task, PR, branch, commit, transition, comment, push, or validation result exists unless an available tool or local command confirmed it.

Read [references/task-state-and-sync.md](references/task-state-and-sync.md) when updating task comments, mapping states, recording execution state, or reporting degraded sync.

## Safety Rules

Never:

- invent a task ID, task content, capability, validation result, commit, PR, or task update;
- claim a platform was checked unless a matching connected tool was actually invoked;
- inspect MCP configuration, environment variables, secret stores, or authentication caches to obtain platform credentials;
- bypass a platform MCP by sending its credentials through `curl`, an HTTP client, an SDK, browser automation, or custom code;
- assume Asana, Linear, Jira, GitHub, GitLab, Playwright, or any specific tool is available;
- assume task status names or workflow transitions;
- mark a task complete merely because code compiles;
- overwrite, reset, clean, discard, or include unrelated user changes without explicit authorization;
- create branches, commits, pushes, PRs, task comments, or task transitions outside the selected mode's checkpoint policy;
- merge a PR, delete a branch, close a task, or perform destructive actions unless the user explicitly authorized that exact action.

## Examples And Validation

Use examples only as platform illustrations, not as built-in integrations:

- [examples/asana.md](examples/asana.md)
- [examples/linear.md](examples/linear.md)
- [examples/jira.md](examples/jira.md)
- [examples/generic-compatible-mcp.md](examples/generic-compatible-mcp.md)
- [examples/project-manager.md](examples/project-manager.md)

When Asana is named and its MCP connector needs installation, connection, or troubleshooting, read [examples/asana.md](examples/asana.md) and surface the official setup guide linked there when the host cannot complete recovery directly.

Use the scenario checks in [tests/scenarios.md](tests/scenarios.md) to validate changes to this skill's behavior.
