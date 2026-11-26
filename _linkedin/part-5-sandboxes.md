"It runs in a container" is not a sandbox.

I tested this while writing the last part of my agent builder series. A container with default networking and my home directory mounted read-only could still read my SSH keys and reach the open internet. Same result as no sandbox at all.

So I added negative checks to factory-doctor: checks that pass only when access is denied.

- Workspace readable: should pass
- ~/.ssh readable: should fail
- example.com reachable: should fail

Same seven-line config, three runs:
- bare VM: 2 failures
- convenient container: 2 failures
- --network none, read-only root, nothing from home: 3 passes, exit 0

That's evidence, not a claim. And you can run it as the first step of every agent session.

Part 5 of "Field Notes from an Agent Builder" covers the lethal trifecta, the four boundaries I check in any agent runtime, process sandboxes vs. containers vs. gVisor vs. microVMs, and why I think the sandbox is now the real permission model for agents.

https://www.dewanahmed.com/sandboxes-permission-model/

How do you verify your agent sandbox actually blocks what it claims to?

#AIAgents #AISecurity #Sandboxing
