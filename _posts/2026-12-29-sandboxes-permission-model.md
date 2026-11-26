---
title: "Sandboxes Are the New Permission Model"
date: 2026-12-29T00:00:00-04:00
author: Dewan Ahmed
series: agent-builder
series_part: 5
header:
  teaser: "/assets/images/2026/agent-builder-part-5.png"
tags:
  - ai
  - agents
  - security
---

{% include agent-builder-series.html %}

Earlier this year I wrote about [sandboxing OpenClaw with nono]({% post_url 2026-02-03-sandbox-openclaw-nono %}). The point of that post was simple: an agent with shell access inherits everything you can do, and "the model will behave" is not a security boundary.

Since then I've stopped thinking of sandboxes as a hardening step you add at the end. For agents, the sandbox *is* the permission model. Prompts, tool descriptions, and approval dialogs all help, but the only thing that reliably limits what an agent can do is what its runtime can reach.

This last post in the series is about what that means in practice, and how I test that a sandbox actually does what I think it does.

## Why the old permission model breaks

Traditional permissions assume the principal is a person or a service with predictable intent. An agent is neither. It acts on your behalf, with your credentials, while reading content written by strangers: web pages, issues, pull request comments, tool results, and artifacts from other agents.

Simon Willison's [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) is the clearest framing I know. If an agent has:

1. access to private data,
2. exposure to untrusted content, and
3. a way to communicate externally,

then a prompt injection can turn it into an exfiltration tool. You can't patch that with a better system prompt, because the attack arrives as data the agent has to read to do its job.

Every post in this series has run into this. Tool results in [part 2]({% post_url 2026-12-08-mcp-tool-design %}) are untrusted input. Artifacts from a remote agent in [part 3]({% post_url 2026-12-15-a2a-boundaries %}) are untrusted input. Coding agents in [part 4]({% post_url 2026-12-22-coding-agents-software-factory %}) read issues and dependencies written by anyone. The fix in each case is the same: take away one leg of the trifecta, and enforce it below the agent.

## What a sandbox has to control

When I evaluate an agent runtime, I look at four boundaries:

| Boundary | The question | What good looks like |
| --- | --- | --- |
| Filesystem | What can the agent read and write? | The workspace only. No home directory, no SSH keys, no cloud config |
| Network | Where can it connect? | Deny by default, with an allowlist for the registries and APIs the task needs |
| Credentials | What secrets are in reach? | Short-lived, task-scoped tokens injected by the runtime, never in the prompt |
| Lifetime | How long does it live and what survives? | Ephemeral by default, with explicit outputs like a branch or an artifact |

And then there's the isolation mechanism itself. Roughly from lightest to strongest:

- **OS-level process sandboxes.** [Landlock](https://docs.kernel.org/userspace-api/landlock.html) on Linux and Seatbelt on macOS. This is what nono uses. Fast, local, and great for an agent on your laptop. The agent still shares your kernel.
- **Containers.** Namespaces and cgroups. Good packaging and good defaults if you configure them, but the kernel is shared and the defaults are built for convenience.
- **User-space kernels.** [gVisor](https://gvisor.dev/) intercepts system calls so the workload never talks to the host kernel directly.
- **MicroVMs.** [Firecracker](https://firecracker-microvm.github.io/) and [Kata Containers](https://katacontainers.io/) give each workload its own kernel with startup times measured in fractions of a second. This is where most hosted agent sandboxes are heading, and it's what I'd want for code from strangers.

The mechanism matters less than people think, though. A microVM with your production credentials mounted in it is worse than a process sandbox with nothing in it. **Configuration beats isolation strength.** Which brings me to testing.

## Don't trust a sandbox. Test it.

Here's the thing that bothered me. Every agent platform says its environment is sandboxed. Very few give you a way to check. And the failure mode is silent: a sandbox that leaks looks exactly like a sandbox that works, right up until it doesn't.

So I added a small feature to [factory-doctor](https://github.com/dewan-ahmed/factory-doctor), the preflight tool from part 4: checks that are supposed to **fail** ([PR #7](https://github.com/dewan-ahmed/factory-doctor/pull/7)). Set `"expect": "fail"` and the check passes only when the underlying access is denied:

```json
{
  "version": 1,
  "checks": [
    {"name": "Workspace is readable", "kind": "file", "target": "/tmp", "timeout_ms": 100},
    {"name": "Host SSH keys are hidden", "kind": "file", "target": "/home/ubuntu/.ssh", "expect": "fail", "timeout_ms": 100},
    {"name": "Arbitrary egress is blocked", "kind": "http", "target": "https://example.com", "expect": "fail", "timeout_ms": 3000}
  ]
}
```

That's a sandbox contract in eleven lines. The workspace should work. My SSH keys and the open internet should not.

On my unsandboxed VM, it fails the way it should:

```text
PASS  Workspace is readable (0 ms)  directory is readable
FAIL  Host SSH keys are hidden (0 ms)  expected failure but check passed: directory is readable
FAIL  Arbitrary egress is blocked (153 ms)  expected failure but check passed: HTTP 200

FAIL: 1 passed, 0 warned, 2 failed
```

Next, I tried a container with the convenient settings: default networking, and my home directory mounted read-only so the agent can see my git config.

```sh
docker run --rm \
  -v /tmp/factory-doctor:/usr/local/bin/factory-doctor:ro \
  -v /home/ubuntu:/home/ubuntu:ro \
  -v /tmp/sandbox-check.json:/work/sandbox-check.json:ro -w /work \
  debian:stable-slim factory-doctor check --config sandbox-check.json
```

```text
PASS  Workspace is readable (0 ms)  directory is readable
FAIL  Host SSH keys are hidden (0 ms)  expected failure but check passed: directory is readable
FAIL  Arbitrary egress is blocked (97 ms)  expected failure but check passed: HTTP 200

FAIL: 1 passed, 0 warned, 2 failed
```

Same result as no sandbox at all. It's "in a container," and it can still read my keys and talk to anything on the internet. Read-only doesn't help when the problem is reading.

Then the version I'd actually hand to an agent: no network, read-only root, nothing from my home directory.

```sh
docker run --rm --network none --read-only \
  -v /tmp/factory-doctor:/usr/local/bin/factory-doctor:ro \
  -v /tmp/sandbox-check.json:/work/sandbox-check.json:ro -w /work \
  debian:stable-slim factory-doctor check --config sandbox-check.json
```

```text
PASS  Workspace is readable (0 ms)  directory is readable
PASS  Host SSH keys are hidden (0 ms)  failed as expected: file_not_found
PASS  Arbitrary egress is blocked (0 ms)  failed as expected: http_request_failed

PASS: 3 passed, 0 warned, 0 failed
```

Exit code `0`. Now I have evidence, not a claim. And because it's a config file and an exit code, I can run the same contract as the first step of every agent session, in every runtime I use, and fail the session if the sandbox has drifted.

In a real setup, `--network none` is usually too strict. The agent needs a package registry and a Git host. That's fine. Add those as normal checks that must pass, keep `example.com` (or anything else off the allowlist) as a check that must fail, and you've tested both sides of the egress policy.

## Credentials are the other half

Filesystem and network rules keep the agent from finding secrets. Credential design keeps you from handing them over.

The pattern I keep coming back to, in [artifact-bridge](https://github.com/dewan-ahmed/artifact-bridge) and in the MCP servers from part 2: **the runtime holds the credential, the tool uses it, and the model never sees it.** A header is read from a named environment variable. A token is scoped to one repo and expires with the session. Reports strip credentials and query strings from URLs. If the model is compromised by an injected prompt, there's nothing in its context to leak.

The next step, which I think is where the industry is going, is to move credentials out of the sandbox entirely: a proxy outside the boundary that attaches auth to allowed requests. The agent can call the API, but it can't copy the key.

## Where I think sandboxes are heading

- **Sandboxes become the default runtime for agents.** Running a coding agent directly on a laptop with full access will look the way running production as root looks today. Possible, occasionally necessary, and a bit embarrassing.
- **Policy becomes portable.** Today every platform has its own way to describe network and filesystem rules. I expect that to converge into something declarative that travels with the repo, the same way `AGENTS.md` carries instructions.
- **Verification becomes a feature.** Platforms will expose evidence of what a session could reach, and teams will test that evidence in CI the way they test everything else. That's why I wanted negative checks in factory-doctor.
- **Approval prompts fade.** Clicking "allow" forty times a session trains people to click "allow." A tight sandbox with a short allowlist is both safer and less annoying.

## Talks worth your time

- [Sandboxes Aren't Optional: 4 Ways to Isolate Coding Agents](https://www.youtube.com/watch?v=iH_wV8BbS24), Robert Brennan (OpenHands), AI Engineer.
- [Prompt Injection, explained](https://www.youtube.com/watch?v=FgxwCaL6UTA), Simon Willison.
- [2026 in LLMs](https://www.youtube.com/watch?v=9RKrSF9d2SE), Simon Willison, AI Engineer.

## Wrapping up the series

Five posts, one idea underneath all of them: the model is the least predictable part of an agent system, so everything around it should be as predictable as possible. A loop you can test. Tools with schemas and actionable errors. Protocol boundaries only where there's a real boundary. A software factory that fails fast and stops spending. And a sandbox you've actually tested.

If you're building agents and disagree with any of this, I'd like to hear about it. That's how these opinions get better.

---

**A note on how this was made:** I developed this series with an AI assistant. The structure, the examples from my own projects, and the opinions are mine, pressure-tested and organized through that conversation.
{: .disclaimer}
