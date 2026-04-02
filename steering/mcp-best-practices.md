---
title: MCP (Model Context Protocol) Best Practices
inclusion: always
---

# MCP (Model Context Protocol) Best Practices

## Server Configuration
- Use workspace-level config (`.kiro/settings/mcp.json`) for project-specific servers
- Use user-level config (`~/.kiro/settings/mcp.json`) for global/cross-workspace servers
- Workspace config takes precedence over user config for server name conflicts
- Always specify exact versions or use `@latest` for stability

## Installation and Setup
- Use `uvx` command for Python-based MCP servers (requires `uv` package manager)
- Install `uv` via pip, homebrew, or follow: https://docs.astral.sh/uv/getting-started/installation/
- No separate installation needed for uvx servers - they download automatically
- Test servers immediately after configuration, don't wait for issues

## Security and Auto-Approval
- Use `autoApprove` sparingly and only for trusted, low-risk tools
- Review tool capabilities before adding to auto-approve list
- Regularly audit auto-approved tools for security implications
- Consider environment-specific auto-approve settings

## Error Handling and Debugging
- Set `FASTMCP_LOG_LEVEL: "ERROR"` to reduce noise in logs
- Use `disabled: false` to temporarily disable problematic servers
- Servers reconnect automatically on config changes
- Use MCP Server view in Kiro feature panel for manual reconnection

## Common MCP Server Examples
```json
{
  "mcpServers": {
    "aws-docs": {
      "command": "uvx",
      "args": ["awslabs.aws-documentation-mcp-server@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": []
    },
    "filesystem": {
      "command": "uvx",
      "args": ["mcp-server-filesystem@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": ["read_file", "list_directory"]
    }
  }
}
```

## Testing MCP Tools
- Test MCP tools immediately after configuration
- Don't inspect configurations unless facing specific issues
- Use sample calls to verify tool behavior
- Test with various parameter combinations
- Document working examples for team reference

## Performance Optimization
- Disable unused servers to improve startup time
- Use specific tool names in auto-approve rather than wildcards
- Monitor server resource usage and adjust as needed
- Consider server-specific environment variables for optimization

## Development Workflow
- Add MCP servers incrementally, test each addition
- Use version pinning for production environments
- Document server purposes and usage in team documentation
- Create project-specific server collections for different use cases

## Troubleshooting
- Check server logs in Kiro's MCP Server view
- Verify `uv` and `uvx` installation if Python servers fail
- Test server connectivity outside of Kiro if needed
- Use command palette "MCP" commands for server management
- Restart servers via MCP Server view rather than restarting Kiro

## Best Practices for Tool Usage
- Understand tool capabilities before first use
- Use descriptive prompts when calling MCP tools
- Handle tool errors gracefully in workflows
- Combine multiple MCP tools for complex tasks
- Cache results when appropriate to avoid repeated calls

## Development Integration
- Use Context7 MCP server to verify dependency compatibility before adding libraries
- Leverage AWS-Knowledge MCP server for current AWS documentation and best practices
- Use aws-api-mcp-server for AWS API interactions and validation
- Reference official sources through MCP servers when available in documentation

## Agentic Tool Design for MCP Servers (Research-Validated)

These patterns are grounded in production agentic AI research (2025-2026).

### Tool Design Principles
- **Single-purpose tools:** Each tool should do one thing well. A tool called `manage_database` that handles reads, writes, schema changes, and backups is harder for the model to use correctly than four separate tools.
- **Descriptive names and documentation:** The tool name and description are the primary signals the model uses to decide when and how to call a tool. Invest in clear, unambiguous descriptions with usage examples.
- **Validate all inputs:** Every tool call from an LLM should be validated before execution. Simple rule: reject, fix, or escalate; no silent failures.
- **Minimize tool count per context:** Dynamically limit the set of available tools to those relevant to the current task. Aim for 5-15 tools visible at any time; beyond 20, selection accuracy degrades.
- **Parallel tool calling:** Design tools to be independently callable when possible. Claude 4.6 can execute multiple tool calls simultaneously when there are no dependencies.

### Tool Call Optimization
- Function calling adds 15-30% token overhead but unlocks 10x more agent capabilities
- Reduce unnecessary tool calls by providing sufficient context upfront
- Cache tool results when the underlying data doesn't change frequently
- Use batch endpoints for non-real-time tool operations (50% cost reduction typical)
- Implement retry logic with exponential backoff for transient tool failures
- Set timeouts on all tool calls; a hanging tool call blocks the entire agent loop

### MCP Server Architecture (Agentic Patterns)
- Streamable HTTP replaced SSE as the recommended remote transport (spec 2025-03-26)
- Build one MCP server per domain/system; don't create monolithic servers
- Each server should expose a focused set of tools (5-15 is optimal)
- Enforce authorization at the tool level, not just the server level
- Design tools to be idempotent where possible
- Return structured error responses with actionable error messages
- Include execution metadata (duration, tokens used, cache hit/miss) in responses
- Use connection pooling for database-backed tools
- Return only the data the agent needs; avoid returning entire database rows when a single field suffices
- Implement pagination for list operations; never return unbounded result sets