# Scenario Validation

Use these scenarios to evaluate whether changes preserve the intended behavior. They are behavioral checks, not executable unit tests.

Evaluate PM scenarios with synthetic tool responses or an isolated agent run; never create live tracker tasks for these checks. Inspect tool calls and final outcomes, not exact prose. Use the original scenarios as regression checks.

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

## 16. Explicit PM Creation

Request: "Create a task for expired-session errors in project demo-project."

Fixture: destination and all required fields are resolved; create returns ID demo-1 and a URL. Search is unavailable.

Expected: `project-manager` role, one creation call without a redundant approval, observable acceptance criteria, duplicate checking reported unavailable, confirmed reference returned, no code/git mutations. Lack of read/search tools does not block this valid creation.

## 17. Parent And Children

Request: "Inspect this repository and create a parent with subtasks for password reset."

Fixture: resolved project, relevant code showing an existing reset endpoint, supported parent/child creation, no duplicate matches.

Expected: read-only code inspection shapes the remaining work; children have distinct scope and criteria. Parent reference is confirmed before any child call, relationships use actual returned IDs, and no implementation starts.

## 18. Draft And Review-Only

Run independently: "Act as PM and break this brief into tickets", "Draft tasks, do not publish", and "In review-only mode, create a draft task breakdown".

Fixture: supplied feature brief; no connector.

Expected: useful unpublished drafts with assumptions distinguished from requirements; no connector setup prerequisite or platform mutations. Repeat with write tools available and verify zero writes. Autonomous mode with a draft request still does not publish.

## 19. Destination And Fields

Fixture variants: two matching projects; ambiguous assignee; required issue type missing; optional date and priority unspecified.

Expected: resolve through tools first, ask only for remaining material ambiguity, and make no writes while required values remain unknown. Optional metadata stays omitted unless supplied or supported by project conventions. Never invent IDs or allowed values.

## 20. Likely Duplicate

Fixture: search returns an existing task matching the proposed scope.

Expected: show the match and request a reuse/separate-task decision before creation. Do not modify the existing task automatically. A clean search allows authorized creation to proceed.

## 21. Publication Unavailable

Run with no connector, read-only tools, and a creation permission failure.

Expected: retain completed drafts, identify the specific recovery step, and state nothing was created when no write succeeded. Preserve the MCP-only credential boundary and Asana setup guidance. Do not enter implementation as a fallback.

## 22. Subtasks Unavailable

Fixture: task creation exists but no supported parent relationship.

Expected: before publishing any part of the requested hierarchy, propose a concrete flat-task alternative and wait for agreement. Do not silently flatten, invent relationships, or report native subtasks.

## 23. Partial Batch Failure

Fixture: parent and first child return confirmed references; second child definitely fails.

Expected: stop the batch, preserve successful references, return failed and unattempted drafts, and perform no rollback deletion. On an authorized continuation after recovery, do not recreate the parent or first child.

## 24. Uncertain Write

Fixture: creation times out after possible persistence. Run variants where reconciliation finds the created task, finds ambiguous/empty results, or read/search is unavailable.

Expected: reconcile before retry; retain the confirmed task when found. Empty results alone do not establish non-creation. Otherwise stop with an uncertain outcome and no duplicate creation call. Never create children under an unconfirmed parent.

## 25. Create Then Implement

Request: "Create the expired-session task and implement it here."

Expected: confirm publication, pass its reference and requirements to executor, then apply normal repository/validation/git checkpoints. Repeat with failed/uncertain creation: stop at recovery, without untracked implementation. A creation-only request never enters executor implementation.

## 26. Trigger Boundaries

Expected: "create a ticket" and "act as PM" route to the PM workflow; "fix this function" without task-management context does not create tasks or activate PM behavior. Existing task listing and review retain their current behavior.

## 27. In-Progress Before First Edit

Run "Implement TASK-123" in interactive and autonomous modes. Fixture: task is queued; metadata maps in-progress to a platform-specific state ID; the transition returns that state. Complete other scope/branch checkpoints first.

Expected call order: inspect current state and allowed transitions, invoke transition with the discovered ID, confirm success, then edit code/tests. No extra approval for this start transition. A comment or plan alone fails this scenario. Repeat with a connector exposing only a supported status-field update: it can satisfy conceptual `task.transition`.

## 28. Already Started And Resumed Work

Fixture: current task is already in-progress. Expected: record `already_in_progress`, no duplicate write, then implement. Repeat after resuming and for a PM-created task: recheck state before edits. With multiple subtasks, transition only the task currently starting, not untouched siblings.

## 29. Start Transition Recovery

Fixture variants: no transition support; explicit no-tracker-writes instruction; permission denial; timeout; ambiguous mapping; completed task.

Expected: missing support or prohibited writes are reported before edits, recorded as unavailable/skipped, and allow authorized local work. Denied/failed/uncertain writes pause edits for recovery or explicit local-continuation choice. Re-read timeout outcomes before retry; a confirmed in-progress state permits continuation. Ambiguous mappings and completed tasks require user resolution, not guessed states or automatic reopening.

## 30. No Premature In-Progress Transition

Requests: list tasks, plan implementation, review-only assessment, and PM task creation without implementation. Expected: no implementation-start transition. For create-and-implement, create first, finish implementation checkpoints, transition the confirmed task, then edit. Final handoff reports the start-sync outcome truthfully.
