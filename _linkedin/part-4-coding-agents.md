When a coding agent session goes sideways, the model is rarely the first thing I'd fix.

More often, the session:
- started on a machine without the test runner
- explored a repo with no instructions
- downloaded a login page and called it build.zip
- kept spending because nothing told it to stop

That's a software factory problem, and it's the same discipline we've always applied to CI runners, just with a worker that will try to work around a broken environment for an hour instead of failing fast.

Part 4 of "Field Notes from an Agent Builder" is about the factory around the agent, with real output from three small, vendor-neutral CLIs I built:

- factory-doctor: preflight checks that fail in under a second, before any tokens are spent
- artifact-bridge: artifact downloads where the model never sees the token
- runway: a budget forecast that exits non-zero before you run out, not after

https://www.dewanahmed.com/coding-agents-software-factory/

What's the most common reason your coding agent sessions fail?

#CodingAgents #SoftwareFactory #DevOps
