# Project Manager Examples

These examples use conceptual capabilities, not built-in integrations. Replace sample destinations with resolved platform projects.

## Single Task

```text
Use $skilled-task-executor as PM to create a task in the client portal
project for showing a clear error when a session expires.
```

Resolve the project, inspect relevant context, write observable acceptance criteria, and search for duplicates if possible. With scope and required fields resolved, invoke `task.create` without a second publication approval. Return the confirmed reference; do not implement it.

## Feature Breakdown

```text
Create a parent task and subtasks in the client portal project for
password reset. Include requesting a reset, using an expiring link,
and coverage for invalid or expired links. Inspect the repository first.
```

Read relevant code without edits, avoid duplicating already implemented work, and create a parent before its children. Each child has its own scope and acceptance criteria. If child creation is unsupported, propose a flat-task alternative and wait for agreement before publishing the batch.

## Draft Only

```text
Act as PM and break the following feature brief into tickets.
Draft only; do not publish anything: add account notification preferences.
```

Return a parent and actionable child drafts with assumptions labeled. No connector is required for this supplied brief. Do not invent existing project settings or require platform setup to draft. Review-only mode likewise prevents publication, even if creation wording also appears.

## Create And Implement

```text
Create a task in the client portal project for the expired-session bug,
then implement it in this repository with the normal checkpoints.
```

Create the resolved task, retain its confirmed reference, and pass its requirements to the executor workflow. If creation fails or is uncertain, return drafts and the recovery step before implementation. Creation authorization does not waive subsequent branch, commit, push, or PR checkpoints.
