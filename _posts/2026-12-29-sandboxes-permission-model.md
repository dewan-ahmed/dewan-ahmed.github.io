---
title: "Sandboxes Are the New Permission Model"
date: 2026-12-29T00:00:00-04:00
author: Dewan Ahmed
excerpt: "For agents, the sandbox is the permission model. Here is what it has to control, and how I test that it actually blocks what it claims to."
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

Back in February I wrote about [sandboxing OpenClaw with nono]({% post_url 2026-02-03-sandbox-openclaw-nono %}). I ran a few harmless commands against my own VM, watched the agent list my home directory and read `/etc/passwd` without blinking, then watched the kernel say no once nono was in front of it.

That post changed how I think about sandboxes. I used to treat them as hardening, the thing you add once the agent works. Now I think the sandbox is the permission model. Prompts, tool descriptions, and approval dialogs all help, but what actually limits an agent is what its runtime can reach.

This is the last part of the series, so I want to do more than repeat that opinion. I wanted a way to check a sandbox instead of trusting its marketing page, so I built one and ran it against a few setups. Some of the results were not what the word "container" suggests.

## The old permission model doesn't fit

Traditional permissions assume the principal is a person or a service with predictable intent. An agent is neither. It acts on your behalf, with your credentials, while reading content written by strangers: web pages, issues, pull request comments, tool results, and artifacts from other agents.

The clearest framing I know is Simon Willison's [lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/). If an agent has:

1. access to private data,
2. exposure to untrusted content, and
3. a way to communicate externally,

then a prompt injection can turn it into an exfiltration tool. A better system prompt won't save you, because the attack arrives as data the agent has to read to do its job.

I kept running into this while writing the series. Tool results in [part 2]({% post_url 2026-12-08-mcp-tool-design %}) are untrusted input. So are artifacts from a remote agent in [part 3]({% post_url 2026-12-15-a2a-boundaries %}), and the issues and dependencies a coding agent reads in [part 4]({% post_url 2026-12-22-coding-agents-software-factory %}). Every time, I ended up at the same answer: take away one leg of the trifecta, and enforce it somewhere below the agent.

## What I look at in a sandbox

When I evaluate an agent runtime, I ask four questions:

| Boundary | The question | What good looks like |
| --- | --- | --- |
| Filesystem | What can the agent read and write? | The workspace only. No home directory, no SSH keys, no cloud config |
| Network | Where can it connect? | Deny by default, with an allowlist for the registries and APIs the task needs |
| Credentials | What secrets are in reach? | Short-lived, task-scoped tokens injected by the runtime, never in the prompt |
| Lifetime | How long does it live and what survives? | Ephemeral by default, with explicit outputs like a branch or an artifact |

Then there's the isolation mechanism itself. Roughly from lightest to strongest:

- **OS-level process sandboxes.** [Landlock](https://docs.kernel.org/userspace-api/landlock.html) on Linux and Seatbelt on macOS. This is what nono uses. Fast, local, and great for an agent on your laptop. The agent still shares your kernel.
- **Containers.** Namespaces and cgroups. Good packaging, and good isolation if you configure it, but the kernel is shared and the defaults are built for convenience.
- **User-space kernels.** [gVisor](https://gvisor.dev/) intercepts system calls so the workload never talks to the host kernel directly.
- **MicroVMs.** [Firecracker](https://firecracker-microvm.github.io/) and [Kata Containers](https://katacontainers.io/) give each workload its own kernel with startup times measured in fractions of a second. I think this is where most hosted agent sandboxes are heading, and it's what I'd want for running code from strangers.

People love arguing about that list. I care about it less than I used to. A microVM with your production credentials mounted in it is worse than a process sandbox with nothing in it. How you configure the sandbox matters more than which one you picked, and configuration is exactly the part nobody checks.

## So I tested mine

Every agent platform says its environment is sandboxed. Very few give you a way to check. What bothers me is that a leaky sandbox and a working one look identical from the inside. The agent does its task either way. You only find out the difference when something reads a file it shouldn't have.

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

Seven lines, and it describes what I want from a sandbox. The workspace should work. My SSH keys and the open internet should not.

First, the baseline. On my unsandboxed VM, it fails, which is the correct answer:

```text
PASS  Workspace is readable (0 ms)  directory is readable
FAIL  Host SSH keys are hidden (0 ms)  expected access to be denied but it was allowed: directory is readable
FAIL  Arbitrary egress is blocked (153 ms)  expected access to be denied but it was allowed: HTTP 200

FAIL: 1 passed, 0 warned, 2 failed
```

Next, the container I suspect a lot of us actually run: default networking, and my home directory mounted read-only so the agent can see my git config.

```sh
docker run --rm \
  -v /tmp/factory-doctor:/usr/local/bin/factory-doctor:ro \
  -v /home/ubuntu:/home/ubuntu:ro \
  -v /tmp/sandbox-check.json:/work/sandbox-check.json:ro -w /work \
  debian:stable-slim factory-doctor check --config sandbox-check.json
```

```text
PASS  Workspace is readable (0 ms)  directory is readable
FAIL  Host SSH keys are hidden (0 ms)  expected access to be denied but it was allowed: directory is readable
FAIL  Arbitrary egress is blocked (97 ms)  expected access to be denied but it was allowed: HTTP 200

FAIL: 1 passed, 0 warned, 2 failed
```

Same result as no sandbox at all. It's "in a container," and it can still read my keys and talk to anything on the internet. Read-only doesn't help when the problem is reading.

This is the run I'd want every team to see once. Each of those flags looks reasonable on its own, and together they give the agent everything the first run did.

<!-- TODO(Dewan): if you have a real moment where a convenience mount or default network caught you out (on a laptop, in CI, or with an agent), one or two sentences here would land better than anything general. -->

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

Exit code `0`. Now I have a result I can point to instead of a vendor's word for it. And because it's a config file and an exit code, I can run the same check as the first step of every agent session, in every runtime I use, and fail the session if the sandbox has drifted.

In a real setup, `--network none` is usually too strict. The agent needs a package registry and a Git host. That's fine. Add those as normal checks that must pass, keep `example.com` (or anything else off the allowlist) as a check that must fail, and now you're testing both sides of the egress policy.

One thing I got wrong the first time: my original version counted *any* failure as a successful denial. Code review on the PR caught it. If `example.com` returned a 404, the HTTP check "failed", so the negative check passed, even though the sandbox had clearly reached the internet. Same for a `git_remote` check on a runner with no `git` installed: nothing was blocked, the probe just never ran. Now factory-doctor only accepts failures that actually mean denial (no HTTP response at all, a missing or unreadable file, a command that ran and failed). Anything else reports `inconclusive_denial` and fails. A sandbox test that passes when the test itself is broken is worse than no test.

<!-- TODO(Dewan): consider adding the allowlist version you actually use (registry + Git host as must-pass, example.com as must-fail) once you've run it, with its real output. -->

## Credentials are the other half

Filesystem and network rules keep the agent from finding secrets. Credential design keeps you from handing them over.

I've ended up with the same pattern in [artifact-bridge](https://github.com/dewan-ahmed/artifact-bridge) and in the MCP servers from part 2. The runtime holds the credential and the tool uses it. The model never sees it. In artifact-bridge, an auth header is read from a named environment variable and never printed, and reports strip credentials and query strings from URLs. factory-doctor scrubs its targets the same way. If an injected prompt takes over the model, there's nothing in its context worth stealing.

What I'd want on top of that is a token scoped to one repo that expires with the session, so even a leaked one isn't worth much.

The next step, and I think this is where things are going, is to move credentials out of the sandbox entirely: a proxy outside the boundary that attaches auth to allowed requests. The agent can call the API, but it can't copy the key.

## Where I think sandboxes are heading

- **Sandboxes become the default runtime for agents.** Running a coding agent directly on a laptop with full access will look the way running production as root looks today. Possible, occasionally necessary, and a bit embarrassing.
- **Policy becomes portable.** Today every platform has its own way to describe network and filesystem rules. I expect that to converge into something declarative that travels with the repo, the same way `AGENTS.md` carries instructions.
- **Verification becomes a feature.** Platforms will expose evidence of what a session could reach, and teams will test it in CI like everything else. That's why I wanted negative checks in factory-doctor.
- **Approval prompts fade.** Click "allow" forty times in a session and you've trained yourself to click "allow." A tight sandbox with a short allowlist is safer and a lot less annoying.

## Talks worth your time

- [Sandboxes Aren't Optional: 4 Ways to Isolate Coding Agents](https://www.youtube.com/watch?v=iH_wV8BbS24), Robert Brennan (OpenHands), AI Engineer.
- [Prompt Injection, explained](https://www.youtube.com/watch?v=FgxwCaL6UTA), Simon Willison.
- [2026 in LLMs](https://www.youtube.com/watch?v=9RKrSF9d2SE), Simon Willison, AI Engineer.

## Wrapping up the series

Looking back over the five posts, I kept making the same argument in different places. The model is the least predictable part of an agent system, so I want everything around it to be as boring as possible. In part 1 that meant a loop I can test. In part 2, tools with schemas and errors an agent can act on. Part 3 was about only adding a protocol boundary where there's a real one, and part 4 was a factory that fails fast and knows when to stop spending. Here, it's a sandbox I've actually checked.

I'm sure some of these opinions are wrong, or will be in a year. If you're building agents and you disagree with any of them, tell me. I'd rather find out from you than from an incident.

---

**A note on how this was made:** I developed this series with an AI assistant. The structure, the examples from my own projects, and the opinions are mine, pressure-tested and organized through that conversation.
{: .disclaimer}
