A2A is a good protocol. It's also the most over-applied idea in agent architecture right now.

If your planner, coder, and reviewer agents run in the same process, under the same credentials, and ship together, they're functions. Putting an Agent Card and an HTTP server in front of each one adds latency, failure modes, and an auth story, and buys you nothing.

Where A2A earns its keep is at a boundary:
- another team owns the agent
- it runs on a different framework or vendor
- the work takes hours and might need a human answer midway
- the other side holds data or credentials you shouldn't

Part 3 of "Field Notes from an Agent Builder" walks through A2A 1.0 end to end (Agent Card, SendMessage, the input-required state that makes it worth it), with JSON I validated against the 1.0 SDK types. If you built against 0.3, the migration is real.

https://www.dewanahmed.com/a2a-boundaries/

Are you using A2A across a real org or vendor boundary yet, or still inside one codebase?

#A2A #AIAgents #MultiAgentSystems
