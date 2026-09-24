# Capability Contract

This skill uses conceptual capabilities, not concrete MCP tool names. The agent must discover which connected tools are available in the current runtime and map them to these concepts before acting.

## Runtime Connector Handshake

For every request involving live task-platform data:

1. Extract any explicitly named platform and the requested operation.
2. Inspect the runtime's callable tool catalog or use its tool-discovery mechanism for that platform and operation.
3. Treat a capability as available only when a matching tool is callable in the current session.
4. Invoke the matching tool and use its returned data to complete the objective.

Discovery alone does not satisfy the request. For example, "list tasks in Asana" requires calling an Asana-capable list or search operation and returning its results.

When no matching tool is callable, distinguish these cases:

- **Installed but disconnected or unauthenticated:** prompt the user to connect, enable, or sign in to the integration.
- **Not installed:** use a host-provided plugin or connector discovery mechanism to find the named platform integration. If the host supports an installation suggestion or install prompt, invoke it; otherwise tell the user exactly which compatible connector is needed.
- **Installed but insufficient permissions:** report the missing scope or permission and ask the user to grant it or choose a narrower operation.
- **No identifiable platform:** ask which task platform or connected task source should be used only when the prompt and available context cannot resolve it.

Do not silently fall back to web search, browser automation, local files, or invented data for private platform requests. Use an alternate source only when the user explicitly authorizes it and it can truthfully satisfy the objective.

## MCP Tool Boundary

Only a task-platform operation exposed as a callable tool in the current runtime satisfies the capability contract. A server entry in MCP configuration, an access token, an environment variable, or an authentication cache does not.

For task-platform operations:

- invoke the exposed MCP, app, or connector tool directly;
- use only data returned by that tool as evidence that the platform was queried or changed;
- never inspect MCP configuration or secret-bearing files to recover credentials;
- never copy, print, persist, or place connector credentials in commands, logs, chat, temporary files, or source code;
- never use connector credentials with `curl`, `wget`, a generic HTTP client, an SDK, browser automation, or custom code to reproduce the platform call;
- never bypass a tool error with a direct API request.

If configuration appears to exist but no matching tools are callable, treat the connector as unavailable or incompletely loaded. Ask the user to enable, reconnect, reinstall, or restart it through the host's supported integration flow. If a callable tool returns an authentication or permission error, report that result and request the corresponding connection or permission fix.

Use direct platform APIs only when the user explicitly requests a separate direct-API workflow; do not obtain credentials from MCP configuration, and do not represent that workflow as MCP-backed.

## Core Principle

The skill decides what should happen next. Connected task, git, source-control, browser, CI, and validation tools decide how an operation is performed.

Do not require a specific platform when an equivalent capability is available through another tool.

## Task Management

Required for a tracker-backed workflow:

- `task.list`: enumerate tasks in a workspace, project, team, section, assignee view, or other collection.
- `task.read`: read a known task by URL, ID, mention, or selected context.
- `task.search`: find candidate tasks from a query, assignee, project, or natural-language reference.
- `task.create`: publish a task in a resolved destination with supported fields.
- `task.subtask.create`: create a child under a confirmed parent; this can map to a creation tool with a parent input.

The required capability depends on the objective. Collection requests need `task.list` or scoped `task.search`. Reading a known item needs `task.read` or a complete readable search record. Publishing needs `task.create` and child creation support for requested hierarchies; read/search is not a universal prerequisite. Drafting from supplied context requires no platform capabilities. If a required live capability is unavailable, stop that operation and follow connector recovery; PM drafts can still be completed locally in the response.

For creation, discover destination and field metadata through exposed workspace, project, user, and schema tools where available. Map these to conceptual `workspace.list`, `project.search`, `project.read`, `user.search`, and `task.fields.read` as appropriate. These are optional discovery aids, not mandatory tool names: unambiguous provided identifiers and tool schemas may supply enough information. Never infer valid IDs, required fields, or allowed values without evidence. Use scoped task search for duplicate checks when supported; disclose its absence without blocking otherwise valid creation.

Recommended:

- `task.comment`: add structured progress updates.
- `task.update`: edit fields or metadata.
- `task.transition`: move the task through its workflow.
- `task.assign`: assign or reassign the task when requested.
- `task.attachment.read`: inspect linked or attached task artifacts.

## Repository And Git

Local command equivalents are acceptable when available:

- `git.status`: inspect current worktree state.
- `git.diff`: inspect pending changes.
- `git.branch.current`: identify the current branch.
- `git.branch.create`: create a task branch.
- `git.branch.checkout`: switch branches.
- `git.commit`: create a commit.
- `git.push`: push a branch or commit.

Branch, commit, and push actions are state-changing and must follow the selected mode and checkpoint policy.

## Code Hosting And Review

Use whatever provider exposes equivalent pull-request or merge-request behavior:

- `pr.read`: inspect an existing review request.
- `pr.create`: create a pull request or merge request.
- `pr.update`: update title, body, labels, reviewers, or metadata.
- `pr.comment`: add comments to a PR.
- `pr.merge`: merge a PR.

If `pr.create` is unavailable, prepare the title and body for the user and report that automatic PR creation is unavailable. Never claim a PR was created without a returned PR reference.

## Validation

Detect validation from repository conventions, package scripts, project files, CI config, or explicit user instructions:

- `lint.run`
- `typecheck.run`
- `test.unit`
- `test.integration`
- `test.e2e`
- `build.run`
- `ci.read`

Run the strongest relevant validation that is locally available and proportionate to the change. If a category is unavailable, record it as unavailable rather than failed or passed.

## Browser And Manual Verification

Browser capabilities may support user-facing validation:

- `browser.open`
- `browser.interact`
- `browser.screenshot`
- `browser.console`

Use browser validation when the task affects UI behavior and the environment supports it. If browser or E2E support is missing, offer a manual test checklist when the selected mode requires user-facing confidence.

## Graceful Degradation

Missing recommended or optional capabilities should narrow the workflow, not fail it. Report the adaptation clearly:

- Read-only task tools: implement from the task but do not update or comment on it.
- No transition support: add a comment if possible and report that status transition is unavailable.
- No PR support: prepare PR metadata and stop after commit or push according to user authorization.
- No E2E support: run available lower-level validation and offer manual checks.

Missing required live capabilities prevent completion of the platform operation. Route to connection, installation, or permission recovery instead of claiming success. Pasted context can support implementation or PM drafts, but cannot replace live list, search, create, update, or transition operations. See [task creation](task-creation-workflow.md) for unpublished drafts and hierarchy alternatives.

Never hide degraded behavior. Completion summaries must distinguish completed actions, skipped actions, unavailable capabilities, and actions awaiting user approval.
