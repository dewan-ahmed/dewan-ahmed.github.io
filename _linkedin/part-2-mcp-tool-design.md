MCP won the tool layer. I don't think that's the interesting part anymore.

The interesting part is that most MCP servers are designed like someone exported an OpenAPI spec and called it a day. 60 tools, one per endpoint, each returning a wall of JSON.

While building one around my flowfacts CLI, I ran into something small that changed how I write tools: an exception in a handler reached the model as "Error executing tool," with none of the detail. Switching to ToolError put the real message in front of the model, and it could fix its own call.

That's the theme of part 2 of "Field Notes from an Agent Builder":

- Shape tools around tasks, not endpoints
- Use the schema to reject bad input before your code runs
- Return less
- Make every error tell the model what to do next
- Keep credentials out of the model entirely

Plus what changed in the 2026 spec (stateless servers, server/discover, tasks as an extension) and the talks I'd watch if you're new to MCP.

https://www.dewanahmed.com/mcp-tool-design/

What's the worst MCP tool description you've seen?

#MCP #AIAgents #AIEngineering
