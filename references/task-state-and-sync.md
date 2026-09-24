# Task State And Sync

Use semantic state and structured comments so this skill remains portable across Asana, Linear, Jira, GitHub Issues, and other task platforms.

## Semantic States

Map platform-specific statuses to these conceptual states only after inspecting available workflow values:

- `discovered`: task found and read.
- `acknowledged`: task understood, but implementation has not started.
- `implementation_started`: work has begun.
- `implementation_complete`: code changes are done, validation may still be pending.
- `validation_complete`: validation has passed or has a documented final status.
- `ready_for_review`: PR or review handoff is ready.
- `ready_for_merge`: review is complete and merge is the next step.
- `completed`: task is complete according to the user and platform workflow.
- `blocked`: progress cannot continue without missing information, failing dependency, unavailable permission, or external action.

Never assume a platform has statuses with these names. When `task.transition` exists, inspect available platform transitions and choose the closest match only when the selected mode allows it.

`implementation_started` must be synchronized at the [implementation-start gate](execution-workflow.md#implementation-start-gate), before implementation edits. A local semantic state or progress comment never counts as a confirmed platform transition.

## Execution State Convention

Maintain this shape internally or in notes when a task run is complex:

```yaml
execution:
  role: executor
  mode: interactive
  task:
    platform: unknown
    id: ""
    title: ""
    url: ""
    status: ""
    assignee: ""
  capabilities:
    requested_platform: unknown
    connector_state: unknown
    discovery_attempted: false
    available: []
    unavailable: []
    degraded: []
  repository:
    path: ""
    current_branch: ""
    base_branch: ""
    dirty_worktree: false
    unrelated_changes: []
  implementation:
    requirements: []
    plan: []
    files_changed: []
  validation:
    lint: pending
    typecheck: pending
    unit: pending
    integration: pending
    e2e: pending
    build: pending
    manual: pending
  git:
    branch_created: false
    commit_created: false
    commit: ""
    pushed: false
  pull_request:
    created: false
    id: ""
    url: ""
  task_sync:
    implementation_start:
      outcome: pending
      target_state_id: ""
      target_state_name: ""
      confirmed_state: ""
      reason: ""
    comment_added: false
    status_updated: false
    final_state: ""
```

Use empty strings or omitted fields when facts are unavailable. Do not invent data to complete the shape.

Implementation-start outcomes are `pending`, `transitioned`, `already_in_progress`, `unavailable`, `skipped_by_user`, `failed`, or `uncertain`. Set `confirmed_state` only from tool evidence and retain reasons for any degraded outcome. `failed` and `uncertain` block edits until resolved or the user explicitly chooses local continuation; record that choice. Include this outcome in the final handoff.

For PM work, set `role: project-manager` and add this creation state under `execution`. Omit irrelevant implementation/git fields. Local draft keys identify pending items; they are not platform task IDs.

```yaml
creation:
  intent: draft
  destination:
    platform: ""
    workspace_id: ""
    project_id: ""
    label: ""
  duplicate_check: unavailable
  items:
    - draft_key: task-1
      parent_draft_key: ""
      title: ""
      description: ""
      requested_fields: {}
      publication: draft
      task_id: ""
      task_url: ""
      parent_task_id: ""
      unapplied_fields: []
      error: ""
```

`intent` is `draft` or `publish`; duplicate checks are `not_run`, `checked`, `matches_found`, or `unavailable`. Per-item publication is `draft`, `pending`, `created`, `failed`, or `uncertain`. Record `created` only with a tool-confirmed reference; record a missing/ambiguous write confirmation as `uncertain`. Preserve successful references across retries and report the unresolved items individually. These are internal outcomes, not tracker statuses. Publishing a task never means its implementation is completed.

## Structured Task Comments

When `task.comment` is available and the selected mode allows task comments, prefer concise structured updates.

These comment statuses describe work progress, not proof of tracker state. If platform synchronization is unavailable or skipped, explicitly note that distinction in any authorized comment. The start transition does not depend on comment capability.

Implementation start:

```markdown
## Implementation Update

Status: In Progress

### Scope
- <task requirement being addressed>

### Plan
- <short implementation step>
- <short validation step>

### Branch
<branch name or "Not created yet">
```

Progress update:

```markdown
## Implementation Update

Status: In Progress

### Completed
- <completed work>

### Validation
- <check>: <PASS|FAIL|PENDING|UNAVAILABLE|SKIPPED>

### Next
- <next action>
```

PR ready:

```markdown
## PR Ready

Status: Ready for Review

PR: <PR reference>

### Changes
- <user-facing change>

### Validation
- <check>: <PASS|FAIL|UNAVAILABLE|SKIPPED>
```

Blocked:

```markdown
## Blocked

Status: Blocked

### Blocker
- <missing information, failing dependency, permission issue, or unavailable capability>

### Needed
- <specific user or external action needed>
```

## Degraded Sync

If a platform lacks a capability, adapt and say exactly what changed:

- No `task.comment`: keep updates in the chat only.
- No `task.update`: do not edit task fields.
- No `task.transition`: comment if possible, but do not claim status changed.
- No task write capabilities: for authorized implementation, complete local work and report unavailable sync; for PM work, return unpublished drafts and setup recovery, without starting implementation.

Task synchronization is successful only when the connected tool returns confirmation. If confirmation is absent, report the update as attempted or unavailable, not complete.
