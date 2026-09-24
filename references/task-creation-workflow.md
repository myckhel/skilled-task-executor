# Task Creation Workflow

Use the `project-manager` role for task drafting, creation, and requested feature breakdowns. Keep the existing execution modes; the role does not grant publication permission. This workflow covers single tasks and parent/subtask breakdowns, not milestone or sprint scheduling.

## Intent And Context

1. Identify the requested outcome, role, mode, and whether the user wants drafts or publication. "Act as PM" or "break this feature into tickets" alone means draft; "create these tickets in the project" authorizes publication. Draft-only and review-only instructions take precedence over inferred creation intent.
2. Understand the brief, intended users, constraints, and success criteria. Read relevant code and documentation when available to ground technical tasks. Do not modify code, git, or repository files during PM work unless separately requested.
3. For live context or publication, follow the [capability contract](capability-contract.md). Resolve the platform and destination through available tools or unambiguous supplied identifiers. Do not force connector setup merely to draft from supplied context.
4. Discover required creation fields, allowed values, and any established project conventions. Resolve requested assignees to real identities. Set dates, priorities, labels, and estimates only when supplied or justified by those conventions; omit optional unknowns.
5. Ask about scope conflicts, ambiguous destinations or identities, and missing required fields only when tools or context cannot resolve them. Do not guess platform IDs or factual requirements.

A later explicit instruction such as "leave review-only mode and publish these drafts" changes the mode and publication intent. If the active request still specifies review-only while also asking to create or implement, preserve review-only and explain the conflict; do not infer permission from role selection alone.

## Draft Quality

Use one task for a cohesive request. When a breakdown is requested, draft a parent describing the overall outcome and children that are independently actionable. Give each child its own scope and observable acceptance criteria, and identify dependencies without implying they were already linked in the tracker.

Adapt this template to the platform's supported description format. Omit optional sections when unnecessary:

```markdown
# <Action-oriented title>

## Objective And Context
<Problem, intended outcome, and confirmed requirements>

## Scope
- <Work included>

## Acceptance Criteria
- <Observable behavior or verifiable outcome>

## References
- <Relevant brief, task link, or repository file>

## Exclusions
- <Explicitly excluded work, when useful>

## Dependencies And Validation
- <Prerequisite or useful validation guidance, when relevant>

## Assumptions And Open Questions
- <Clearly labeled assumption or unresolved question, if any>
```

Do not present inferred implementation choices as confirmed requirements. Resolve questions that materially change scope before publication; a task explicitly requested to investigate an unknown can describe that unknown as its objective. Task creation does not imply tests were run or the feature was implemented.

## Duplicate Check And Publication

- Before publishing, search the resolved destination for overlapping work if suitable search is available. Surface strong matches and ask whether to reuse the existing item or create separate work. Do not update existing tasks automatically. If search is unavailable, disclose that duplicate checking was unavailable and proceed with an otherwise resolved, authorized creation.
- Explicit creation authorizes the requested single task or breakdown, including in interactive mode. Draft/plan requests and review-only mode publish nothing. Autonomous mode does not authorize an unsolicited backlog.
- Preflight the requested hierarchy and required fields before the first write. If native child creation is unavailable, show the proposed flat-task alternative and wait for the user's choice before changing the structure or publishing the batch.
- Invoke only exposed MCP/connector tools. A conceptual capability may be implemented by a general creation tool with a parent field; no specific method name is required.
- Create a parent first and record its confirmed reference before creating children. Record every returned task ID/URL immediately. Apply requested fields through supported creation inputs or authorized follow-up field operations; report any unapplied fields or relationships.
- If creation is unavailable, return completed drafts and the matching connection, installation, or permission recovery step. State explicitly that the tasks were not published. Never obtain tokens or use direct HTTP as a fallback.

## Failure And Retry

On a definite creation failure, stop the batch, retain successful task references, and return remaining drafts with the error and next step. Do not delete successful tasks as rollback or recreate them on continuation.

A timeout or response without a confirmable task reference is an uncertain write, not proof of failure. Reconcile through available read/search tools using destination, content, and returned references where available. Resume only when the result is established: retain an identified creation, or retry after confirmed non-creation. Empty or ambiguous search results alone may not establish non-creation. If uncertainty remains or reconciliation tools are unavailable, report it and stop rather than blindly retrying. Children must never use an unconfirmed parent reference.

Confirmed non-creation requires an authoritative connector/platform outcome for that attempt, such as a definitive rejection before persistence or a documented request-status result stating no task was created. If the connector offers no such evidence, keep the outcome uncertain. A matching title alone is insufficient to adopt an existing task as the result of the attempt; verify its scope and identity with available evidence.

## Completion And Executor Handoff

Return the destination, confirmed task IDs/URLs and hierarchy, unpublished drafts, uncertain outcomes, and any missing fields or capabilities. Distinguish fully published, partially published, and draft-only results using the [execution-state convention](task-state-and-sync.md).

For "create and implement" requests, pass confirmed task references and their requirements to the [execution workflow](execution-workflow.md). Keep its repository assessment, validation, and git/PR checkpoints. Stop at creation recovery if the requested publication is incomplete or uncertain; do not silently turn that request into untracked implementation.
