---
name: mcp
description: Build or configure an MCP server for a new capability or external service.
allowed-tools: ["Read", "Write", "Bash", "Glob"]
argument-hint: "<service to integrate or 'add <server-name>'>"
---

<objective>
Build or configure MCP server: $ARGUMENTS

Deliver: working MCP server (or configuration for existing one), tested, documented.
</objective>

<execution_context>
@skills/mcp-builder.md
@mcp-configs/mcp-servers.json
</execution_context>

<context>
**MCP Task**: $ARGUMENTS
</context>

<process>
1. Determine if a pre-built MCP server exists for this use case
2. If yes: configure it in mcp-configs/mcp-servers.json and test
3. If no: build a custom MCP server using TypeScript SDK
4. Add to AGENTS.md if it's a reusable swarm component
5. Test with: claude mcp test <name>
</process>
