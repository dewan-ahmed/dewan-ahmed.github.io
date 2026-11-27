---
title: "Agents Are Loops: What I Look for in an Agent Framework"
date: 2026-12-01T00:00:00-04:00
author: Dewan Ahmed
excerpt: "Every agent framework is a model in a loop. Here is the loop, how I test it with a fake model, and the five questions I ask before adopting a framework."
series: agent-builder
series_part: 1
header:
  teaser: "/assets/images/2026/agent-builder-part-1.png"
tags:
  - ai
  - agents
  - software-engineering
---

{% include agent-builder-series.html %}

In November I wrote four small command-line tools for agents to call: [factory-doctor](https://github.com/dewan-ahmed/factory-doctor), [artifact-bridge](https://github.com/dewan-ahmed/artifact-bridge), [flowfacts](https://github.com/dewan-ahmed/flowfacts), and [runway](https://github.com/dewan-ahmed/runway). None of them is an agent. They're the boring pieces an agent leans on. Is this machine ready? Can I fetch this file without leaking a token? What's actually in this pipeline? Are we about to blow the budget? Building them meant a lot of time watching agents use them, and that changed how I think about agent frameworks.

So when someone asks me which framework to use, I answer with a question: what happens when your agent crashes halfway through a task?

If the answer is "it starts over and we pay for the tokens again", the framework decision is still open. If the answer is "I don't know", we should talk about that first.

This is part 1 of a five-part series. I'm writing it for people who have already wired a model to a tool, watched it do something impressive, then watched it do something baffling five minutes later. I'll go through the agent loop, MCP, A2A, coding agents, and sandboxes, and I'll tell you where I think each one is heading. All four tools are open source, so you can check my work.

## Strip the branding and every agent is the same loop

An agent is a model in a loop with tools. The model reads the conversation so far and either answers or asks for a tool call. Your code runs the tool, appends the result, and goes around again. Everything a framework adds sits on top of that.

Here's a complete agent in about 50 lines of Python. It has one tool, and that tool runs factory-doctor's preflight checks.

```python
import os
import subprocess

import anthropic

client = anthropic.Anthropic()        # reads ANTHROPIC_API_KEY
MODEL = os.environ["AGENT_MODEL"]     # any tool-capable model

TOOLS = [{
    "name": "run_preflight",
    "description": (
        "Run factory-doctor preflight checks and return its JSON report. "
        "Use this before starting work to confirm tools, services and repo access."
    ),
    "input_schema": {
        "type": "object",
        "properties": {
            "config": {"type": "string", "description": "Path to a factory-doctor config file."}
        },
        "required": ["config"],
    },
}]


def run_preflight(config: str) -> str:
    result = subprocess.run(
        ["factory-doctor", "check", "--config", config, "--json"],
        capture_output=True, text=True, timeout=60,
    )
    # Exit code 1 means a critical check failed; the JSON report is still on stdout.
    return result.stdout or result.stderr


HANDLERS = {"run_preflight": run_preflight}


def run_agent(task: str, max_turns: int = 10) -> str:
    messages = [{"role": "user", "content": task}]
    for _ in range(max_turns):
        response = client.messages.create(
            model=MODEL, max_tokens=2048, tools=TOOLS, messages=messages,
        )
        messages.append({"role": "assistant", "content": response.content})

        if response.stop_reason != "tool_use":
            return "".join(b.text for b in response.content if b.type == "text")

        results = []
        for block in response.content:
            if block.type != "tool_use":
                continue
            try:
                output = HANDLERS[block.name](**block.input)
                results.append({"type": "tool_result", "tool_use_id": block.id, "content": output})
            except Exception as exc:  # errors go back to the model, not up the stack
                results.append({"type": "tool_result", "tool_use_id": block.id,
                                "content": f"{type(exc).__name__}: {exc}", "is_error": True})
        messages.append({"role": "user", "content": results})

    raise RuntimeError(f"agent did not finish within {max_turns} turns")
```

I used the Anthropic SDK for this example. The OpenAI, Gemini, and open-weight equivalents look almost identical: a list of tool schemas, a response that says "call this tool with these arguments", and a message type for sending results back.

A few lines in there do more work than they look like they do.

`max_turns` is the one I'd never ship without. A loop with no limit keeps calling the model for as long as the model keeps asking for tools, and you find out what that costs from the invoice. A turn limit stops one runaway run. It doesn't help when every run looks fine and the month doesn't, which is the problem I wrote runway for. It forecasts spend to the end of a budget period and exits non-zero when the forecast crosses your warning or stop threshold. `runway anomalies` also flags a day that spends far more than the same kind of day (weekday or weekend) usually does, with a reason code that says why.

The `except` block sends errors back to the model as a tool result instead of raising them. If factory-doctor isn't on the PATH or the check times out, the model reads the error and can decide what to do next. If you raise, one bad path ends the whole run.

And the tool returns facts, not prose. factory-doctor prints a JSON report and exits `0` when every critical check passed or `1` when one didn't. A check marked `"critical": false` shows up as a warning without failing the run. The model never has to work out whether "mostly fine" means go.

If you've never written this loop by hand, do it once. It's much easier to judge what a framework is selling when you know which parts you could have written yourself in an afternoon.

## What the loop doesn't give you

The 50-line version is fine on a laptop. It falls apart once a task outlasts one sitting, needs a person to sign off, or spends real money. That's where frameworks earn their keep, and it's where I'd spend my evaluation time:

| Problem | What you need | Where you'll find it |
| --- | --- | --- |
| The process dies on turn 14 | Durable state you can resume from | Checkpointers (LangGraph), durable execution engines (Temporal and friends) |
| A person must approve a step | Pause, persist, resume hours later | Interrupts and approval hooks |
| The conversation outgrows the context window | Compaction, summarization, trimming tool output | Agent harnesses like the Claude Agent SDK; your own code otherwise |
| You can't tell why it did that | Traces of every model call, tool call, and token count | OpenTelemetry instrumentation, vendor tracing |
| One agent's context is full of the wrong things | Subagents with isolated context | Handoffs (OpenAI Agents SDK), subagents, graph nodes |
| The bill surprised you | Turn, token, and spend budgets | Usually your own code |

None of those rows is about a nicer way to define a tool. Every framework already handles that well enough. The differences I care about are in the runtime: state, interruption, context, observability, and cost. Look at how many of those rows end in "your own code". That's the part a framework demo tends to skip.

## The five questions I ask a framework

Before I look at a single tool decorator, I want answers to these:

1. Can I resume after a crash without repeating side effects? If a tool already opened a pull request, the resumed run shouldn't open a second one.
2. Can I pause for a human and pick it back up tomorrow? Real workflows wait on people, and keeping a Python process alive overnight isn't a plan.
3. Can I see every model call and tool call, with token counts? If I can't trace it, I can't debug it, and I definitely can't explain the invoice.
4. Can I swap the model provider without rewriting the agent? Models leapfrog each other every few months. My tool layer shouldn't have to care.
5. Can I unit-test the loop with a fake model? If the only way to test the agent is to call a real model, the test suite turns into an eval suite with a monthly bill.

You can check the last one yourself in a few minutes. Here's the loop from above, tested with a scripted fake client:

```python
import os
from types import SimpleNamespace as NS

os.environ.setdefault("ANTHROPIC_API_KEY", "test")
os.environ.setdefault("AGENT_MODEL", "fake-model")
import agent  # the module above

script = [
    NS(stop_reason="tool_use", content=[
        NS(type="tool_use", id="t1", name="run_preflight", input={"config": "factory-doctor.json"}),
    ]),
    NS(stop_reason="end_turn", content=[NS(type="text", text="All critical checks passed.")]),
]
agent.client = NS(messages=NS(create=lambda **_: script.pop(0)))
agent.HANDLERS["run_preflight"] = lambda config: '{"passed": 4, "failed": 0}'

assert agent.run_agent("Is this machine ready?") == "All critical checks passed."
```

No real API key and no network calls. The fake client returns a tool call, then a final answer, and the assertion checks that the loop handled both. I test the loop and the tools this way and save real model calls for evals that measure behaviour. Those I run on purpose, not on every commit.

## Where I've landed, for now

These are opinions. They've gotten stronger the more time I've spent building tools for agents, but I expect at least one of them to age badly.

### 1. Put deterministic work in tools, not prompts

The cheapest reliability win I know of is taking work away from the model.

flowfacts is my clearest example. Hand a migration agent a long Jenkinsfile and ask it to reason about it, and you've spent a chunk of context on boilerplate and given the model room to confidently "remember" a stage that isn't there. With flowfacts, the agent runs one command and gets compact JSON back: stages, dependencies, credential IDs (never the values), artifacts, and an explicit `unresolved` list for things like dynamic Groovy.

That `unresolved` list is my favourite part of the project. It marks where deterministic extraction stops and judgment has to start, so the model spends its attention on the parts that need it instead of re-deriving what a parser already knows.

My rule of thumb now: if a program can compute the answer, don't ask the model for it.

### 2. Exit codes and JSON are agent UX

We spent decades making CLIs nice for humans, with colours, spinners, and friendly messages. Agents want close to the opposite: stable JSON on stdout, diagnostics on stderr, and exit codes that each mean one thing.

All four of my tools follow the same contract. `0` means go ahead. A small set of non-zero codes mean specific failures, like runway's `1` for "the forecast hit the warning threshold" and `2` for "the forecast hit the stop threshold". And one code means you called me wrong: `2` for factory-doctor, artifact-bridge, and flowfacts, and `3` for runway, since its `2` was already taken.

artifact-bridge has the small example I like best. A login page can come back with a 200, and if all you check is the status code, the agent thinks it has your build. So artifact-bridge rejects HTML login and error pages by default and exits `1`, and the agent gets a clear failure instead of a `build.zip` full of HTML.

It's boring, and I want it boring. The same binary works from a terminal, a CI step, an agent hook, or an MCP server, which is where part 2 picks up.

### 3. Start with one agent and good tools

Multi-agent architectures look great on a whiteboard. Most of the ones I've seen would have worked better as one agent with better tools.

Kubernetes upgrade checks are a good example. On a whiteboard, that's four agents: one renders the Helm charts, one looks up which APIs the target release removed, one writes migration notes, and one reviews the result. When I built [k8s-workers](https://github.com/dewan-ahmed/k8s-workers), the lookup became a deterministic catalog behind one API call. It reports every removed or deprecated `apiVersion` in your manifests with its replacement, so a single agent can decide what to change. No model has to remember that `batch/v1beta1` CronJob was removed in 1.25.

There's one reason to split that I fully buy: context isolation. A subagent that reads 200 files to answer "where is retry logic configured?" and comes back with one paragraph keeps those 200 files out of the main agent's context. That's worth a lot. "The planner agent hands off to the coder agent, who hands off to the reviewer agent" usually isn't. You end up with three prompts working on one problem and two handoffs where information can get lost.

I split when the context or the trust boundary forces it. Trust boundaries get their own post in part 3.

### 4. Frameworks are thinning, and that's good

Look across the major options today (LangGraph, the OpenAI Agents SDK, the Claude Agent SDK, Google's ADK, Microsoft Agent Framework, Pydantic AI, Mastra) and they've converged on the same core: a loop, typed tools, state, tracing, and some form of handoff. Most of the clever abstractions from 2023 have been sanded off.

I expect that to continue. The loop becomes a commodity, and the decisions that matter move to the tool layer, which is increasingly MCP, and to the environment the agent runs in, which is increasingly a sandbox. That's why the rest of this series spends its time there. My bet is that once you get the tools and the environment right, picking a framework stops feeling like a big decision.

## Talks worth your time

If you want to go deeper on the ideas in this post, these are the talks I point people to:

- [How We Build Effective Agents](https://www.youtube.com/watch?v=D7_ipDqhtwk), Barry Zhang (Anthropic), AI Engineer.
- [12-Factor Agents: Patterns of reliable LLM applications](https://www.youtube.com/watch?v=8kMaTybvDUw), Dex Horthy (HumanLayer), AI Engineer.
- [The Great Loops Debate](https://www.youtube.com/watch?v=c35YoMdnI78), Dex Horthy, Geoff Huntley, Ian Livingstone, and Greg Pstrucha, AI Engineer.
- [Software Is Changing (Again)](https://www.youtube.com/watch?v=LCEmiRjPEtQ), Andrej Karpathy, Y Combinator.

## Next up

In part 2, I take flowfacts and expose it over MCP, then share what I've learned about designing tools for models: fewer of them, shaped like tasks, with errors the model can act on. MCP also changed a lot in 2026, and some of those changes are worth understanding even if you never write a server.

---

**A note on how this was made:** I developed this series with an AI assistant. The structure, the examples from my own projects, and the opinions are mine, pressure-tested and organized through that conversation.
{: .disclaimer}
