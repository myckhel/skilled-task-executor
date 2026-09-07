# Generic Compatible MCP Example

This example applies to any task-management MCP or IDE connector that can expose conceptual capabilities compatible with the skill.

## User Request

```text
Use $skilled-task-executor to work on the task assigned to me about checkout tax calculation.
```

## Minimum Capability

At least one of these must be available:

- `task.list`: enumerate tasks in the requested collection.
- `task.search`: find the user's assigned task.
- `task.read`: read a directly referenced task.

The agent must inspect the runtime's callable connectors and invoke the capability required by the objective. If no matching connector is installed, it uses any available host installation-suggestion flow and prompts the user to install the named platform integration. If installed but unauthenticated, it prompts the user to connect it. Pasted task content is only a fallback for reading or implementing a known task, not for live list or search requests.

MCP configuration and credentials are never used as a direct API fallback. If the connector is configured but its tools are not exposed, the agent asks the user to enable, reconnect, reinstall, or restart the connector instead of extracting a token and issuing HTTP requests itself.

## Expected Behavior

The agent adapts to available capabilities:

- with read-only task access, it can implement from task context but cannot sync progress;
- with comments but no transitions, it can post updates but not move workflow state;
- with no PR creation, it can prepare PR title/body for the user;
- with no E2E support, it can run available validation and provide manual checks.
