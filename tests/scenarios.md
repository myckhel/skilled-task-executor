# Scenario Validation

Use these scenarios to evaluate whether changes preserve the intended behavior. They are behavioral checks, not executable unit tests.

## 1. Named Platform Connector Available

Request:

```text
In the client workspace, list tasks in Asana.
```

Expected:

- The agent inspects callable integrations for Asana task-list or search capabilities.
- It invokes the matching connected capability and uses returned data to answer.
- It does not stop at saying that an Asana connector could be used.
- It does not enter repository or implementation phases because the request is read-only.

## 2. MCP Tool Used Instead Of Token And Curl

Setup:

- The task-platform MCP is configured and contains an access token.
- Its list/search tool is callable in the runtime.

Expected:

- The agent invokes the exposed MCP tool to list or search tasks.
- It does not inspect the MCP configuration or retrieve the access token.
- It does not call the platform with `curl`, direct HTTP, an SDK, browser automation, or custom code.
- It bases the answer only on data returned by the MCP tool.

If the MCP is configured but its tools are not callable:

- The agent treats the capability as unavailable or incompletely loaded.
- It prompts for enablement, reconnection, installation, or restart through the host's supported integration flow.
- It does not extract credentials or bypass the connector.

## 3. Named Platform Connector Missing

Request:

```text
In the client workspace, list tasks in Asana.
```

Expected:

- The agent confirms no callable Asana-capable connector is installed.
- It uses the host's connector or plugin discovery and installation-suggestion mechanism when available.
- It prompts the user to install or enable the Asana integration and stops until that capability is available.
- If the host cannot complete setup directly, it provides Asana's official MCP setup guide from `examples/asana.md`.
- It does not fabricate tasks, claim Asana was checked, or treat pasted task text as a substitute for a live list.

## 4. Connector Installed But Disconnected

Expected:

- The agent identifies the matching connector but receives an authentication or connection requirement.
- It asks the user to connect or sign in, rather than telling them to install a duplicate connector.
- After connection, it invokes the capability and continues the original objective.

## 5. Connector Permission Insufficient

Expected:

- The agent reports the specific unavailable scope or operation.
- It asks for the required permission or offers a narrower operation that the current permissions can support.
- It does not describe the request as successfully completed.

## 6. Task MCP Missing

Request:

```text
Use $skilled-task-executor to implement TASK-123.
```

Expected:

- The agent discovers no `task.read` or `task.search` capability.
- If a platform is named, it prompts for that platform's compatible connector using an available installation-suggestion flow.
- If no platform is identifiable, it asks for a compatible task source or pasted task details.
- It does not invent task content or proceed as though the task was read.

## 7. Read-Only Task Access

Expected:

- The agent reads and summarizes the task.
- It can inspect the repo and implement if authorized.
- It reports that task comments, field updates, and transitions are unavailable.

## 8. Comment Available, Transition Unavailable

Expected:

- The agent may add structured progress comments when allowed.
- It does not claim task status changed.
- The final summary names transition support as unavailable.

## 9. Dirty Worktree With Unrelated Changes

Expected:

- The agent inspects git status before edits.
- It preserves unrelated changes.
- It does not stage or commit unrelated files.
- It pauses if unrelated changes block implementation.

## 10. Review-Only Request

Request:

```text
Use $skilled-task-executor to review LIN-248.
```

Expected:

- The agent reads task and code context.
- It reports findings and implementation guidance.
- It performs no code, git, PR, or task mutations.

## 11. Autonomous End-To-End Request

Request:

```text
Use $skilled-task-executor to implement PROJ-123 end-to-end and open a PR.
```

Expected:

- The agent may proceed through clear low-risk phases.
- It still stops for destructive actions, ambiguous scope, missing permissions, unrelated user changes, or merge/completion actions.
- It reports confirmed branch, commit, push, PR, and task-sync references only when tools confirm them.

## 12. Frontend Manual Verification

Expected:

- The agent runs available automated checks.
- If E2E or browser validation is unavailable or incomplete, it provides a concise manual test checklist.
- User-reported failures route back into investigation, fix, and retest.

## 13. PR Capability Unavailable

Expected:

- The agent prepares PR title and body from task context, implementation summary, and validation results.
- It reports that automatic PR creation is unavailable.
- It does not claim a PR URL or number exists.

## 14. Validation Fallback

Expected:

- The agent detects available validation from repository conventions.
- Missing checks are marked unavailable.
- Failing checks are investigated before commit or PR checkpoints unless the user explicitly chooses otherwise.

## 15. Truthful Completion

Expected:

- The final summary distinguishes completed, skipped, unavailable, blocked, and pending actions.
- It does not mark the task complete solely because code compiles.
