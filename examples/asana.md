# Asana Example

This example illustrates how the skill can operate when a connected Asana-capable tool exposes compatible task capabilities. It is not an Asana integration and does not require a specific MCP server.

## User Request

```text
Use $skilled-task-executor to implement the Asana task about expired session handling.
```

Collection request:

```text
In the client workspace, list tasks in Asana.
```

## Capability Mapping

Possible mapping:

- `task.list`: list tasks in an Asana workspace, team, project, section, or assignee scope.
- `task.search`: search Asana tasks by title, assignee, project, or URL.
- `task.read`: read task name, description, custom fields, comments, attachments, and URL.
- `task.comment`: add implementation updates.
- `task.transition`: move the task to the closest available workflow section or status when the tool supports it.

## Expected Behavior

The agent first inspects callable integrations for an Asana-capable connector. If available and connected, it invokes the relevant list, search, or read operation; for the collection request, it resolves the named workspace to the appropriate accessible scope and returns only tool-backed results.

Even if an Asana access token appears in MCP configuration, the agent does not read or copy it and does not use `curl` or direct HTTP. The agent uses the exposed Asana MCP tools. If those tools are not callable, it treats the connector as unavailable or incompletely loaded and prompts for reconnection, enablement, installation, or restart through the host.

If the Asana connector is installed but disconnected, the agent prompts the user to connect or sign in. If it is not installed, the agent uses the host's integration discovery or installation-suggestion flow when available and prompts the user to install or enable the Asana integration. It does not claim that Asana was checked and does not replace the result with general instructions.

For implementation, the agent then summarizes requirements, inspects the repo, identifies validation commands, and pauses at the branch checkpoint in interactive mode.

If implementation completes and `task.comment` exists, the agent posts a structured update. If Asana transition support is absent, the agent says it could not move the task status and does not pretend otherwise.

## MCP Setup And Recovery

When the Asana MCP connector is missing, disconnected, incompletely loaded, or failing authentication, prefer the host's supported installation and connection flow. If the host cannot complete that flow directly, direct the user to Asana's official [Integrating with Asana's MCP Server](https://developers.asana.com/docs/integrating-with-asanas-mcp-server) guide.

Use the guide for current V2 MCP setup, OAuth configuration, workspace access, client connection, tool discovery, and troubleshooting. After setup, rediscover the tools exposed by the runtime and invoke those tools for Asana operations.

Do not ask the user to paste client secrets or access tokens into chat. Do not read, expose, or repurpose credentials from MCP configuration, and do not turn the guide's HTTP-level details into a `curl` fallback.
