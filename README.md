<div align="center">

# Skilled Task Executor

### From task to tested change.

A tool-agnostic coding-agent skill for executing software development tasks from connected task-management platforms.

[![Format: Agent Skill](https://img.shields.io/badge/FORMAT-Agent_Skill-2563EB?style=flat-square)](SKILL.md)
[![Access: MCP Tools](https://img.shields.io/badge/ACCESS-MCP_Tools-0F766E?style=flat-square)](references/capability-contract.md)
[![Modes: 3](https://img.shields.io/badge/MODES-3-DB2777?style=flat-square)](#choose-your-mode)
[![Validation: Scenarios](https://img.shields.io/badge/VALIDATION-Scenarios-525252?style=flat-square)](tests/scenarios.md)

**Read the task. Respect the context. Show the evidence.**

[Get started](#get-started) · [Try a prompt](#try-a-prompt) · [Read the docs](#documentation) · [Troubleshoot](#troubleshooting)

</div>

---

## What it does

Give your coding agent a task reference and a repeatable way to carry it through discovery, implementation, validation, and handoff. Start small by listing tasks, review a ticket before committing to it, or authorize an implementation workflow with checkpoints for external actions.

The skill defines the workflow. Your MCPs and IDE integrations perform the operations.

| You ask for | The skill guides the agent to |
| --- | --- |
| Tasks in a project | Discover and invoke the matching connector, then return live results. |
| A task review | Read requirements and relevant code, report findings, and preserve existing state. |
| An implementation | Understand acceptance criteria, inspect the repository, plan, edit, and validate. |
| A review handoff | Review the diff, follow commit/PR checkpoints, and retain the task reference. |
| A progress update | Use available task tools and report exactly which updates succeeded. |

This repository contains a skill package: instructions, protocol references, examples, and behavioral validation scenarios. It does not ship an MCP server, platform adapter, or integration SDK. Codex packaging is included; use in other agents requires support for loading these instructions and exposing compatible tools.

## Get started

### 1. Install the skill

In Codex, ask the skill installer:

```text
Use $skill-installer to install the skill at the root of
https://github.com/myckhel/skilled-task-executor
with the name skilled-task-executor.
```

For a manual personal installation, clone the package into the skills directory. This command assumes the default Codex home and a destination that does not already exist:

```sh
git clone https://github.com/myckhel/skilled-task-executor.git \
  ~/.codex/skills/skilled-task-executor
```

If you use a custom `CODEX_HOME`, use its `skills/skilled-task-executor` directory instead. Keep the whole package together: `SKILL.md` links to files in `references/`, `examples/`, and `tests/`. Use your host's supported project-local skill location when installing for only one repository.

After installation, try the explicit invocation below on your next turn. If your host has not discovered the skill, refresh its skill list or start a new session.

### 2. Connect your task platform

Install or enable a compatible task-platform MCP or connector in your coding agent's host, then complete its sign-in flow. The platform's tools must be callable in the agent's current session.

For Asana, follow the official [MCP setup guide](https://developers.asana.com/docs/integrating-with-asanas-mcp-server) and the [Asana setup and recovery notes](examples/asana.md#mcp-setup-and-recovery). For other platforms, use the setup flow supported by your connector and host.

Installing this skill does not install a platform integration. An entry in MCP configuration alone does not establish that the agent can call its tools.

### 3. Start with a read-only request

```text
Use $skilled-task-executor to list tasks in the client workspace
through my connected task platform.
```

Name the platform or project when needed. The agent should invoke a matching list/search tool and return its results. If access is unavailable, it should explain the connection, installation, or permission step needed to continue.

## Try a prompt

Explicit invocation makes the intended skill clear. Automatic invocation is also enabled for requests with task-management context; ordinary coding requests without that context are outside its intended trigger.

### Find work

```text
In the client workspace, list tasks in Asana.
```

### Review before implementing

```text
Use $skilled-task-executor in review-only mode to assess TASK-123
from my connected tracker. Read the relevant code and report gaps
against its acceptance criteria.
```

### Implement with checkpoints

```text
Use $skilled-task-executor to implement TASK-123 in interactive mode.
Confirm the implementation plan and pause at the git and task-update checkpoints.
```

### Carry work through to review

```text
Use $skilled-task-executor to implement TASK-123 end-to-end in autonomous
mode, run the relevant validation, and prepare a PR handoff.
```

Replace sample identifiers with real task IDs or URLs from your connected tracker. A request to list tasks stays a read-only query; it does not begin a coding workflow.

## Choose your mode

| Mode | Best for | Behavior |
| --- | --- | --- |
| `interactive` | Collaborative implementation | Default. Pauses at meaningful state changes such as branches, commits, pushes, PRs, and task updates. |
| `autonomous` | Explicitly authorized end-to-end work | Proceeds through clear, authorized phases; stops for ambiguity, missing permissions, destructive actions, and applicable external-action checkpoints. |
| `review-only` | Inspection, analysis, and estimates | Reads task and code context without modifying code, git, PRs, or task state. |

See [safety and checkpoints](references/safety-and-checkpoints.md) for the complete action boundaries. Autonomous mode does not grant unrestricted permission to merge or mark tasks complete.

## How the workflow fits together

```text
Task request
    |
    v
Discover callable platform tools
    |
    +-- Unavailable --> Connection / setup / permission handoff
    |
    +-- List or search --> Tool-backed results
    |
    +-- Implementation --> Understand --> Plan --> Implement --> Validate
                                                                  |
                                                                  v
                                             Review --> Git / PR / task sync
                                                        (with checkpoints)
```

The skill maps conceptual capabilities to tools exposed by your environment. Names such as `task.read` are protocol labels, not MCP method names to invoke literally.

| Capability group | What is needed |
| --- | --- |
| Task discovery | `task.list` or scoped `task.search` for collections; `task.read` or a complete readable search record for a known item. |
| Task synchronization | Comment, update, and transition tools for the requested changes. Missing write support is reported. |
| Git and review | Available local git commands and review-provider tools, subject to checkpoints. |
| Validation | Relevant tests, lint, type checks, builds, CI results, or browser checks available in the project. |

Missing optional capabilities narrow the handoff: the agent can prepare PR text when PR creation is unavailable, or provide manual checks when browser validation is unavailable. Missing live task access requires a connection recovery step. Pasted task details can provide implementation context, but cannot supply a live task listing.

## MCP access and credentials

Platform operations must use the MCP or connector tools exposed by the runtime. The skill explicitly prohibits extracting access tokens from MCP configuration, environment variables, secret stores, or authentication caches to recreate those operations with `curl`, SDKs, or custom HTTP requests.

If tools are missing or fail authentication, the agent should direct you to the supported connection flow. Never paste connector secrets into a task prompt. See the [MCP tool boundary](references/capability-contract.md#mcp-tool-boundary) for the full rule.

These are agent instructions. Enforcement also depends on your host's tool permissions and the agent following the skill.

## Documentation

| Guide | Use it to |
| --- | --- |
| [Skill entry point](SKILL.md) | Understand triggers, modes, required discovery, and core rules. |
| [Capability contract](references/capability-contract.md) | Map available tools and handle missing connectors. |
| [Execution workflow](references/execution-workflow.md) | Follow the phases from task discovery to completion. |
| [Task state and sync](references/task-state-and-sync.md) | Track execution state and write structured task updates. |
| [Safety and checkpoints](references/safety-and-checkpoints.md) | Understand approvals, worktree protection, and credential boundaries. |
| [Asana example](examples/asana.md) | Walk through task listing, implementation, and MCP setup recovery. |
| [Linear example](examples/linear.md) | See a capability mapping for Linear. |
| [Jira example](examples/jira.md) | See a capability mapping for Jira. |
| [Generic MCP example](examples/generic-compatible-mcp.md) | Apply the workflow to another compatible connector. |
| [Validation scenarios](tests/scenarios.md) | Evaluate expected behavior and known failure cases. |

Platform examples illustrate the protocol; they are not bundled integrations or a certification of every MCP implementation.

## Troubleshooting

| Symptom | Next step |
| --- | --- |
| The agent does not select the skill | Invoke `$skilled-task-executor` explicitly and check that the host discovered the installed package. |
| The connector is configured, but tools are unavailable | Enable or reconnect it through the host, then refresh or restart the session as needed. |
| Authentication or access fails | Complete the connector's sign-in flow or correct workspace permissions. Use the Asana guide above when applicable. |
| The agent describes tools but never lists tasks | Verify that it loaded the latest skill and invoked an actual list/search tool. Discovery alone is insufficient. |
| The agent extracts a token or falls back to `curl` | Stop that operation and refresh the installed skill. This violates the MCP tool boundary. |
| Recent skill changes have no effect | Update the installed copy, including its reference files, and refresh the agent session. |
| PR creation or task transitions are unavailable | Expect prepared PR metadata or a precise report of the skipped task update. |

## Contributing and validation

Improvements are especially useful when grounded in a reproducible agent behavior. [Open an issue](https://github.com/myckhel/skilled-task-executor/issues) with the prompt, host, available capability names, expected behavior, and a redacted account of what happened. Omit secrets and private task content.

Keep the entry point compact and place detailed protocol guidance in the relevant reference. Preserve platform portability, add a behavioral scenario when fixing a workflow failure, and check that links resolve.

The checks in [tests/scenarios.md](tests/scenarios.md) are manual behavioral scenarios, not an automated integration suite. When the Codex skill-creator validator is installed, run its `scripts/quick_validate.py` against this skill folder; its Python environment needs `PyYAML`. Structural validation checks package metadata and scaffolding, while scenario evaluation checks agent decisions. Neither substitutes for testing with your actual host and connector.

---

<div align="center">

**Skilled Task Executor** · A clear task. A careful change. A traceable handoff.

[Repository](https://github.com/myckhel/skilled-task-executor) · [Issues](https://github.com/myckhel/skilled-task-executor/issues) · [Workflow](references/execution-workflow.md)

</div>
