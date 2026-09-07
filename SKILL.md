---
name: skilled-task-executor
description: Search, list, read, review, or execute software-development tasks from connected task platforms using a tool-agnostic workflow. Use when a request names a task platform, task, ticket, issue, assigned work item, or project-tracker item; do not use for ordinary coding requests that have no task-management context.
metadata:
  short-description: Execute tracker tasks with agent checkpoints
---

# Skilled Task Executor

Use this skill when the user asks the agent to access a task platform or work from an existing task-management item, such as listing tasks in a project, reading an assigned item, implementing a ticket, or reviewing an issue.

The central rule is: **the skill owns the workflow; connected tools own the integration.** Do not hardcode platform names, status names, branch names, MCP tool names, source-control providers, CI providers, or test frameworks unless the user or repository makes them explicit.

## Operating Modes

- `interactive`: Default. Pause before meaningful state-changing actions, including branch creation, task updates, commits, pushes, PR creation, merges, and completion transitions.
- `autonomous`: Use only when the user explicitly asks for end-to-end execution. Continue through authorized low-risk phases, but still pause for ambiguity, destructive actions, missing capabilities, external transmission that needs approval, merges, or permission-sensitive actions.
- `review-only`: Use when the user asks to review, inspect, analyze, estimate, or assess a task without implementation. Do not modify code, git state, PRs, or task state.

## Required Start

Before answering or acting on task-platform data:

1. Identify the requested platform, scope, and operation: list, search, read, review, or execute.
2. Inspect the runtime's callable tools, apps, MCPs, or connector registry. Do not infer availability from the platform name, a configuration file, stored credentials, or this skill's examples.
3. Map the matching connected integration to the required conceptual capability and invoke the runtime-exposed MCP or connector tool to fulfill the user's objective. Do not stop after describing which tool could be used.
4. If the integration exists but is disconnected, disabled, or unauthenticated, ask the user to connect or sign in to it.
5. If the integration is absent, use the host's connector or plugin discovery and installation-suggestion mechanism when available. Recommend the integration for the platform the user named and prompt the user to install or enable it. Do not install anything without the authorization required by the host.
6. If no installation mechanism exists, name the missing platform capability and ask the user to install a compatible connector. Pasted task details are a fallback for reading or implementing a known task, but they cannot satisfy a request to list or search live platform data.

Live task-platform requests require a matching callable capability: `task.list` or `task.search` for collections, and `task.read` for a known item. Stop after the connection or installation prompt when the capability is unavailable. Do not answer from memory, fabricate platform data, or substitute public web browsing for a private task connector unless the user explicitly requests that route.

An MCP configuration entry or access token is not a callable capability. Never read, extract, copy, reveal, or reuse connector credentials to call the platform with `curl`, an HTTP client, an SDK, browser automation, or custom code. If the MCP is configured but its tools are not exposed or a tool call fails, report that connector state and use the host's connection or installation flow; do not bypass the MCP.

Read [references/capability-contract.md](references/capability-contract.md) when capability availability, platform mapping, or graceful degradation matters.

## Workflow

For implementation work, follow the progressive workflow in [references/execution-workflow.md](references/execution-workflow.md):

1. Discover the platform connector and use it to list, search, or read the requested task data.
2. Understand requirements, acceptance criteria, constraints, linked artifacts, and ambiguity.
3. Inspect the repository, current branch, git status, recent related work, likely files, and validation commands.
4. Produce or confirm an implementation plan before edits when the task is non-trivial.
5. Use the branch checkpoint policy before creating or changing branches.
6. Implement only within the task scope while preserving unrelated user changes.
7. Run available validation and record truthful results.
8. Offer manual testing when automated validation is insufficient or the feature is user-facing.
9. Review the diff before commit.
10. Use checkpoints for commit, push, PR creation, task sync, and completion.

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

When Asana is named and its MCP connector needs installation, connection, or troubleshooting, read [examples/asana.md](examples/asana.md) and surface the official setup guide linked there when the host cannot complete recovery directly.

Use the scenario checks in [tests/scenarios.md](tests/scenarios.md) to validate changes to this skill's behavior.
