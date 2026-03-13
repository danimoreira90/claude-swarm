---
name: mcp-builder
description: >
  Build, configure, and test MCP servers for any external service.
  Use for adding new capabilities to Claude via the Model Context Protocol.
  Activate via /mcp command.
---

# MCP Builder Skill

> MCP (Model Context Protocol) extends Claude with tools to interact with external services.

## What MCP Servers Can Do

- **Database MCP**: Query PostgreSQL, Redis, MongoDB directly from Claude
- **API MCP**: Call any REST API with authentication
- **File system MCP**: Extended file operations beyond built-in tools
- **Communication MCP**: Send Slack messages, emails, create Jira tickets
- **Monitoring MCP**: Query Grafana, PagerDuty, Datadog

## Adding an Existing MCP Server

Most useful MCPs are pre-built. Add them to `mcp-configs/mcp-servers.json`:

```bash
# Add via CLI (recommended)
claude mcp add memory npx -- -y @modelcontextprotocol/server-memory
claude mcp add github -e GITHUB_PERSONAL_ACCESS_TOKEN=$PAT npx -- -y @modelcontextprotocol/server-github
claude mcp add context7 npx -- -y @context7/mcp-server

# Verify
claude mcp list
claude mcp test memory
```

## Building a Custom MCP Server (TypeScript)

```typescript
// my-service-mcp/src/index.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { CallToolRequestSchema, ListToolsRequestSchema } from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "my-service", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

// Define available tools
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "create_ticket",
      description: "Create a support ticket in the system",
      inputSchema: {
        type: "object",
        properties: {
          title: { type: "string", description: "Ticket title" },
          description: { type: "string", description: "Ticket description" },
          priority: { type: "string", enum: ["low", "medium", "high", "critical"] },
        },
        required: ["title", "description", "priority"],
      },
    },
  ],
}));

// Handle tool calls
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "create_ticket") {
    const { title, description, priority } = request.params.arguments as {
      title: string;
      description: string;
      priority: string;
    };

    // Call your external service
    const ticket = await myServiceClient.createTicket({ title, description, priority });

    return {
      content: [
        {
          type: "text",
          text: `Ticket created: #${ticket.id} - ${ticket.url}`,
        },
      ],
    };
  }
  throw new Error(`Unknown tool: ${request.params.name}`);
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
```

```json
// package.json
{
  "name": "my-service-mcp",
  "bin": { "my-service-mcp": "./dist/index.js" },
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

## MCP Server Configuration

```json
// In mcp-configs/mcp-servers.json
{
  "mcpServers": {
    "my-service": {
      "command": "node",
      "args": ["/path/to/my-service-mcp/dist/index.js"],
      "env": {
        "MY_SERVICE_API_KEY": "YOUR_KEY_HERE"
      },
      "description": "Custom MCP for My Service — creates tickets, queries status"
    }
  }
}
```

## MCP Testing Checklist

```
[ ] Server starts without errors
[ ] All tools listed correctly
[ ] Tool schemas validate properly
[ ] Tool calls return correct format
[ ] Error handling returns graceful messages
[ ] API credentials from environment (not hardcoded)
[ ] Timeout handling (external calls can fail)
```

## Pre-built MCP Servers to Know

```bash
# Core infrastructure
@modelcontextprotocol/server-github
@modelcontextprotocol/server-postgres
@modelcontextprotocol/server-redis
@modelcontextprotocol/server-memory
@modelcontextprotocol/server-filesystem

# Search & Research
exa-mcp-server
@context7/mcp-server

# Cloud
@railway/mcp-server
@supabase/mcp-server-supabase

# AI/ML
@langfuse/mcp-server
@qdrant/mcp-server-qdrant
```
