Every agent framework I've tried is, underneath, the same 30 lines of code.

Call the model. If it asks for a tool, run the tool. Append the result. Repeat until it stops or you hit a turn limit.

Once I wrote that loop by hand, framework demos stopped impressing me. Now I evaluate frameworks on five questions instead:

1. Can it resume after a crash without repeating side effects?
2. Can it pause for a human and pick up later?
3. Can I see every model call, tool call, and token?
4. Can I swap the model provider?
5. Can I test the loop with a fake model?

If the answer to #5 is no, I'm not shipping it.

I'm starting a 5-part series, "Field Notes from an Agent Builder," covering agent loops, MCP, A2A, coding agents, and sandboxes. Part 1 has the loop, a fake-model test, and why I push deterministic work into small CLIs like factory-doctor instead of prompts.

https://www.dewanahmed.com/agents-are-loops/

What's the one thing you check before adopting an agent framework?

#AIAgents #AIEngineering #SoftwareEngineering
